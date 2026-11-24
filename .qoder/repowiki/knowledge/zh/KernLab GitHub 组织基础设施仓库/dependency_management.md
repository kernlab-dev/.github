## 1. 总体思路

本仓库是 KernLab GitHub 组织的 Actions 基础设施，自身不引用任何业务或第三方库。**整个组织依赖管理的核心——构建面 SDK 锁定、NuGet 依赖解析（通过 `dotnet build/restore` 默认从官方源下载）、包版本声明与发布渠道——全部被收敛到一组 reusable workflow 并由单入口 `workflow-templates/gate.yml` 驱动。** 各产品仓的 `.csproj` / `global.json` 仅负责「把依赖声明写在本仓」；具体拉取哪一版 .NET SDK、从哪个 NuGet feed、以及是否允许私有 registry，都集中在可复用工作流中统一决策。

## 2. 关键文件

| 文件 | 作用 |
|---|---|
| `workflow-templates/gate.yml` | 全仓唯一入口：继承所有 reusable，承担触发路由、权限（含 `id-token: write`）和参数装配 |
| `CI-GATE.md` | 组织级 CI/Gated 规范文档（0.2.10），定义节点码、发布门、OIDC 排障判例链 |
| `.github/workflows/reusable-e10-policy.yml` | 强校验（来源分支 / Tag / 版本 / CHANGELOG），非法输入短路整条管线 |
| `.github/workflows/reusable-v10-ci.yml` | 构建 + 测试门（build_only、test_solution、test_filter 等扩展槽位），依赖由 dotnet CLI 默认解析 |
| `.github/workflows/reusable-r10-release.yml` | 双模发布：API key 优先 → OIDC Trusted Publishing 兜底，含 snupkg 容错 |
| `.github/workflows/o10-nuget-unlist.yml` | 组织中央运维工具：对 `KernLab.*` 预发布包执行 `nuget delete` 软下架 |
| `.github/workflows/reusable-m10-docs-merge.yml` / `reusable-w10-docs-deploy.yml` / `reusable-v30-docs.yml` | 文档站流水线（构建、死链接检查、squash 合并、上站） |
| `.github/workflows/reusable-docs-check.yml` | DocFx 构建 / 站点检查的基础 reusable |
| `docfx-template/docfx.json` | DocFx 模板入口，固定文档站侧的第三方插件与主题 |

## 3. 架构与约定

- **单入口 gate 模式**：每个子仓只有一个 `gate.yml`（`Gate`），按 `CI-GATE.md §5` 调用 `reusable-e10-policy.yml@main`、`reusable-v10-ci.yml@main`、`reusable-r10-release.yml@main` 等；模板生成禁用 f-string 内嵌 `{{…}}`（避免 GH parser 拒收，见 CI-GATE 判例⑥）。
- **构建面 SDK 精确补丁锁定**：`inputs.dotnet_build: '8.0.422'`，注释明确「通配会被 latestMajor 滚上未知大版本」；release v10 ci 默认沿用。
- **发布面 SDK 最低范围**：`inputs.dotnet_publish: '9.0.x'`，因 NuGet OIDC Trusted Publishing 需要 .NET 9.0.2xx+；publish job 一开始 `rm -f global.json` 再 setup，避免 `global.json` 钉住构建面 SDK 导致跨 SDK metadata 不一致。
- **NuGet feed 策略**：仅 `https://api.nuget.org/v3/index.json`。工作流中未见 `dotnet nuget add source` 或 `--source` 指向私有 feed 之外的自定义源——**所有上游依赖经公共 NuGet 公共注册表解析**。私有访问走的是凭据而非 feed 替换。
- **包发现规则**：`pack_projects` 为空时通过 `grep -rls "<IsPackable>true</IsPackable>" $pack_root --include=*.csproj` 自动发现；无结果即 exit 1。故**包工程必须以 `<IsPackable>true</IsPackable>` 明示**（默认扫描目录 `src`）。
- **版本语义**：version input 优先级最高；tag 触发时用 `${GITHUB_REF_NAME#preview-v}` 剥离前缀，再去掉首字符 `v`，供 `dotnet pack -p:Version=$VERSION`。
- **双模发布**（`reusable-r10-release.yml`）：
  - 若 `secrets.NUGET_API_KEY` 存在 → API key 模式直接 push；
  - 否则请求 `id-token: write` 调用 `NuGet/login@v1` 以 `NUGET_USER` 换取临时 key，再 push。
  - 两种模式均 `--skip-duplicate` 实现幂等重试；snupkg 存在才 push，不存在静默跳过（alpha.19 经验修复）。
