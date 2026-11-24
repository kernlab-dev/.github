# KernLab 家族 GitHub Actions 可复用管线与组织中央运维工具

本仓库作为 kernlab-dev 组织的工程基座，不再直接构建业务产品，而是以 `workflow_call` 暴露一组**按节点码命名**的可复用工作流（e10/v10/v30/m10/r10/w10/o10），并为各产品仓提供 `workflow-templates/gate.yml` 单入口装配模板。

## 1. 采用的系统

- **平台**: GitHub Actions；所有逻辑工作流均以 `workflow_call` 暴露，被各子仓通过 `uses: kernlab-dev/.github/...` 引用。
- **语言栈**: .NET SDK 8.0.x（默认 `8.0.422`）用于构建/策略校验 + 9.0.x 用于 NuGet OIDC Trusted Publishing。
- **文档工具链**: DocFX（`docfx-template/` 为标准站点模板）；新增 v30 docs 质量门对 `.md/.mdx` 做站内相对链接存亡检查。
- **制品库**: nuget.org，支持 API Key 与 OIDC Trusted Publishing 双模式。
- **聚合部署**: 由 `kernlab-dev/kernlab-docs` 聚合层推送到 Cloudflare Pages → `docs.kernlab.dev/<product>/`。

## 2. 关键文件

### 2.1 可复用工作流目录 (`reusable-<node-code>.yml`)

