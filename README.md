# kernlab-dev/.github

本仓是 GitHub 约定的**组织级特殊仓库**，承载三件事：

1. **组织公开门面**：`profile/README.md` 渲染在组织 Overview 页（public 视图）。
2. **组织级默认社区健康文件**：本仓根部的 `SECURITY.md` / `SUPPORT.md` 等，
   会被组织内**没有自建同名文件**的仓库当作回退继承。注意这些默认文件
   **不会**出现在文件浏览器、git 历史、克隆与下载里——它们是隐形的回退。
3. **家族公共工程文档**：[`ENGINEERING.md`](ENGINEERING.md)（跨仓公共开发纪律）、
   [`RULES.md`](RULES.md)（编译器扩展诊断 ID 段位注册表）、
   [`docfx-template/`](docfx-template/)（文档站标准模板——样式/导航/构建/发布一站取用）。

4. **家族共享流水线（逻辑单源）**：[`.github/workflows/`](.github/workflows/) 下两条 reusable——
   `reusable-ci`（CI 门禁：默认 GitHub 托管机，`runners_matrix` 参数切自建 runner 组合）、
   `reusable-docs-deploy`（文档站构建 → kernlab-docs 聚合 → docs.kernlab.dev）。
   各仓只留调用桩；新仓从 [`workflow-templates/`](workflow-templates/)（Actions 页可选）
   ——**发布不入此列**：家族纪律 §9 发布不可集中（nuget.org Trusted Publishing 按 OIDC
   三要素逐字校验 workflow 身份，reusable 指向载体仓将被拒），发布由各仓自带 `release.yml`。
   取 `ci` / `docs-deploy` 桩。注意：本仓 workflows **不会自动跑在别的仓**——
   事件只触发事件所在仓的 workflow，共享靠各仓显式 `uses:` 调用。

组织内不成文的约定、仓地图与迁移台账在成员可见的 **`.github-private`**（不在此公开仓）。

> GitHub 要求组织级默认社区健康文件的载体仓为 public（托管型组织可 internal），故本仓为 public。

MIT © 2026 kernlab.dev