- **OIDC Trusted Publishing 登记**：新仓必须在 nuget.org 登记 `owner=kernlab-dev / repository=<caller> / workflow=<本 reusable 路径>`，否则 push 401；本流水线不做自动注册。
- **包回滚机制**：`o10-nuget-unlist.yml` 是 CentralOps 单实例入口，要求 package_id、version 及确认词 `unlist`；运行时校验版本必须包含 prerelease 段（`*-` glob match）拦截 GA 包误操作，然后执行 `dotnet nuget delete <pkg> <ver> --source https://api.nuget.org/v3/index.json --non-interactive`。正式版回滚须走人工评审通道，不走此 workflow。
- **DocFx 依赖**：`docfx-template/docfx.json` 是文档站唯一的依赖入口（插件、主题、Markdown 处理器均由此声明），`reusable-v30-docs.yml` 在其上附加站内死链接检查。

## 4. 观察到的约定与规则

- **构建面 SDK 禁止通配符**：注释「通配会被 latestMajor 滚上未知大版本」，workflow 输入值字面量 `'8.0.422'`。
- **发布面 SDK 只锁主版本范围**：`'9.0.x'`，因 OIDC 要求 ≥9.0.2xx。
- **包工程必须显式设置 `<IsPackable>true</IsPackable>`**，且位于 `pack_root`（默认 `src`）下才会被发现。
- **Tag 才是正式发布门**：r10 release job 条件 `if: github.event_name == 'push' && inputs.skip_pack != true && inputs.skip_publish != true`；dispatch 空跑整条 publish job 跳过。
- **顶层 caller 持有 `id-token: write`**：gate.yml 在 workflow 级别声明，注释指出缺该授权会触发 entire workflow startup_failure（reusable 的 permission 声明不会冒泡）。
- **Trusted Publishing 绑定目标为 reusable 路径**：`owner=kernlab-dev / repository=<caller> / workflow=github/workflows/reusable-r10-release.yml`；`skip_publish=true` 用于发布腿留在 caller 自己的 workflow（此时 OIDC 绑定 caller workflow）。详见 r10 输入说明。
- **nuget-org 访问仅使用单一官方端点** `https://api.nuget.org/v3/index.json`，未见 GOPRIVATE/VSTS/NPM proxy 等其它类型私有源配置。
- **GA 包禁止走 o10-nuget-unlist**：版本匹配 `*-*` 后才放行，正式稳定版本走人工评审流程。
- **凭证保护**：o10 workflow 以 `secrets.NUGET_UNLIST_USER` 认证；确认词非 `unlist` 立即 `exit 1`，并显式备注「日志自动打码（家族凭据纪律）」。
- **依赖变更安全审计**：当前可见流水线未内建 `dotnet list vulnerable` / `dotnet list outdated`；依赖相关策略约束应通过 `reusable-e10-policy.yml` / `reusable-policy-check.yml` 调用的外部 policy action 实施。

## 5. 结论

本仓库将 .NET 生态中的三类依赖治理关注点做了清晰分层：**依赖声明**归属各产品仓自己的 `csproj`/`global.json`；**SDK 版本与 NuGet feed** 集中在 reusable 工作流的 input 与 `dotnet` CLI 调用中；**包发布渠道（nuget.org 双模）** 与回滚工具 (`o10-nuget-unlist`) 以 central 形式提供，通过 `workflow-templates/gate.yml` 成为全组织统一的交付门。