| 文件 | 节点代码 | 职责 |
|---|---:|---|
| `reusable-e10-policy.yml` | e10 | 强校验：来源分支 ↔ 目标分支矩阵、Tag SemVer、Tag↔Version/CHANGELOG、预发 GA Tag 分支可达性——**唯一合法红** |
| `reusable-v10-ci.yml` | v10 | 完整门禁：build + 全量测试（缺省 sln 自动发现 + tests/*/*.csproj 顺序跑） |
| `reusable-v30-docs.yml` | v30 | docs 快速通道质量门：PR 新增/修改的 md/mdx 站内链接解析（5 分钟超时） |
| `reusable-m10-docs-merge.yml` | m10 | docs 自动合并：竞态复核 + squash + 删除临时分支 |
| `reusable-r10-release.yml` | r10 | 发布管线：build→pack→publish（API Key 优先、OIDC 兜底；snupkg 容错） |
| `reusable-w10-docs-deploy.yml` | w10 | 文档站：DocFX 构建 → kernlab-docs 聚合 → Cloudflare Pages |
| `reusable-e20-triage.yml` | e20 | PR 分诊：全量 diff → 路由 `docs_only` 信号 |
| `reusable-policy-check.yml` | — | **旧名兼容**（→ e10），README 标注勿新增引用 |
| `reusable-ci.yml` / `reusable-release.yml` / `reusable-docs-deploy.yml` | — | **旧名兼容**，仅保留以供迁移过渡 |

### 2.2 组织中央运维工具

- `o10-nuget-unlist.yml`: 组织级 NuGet 包软下架（unlist）工具。输入 package_id/version/确认词 `unlist`，调用 `dotnet nuget delete --non-interactive`，依赖组织 secret `NUGET_UNLIST_USER`。内置 GA 保护：仅允许 prerelease 版本进入，正式版走人工评审。随 `r10-release` 形成发布↔回滚对。

### 2.3 Gate 模板

- `workflow-templates/gate.yml`: **每仓唯一的 workflow** 入口。纯接线：继承组织 reusable，组合 e10-policy → (e20-triage → v10-ci / v30-docs → m10-docs-merge) / r10-release / w10-docs-deploy。
- `workflow-templates/gate.properties.json`: 模板清单描述（name/description/iconName/categories）。
- 旧三件套（`ci.yml`/`docs-deploy.yml`/`policy.yml` + `*.properties.json`）已退役。

## 3. 架构与约定

### 3.1 Gate 分层模型

CI-GATE.md 将管线分成四层：
- **入口层** (e10/e20): 恒跑恒成功或失败，不阻塞下游
- **验证层** (v10/v30/v20-extension/v40-heavy): 需 e10+e20 均成功才跑
- **汇合层** (m10): 在 docs-only 路径上把 PR 分支 squash 合并到目标
- **交付层** (r10/w10): tag push/main push 触发

```gate.yml 约束:
  e10-policy    → e20-triage
                  ├─ v10-ci (needs: [e10, e20], IF: 非 docs-only)
                  └─ v30-docs (needs: [e10, e20], IF: docs_only=true)
                         └─ m10-docs-merge (IF: draft=false, v30 success)
  e10-policy    → r10-release (IF: push, refs/tags/v*)
  e10-policy    → w10-docs-deploy (IF: push, refs/heads/main)
```

### 3.2 Node Code 命名规范

根据 CI-GATE.md §3–§4:
- 前缀字母 = 生命周期阶段：e=入口、v=验证、m=汇合、r=发布、w=运维/部署；o=组织中央工具
- 数字位 = 主要语义（10=policy, 20=triage, 30=docs-check, 40=heavy, etc.）
- 文件名强制为 `reusable-<node-code>.yml`，与工作流 top-level name 一致

### 3.3 Release 双模认证

`reusable-r10-release.yml` 同时支持两种 NuGet 认证路径：
1. **API Key 路径**（现有仓库）：读取 `secrets.NUGET_*` + `id-token: write`（但走 API Key）
2. **OIDC Trusted Publishing**（新仓库推荐）：`id-token: write` + nuget.org 登记 caller 身份
当使用 API Key 时若 OIDC 登录失败则跳过 oidc login，继续走 API Key 推送。

### 3.4 Docs 快速通道

当 e20-triage 判定 `docs_only=true` 且 PR 来自 `docs/*` base dev 时：跳过 v10-ci，改走 v30-docs 死链接检查，通过后由 m10 自动 squash 合并。

### 3.5 Snupkg 与 Publish 保护

- pack 脚本同时产出和上传 `artifacts/*.nupkg` 与 `artifacts/*.snupkg`（符号包历史 404 issue #50 已修复）
- release 发布任务的 `if` 条件限定 `github.event_name == 'push'`，dispatch 仅作 build/pack/upload artifact
- 发布 credential 缺失时应 fail-loud——这是正确信号

### 3.6 权限模型

Gate 顶层声明 `permissions.id-token: write`（因 r10 OIDC publish 需要此最小权限）；e10 仅 `contents: read`。

## 4. 约定的约束

- **组织单源原则**: 所有 reusable 工作流实现放在 `github.com/kernlab-dev/.github` 组织仓库，各产品仓只引用，不得本地重复实现。
- **唯一合法红**: e10-policy 是阻断所有下游的唯一 gate；其失败不会触发 cancelled downstream —— 通过显式 `needs.*.result == 'success'` 守卫实现。
- **无绕过开关**: policy-check 注释明确 "不提供绕过开关"；版本/Changelog/Branch 三角校验任一不一致即 `exit 1`。
- **NuGet 发布仅限 tag push**: release-publish 任务仅在 push event + refs/tags/v* 触发；dispatch 空跑仅 build/pack/upload artifact。
- **OIDC 绑定 caller**: Trusted Publishing 的 OIDC token 绑定 origin 仓库的 owner/repository/workflow 三元组；reusable 载体仓自身身份发布会返回 401，因此发布 leg 通常留由各仓自己的 release.yml 承载（设置 `skip_publish=true`）。
- **Snupkg 必须同步发布**: `artifacts/*.snupkg` 与 *.nupkg 同步处理，避免 nuget symbol service 404。
- **OA 包禁软下架**: o10-nuget-unlist 仅接受 prerelease（含 `-`）版本号，GA 包必须走人工评审流程。
- **Docs 死链接零容忍**: v30-docs 对任何新增/修改的 md/mdx 中的相对链接执行存在性检查，缺少目标即失败。
- **PR CI 时间限**: v30-docs 明确 `timeout-minutes: 5`；历史版本中 CI job 上限 30 分钟（长稳/AOT 拆到 main/nightly/manual 分支）。
- **工作流模板单入口**: 新仓应使用 `workflow-templates/gate.yml` 代替旧的 ci/docs-deploy/policy 三件套。
- **文档站双重守护**: w10 内置缺参数/无文档路径守卫；未启用文档站的仓库成本为零。

## 5. 不适用点

本仓库不包含传统 Makefile、Dockerfile、自研跨平台编译矩阵；构建语义完全收敛于上述 GitHub Actions reusable 工作流与 Gate 装配模板。

**Key files**:
- `workflow-templates/gate.yml`
- `workflow-templates/gate.properties.json`
- `.github/workflows/reusable-e10-policy.yml`
- `.github/workflows/reusable-v10-ci.yml`
- `.github/workflows/reusable-v30-docs.yml`
- `.github/workflows/reusable-m10-docs-merge.yml`
- `.github/workflows/reusable-r10-release.yml`
- `.github/workflows/reusable-w10-docs-deploy.yml`
- `.github/workflows/reusable-e20-triage.yml`
- `.github/workflows/o10-nuget-unlist.yml`
- `.github/workflows/README.md`
- `docfx-template/docfx.json`
- `docfx-template/main.css`
- `docfx-template/toc.yml`
- `VERSIONING.md`
- `CI-GATE.md`
- `RULES.md`