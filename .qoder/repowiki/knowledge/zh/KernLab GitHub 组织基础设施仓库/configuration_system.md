# KernLab 组织级 Workflow Gate 与可复用管线配置

## 1. 采用的方案

本仓库不托管运行态应用配置；它作为 KernLab.dev 组织的 GitHub Actions 模板仓，提供一套**仓库门禁（Gate）模板 + 可复用 workflow**配置体系。核心思路：

- 每个产品仓库只保留一份 `workflow-templates/gate.yml`（纯接线），继承组织 `kernlab-dev/.github/.github/workflows/reusable-*.yml@main`。
- 所有通用逻辑集中到 `.github/workflows/reusable-*.yml` 系列文件；版本对齐统一通过 `uses: …@main` 完成。
- Gate 元数据由 `workflow-templates/gate.properties.json`（name / description / iconName / categories）描述，便于 Action Marketplace/模板索引。
- 运行时凭据仅以 GitHub secrets 注入（如 `NUGET_UNLIST_USER`、`DOCS_MERGE_PAT`、`DOCS_SYNC_TOKEN`），不在仓库中硬编码。

## 2. 关键文件

| 文件 | 作用 |
|---|---|
| `workflow-templates/gate.yml` | 每仓唯一的单入口 Gate，负责强校验→分诊→CI/docs/release→文档站发布 |
| `workflow-templates/gate.properties.json` | Gate 的 name/description/icon/categories 声明 |
| `.github/workflows/reusable-e10-policy.yml` | 版本/来源/Tag 强校验（version_project/changelog_file 可选参数） |
| `.github/workflows/reusable-e20-triage.yml` | PR 路由信号（docs_only 等输出） |
| `.github/workflows/reusable-v10-ci.yml` | 完整门禁（build + tests/*/*.csproj 顺序跑，简单仓零参数自动发现） |
| `.github/workflows/reusable-v30-docs.yml` | docs-only 快速通道质量门 |
| `.github/workflows/reusable-m10-docs-merge.yml` | docs-only 成功后的 squash+删分支自动合并 |
| `.github/workflows/reusable-r10-release.yml` | tag v* 触发，双模发布（API key 优先，OIDC id-token 兜底），snupkg 容错 |
| `.github/workflows/reusable-w10-docs-deploy.yml` | 文档站构建→聚合→上站，双守卫内建 |
| `.github/workflows/o10-nuget-unlist.yml` | 组织中央 NuGet 软下架运维工具（仅 prerelease 包，secret `NUGET_UNLIST_USER`） |
| `CI-GATE.md` | Gate 行为规范文档 |
| `CHANGELOG.md` | Gate/CI 相关变更记录 |
| `RULES.md` | 组织规则清单 |

## 3. 架构与约定

- **单入口设计**：`gate.yml` 是 `name: Gate` 唯一 workflow，被命名为“纯接线”层；扩展节点（v20-extension、v40-heavy、r10-release、w10-docs-deploy）按注释启用，遵循 `CI-GATE §8`。
- **权限显式化**：入口层声明 `contents: read; id-token: write`——后者的缺失会使 r10-release 在 Loom alpha.7 / repo-template 下直接 startup_failure（已在 CI-GATE §5 判例④ 记录为硬约束）。
- **并发控制**：`concurrency.group = gate-${{ github.workflow }}-${{ github.ref }}` 配合 `cancel-in-progress: true`，同一 ref 的新推覆盖旧行。
- **分诊分流**：`e20-triage` 产出 `docs_only` 等输出，下游 `v30-docs`、`m10-docs-merge`、`v10-ci` 通过 `if: needs.e20-triage.outputs.docs_only == 'true'` 条件执行，避免 docs-only PR 误跑构建矩阵。
- **Gate 阶段分层**：入口层(e10/e20) → 验证层(v10/v30) → 自动化(m10) → 按需(r10/w10)，依赖通过 `needs:` 串接。
- **安全约定**：o10-nuget-unlist 要求确认词严格等于 `unlist`，且版本号必须含 prerelease 段（`case "$PACKAGE_VERSION" in *-* ) : ;; * ) fail`），GA 包走人工评审流程。
- **可组合复用**：所有 reusable 都通过 `uses: kernlab-dev/.github/.github/workflows/<job>@main` 引用组织主分支，禁止仓库自持副本；模板元信息集中在 `gate.properties.json` 而非 workflow 内部 YAML。

## 4. 约定与约束

- 每个接入仓库只需保留一份 `workflow-templates/gate.yml`，不得复制 reusable 逻辑（见 `gate.yml` 顶部注释“组织单源；扩展/发布/文档站按需启用”）。
- 发布开关由分支名和触发器共同守护：`r10-release` 仅 `push refs/tags/v*` 激活；`w10-docs-deploy` 仅 `refs/heads/main` push；docs-only 路径还需 `head_ref` 前缀 `docs/` 且 `base_ref == dev`。（强制守卫实现于各自 workflow 的 `if:` 表达式）
- 文档自动化需要仓库 secret `DOCS_MERGE_PAT`；部署需 `DOCS_SYNC_TOKEN`，并在 Gate 中以 `secrets.DOCS_MERGE_PAT` / 同名变量注入（`gate.yml` 中显式映射）。
- NuGet 操作密钥只通过 `secrets.NUGET_UNLIST_USER` 注入；o10-nuget-unlist 对正式/GA 包（不含 `-` 的版本段）拒绝执行（shell `case` + exit 1）。
- 仓库间版本/Release 一致性由 e10-policy 的可选 `version_project` 与 `changelog_file` 参数驱动，缺省跳过（见 `gate.yml` 注释）。该能力属于“有开关但未启用”的配置点，并非强制约束。
- 并发取消策略在所有 gate 实例上统一采用 `${{ github.workflow }}-${{ github.ref }}` 分组键（写入在 `gate.yml` 顶层 `concurrency`）。
- 变更同步要求：任何 reusable workflow 的行为变动应同步更新 `CI-GATE.md`、`CHANGELOG.md` 与 `RULES.md`（仓库根目录现存规范）。