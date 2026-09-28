# KernLab 工程纪律（公共版）

> 本文件是 KernLab 家族的**跨仓公共开发纪律**——从各仓规则源提炼的共性约定。
> 层级：各仓 `AGENTS.md`（仓内唯一规则源，含域专属细则与判例）→ 本文件（家族共性）→
> [`RULES.md`](RULES.md)（编译器扩展诊断 ID 段位注册表）。
> 修订各仓细则改对应仓的 `AGENTS.md`；修订家族共性直接改本文件并提交。

## 1. 命名与身份

- 代码 / 包 / 命名空间前缀统一 **`KernLab.*`**（大写 L）；GitHub 仓名小写连字符 `kernlab-<product>`。
- **nuget.org 包显示大小写由首次发布锁定**（如 Tier 显示 `Kernlab.Tier.*`）；包 ID 大小写不敏感。
  消费者翻 `using`／包引用大小写**必须等上游发下一版**——两段式，别一次改完。
- **跨仓静默契约**（改而漏改不报错，只是规则不生效或降级，必须成批一起翻）：
  1. 生成器硬编码的属性 FQN 常量（漏 → 静默不产码）；
  2. 分析器配置键及其**值里的程序集前缀**（漏 → 静默不执法）；
  3. 诊断 ID 与 `dotnet_diagnostic.<ID>.severity`（漏 → 静默降级）。

## 2. 依赖纪律

- **下层有的不自研**：缺能力向下游仓提 issue，不在上游自研补丁绕过；下游挂起等我方材料视同
  本地待办跟踪——球在我方不行动 = 断 track。
- 文件/IO 一律走 `KernLab.Tier.Core.IO`（统一高性能 IO 抽象），禁直接 BCL `System.IO` 的
  File/Directory——能力缺失提 Tier issue，禁 BCL workaround 自补。
- 生成器**不得引用它为之生成代码的运行时库**；生成物自足（BCL only、不依赖消费方
  `ImplicitUsings`）；模板随包以 EmbeddedResource 分发。
- 跨仓命名空间**禁靠外层相对解析**，一律显式限定（改名时成片断链）。
- 重运行时三方包：核心链项目零直引——一律**可选包**（不引用 = 能力结构化关闭）或
  **SPI + 协议自研内置**；新增白名单须设计文档登记 + 理由。
- **编译期扩展优先复用 [KernLab.Loom](https://github.com/kernlab-dev/kernlab-loom)**：
  通用面（二进制布局/线协议/命令/端点/DI 的源生成与规则分析）直接消费 Loom 包，
  只有产品域专属的生成器与分析器才留仓内（判例：Tier 通用编译期面已迁 Loom，Tier 为 0 号消费者；
  ID 段位见 [RULES.md](RULES.md)——`KLSG` 归 Loom 通用，`K<产品码>SG` 归产品域专属）。

## 3. 编码纪律

- 代码注释用中文；**注释只写当前契约，不写历史演进/日期**（git 历史才是演进记录）；
  工程注释禁写过程归因。
- 提交信息：conventional commits 中文描述（`feat(scope): …` / `fix(scope): …`）；
  一模块一 commit，改完即编译+测试锁定。
- **fire-and-forget 禁令**：高并发下丢弃任意 Task/ValueTask（`_ = Task.Run(...)`）都是泄漏
  （异常未观测/无背压/生命周期失控）；正确形态 = 专用提交原语（Tier 的 `TaskSink.Submit`）；
  KLSG138 分析器强制（见 [RULES.md](RULES.md) 禁用模式族）。
- 热路径纪律：热路径内禁 LINQ/装箱/string 分配；日志用 `[LoggerMessage]`；
  后台循环/定时/受控后台任务用专用原语，**禁裸 Thread/Task.Run/PeriodicTimer**。
- 请求/管理路径禁 sync-over-async（`GetAwaiter().GetResult()`/`.Wait()`），锁内禁 IO。
- 顶层类型一类一文件，大类 partial 分装；跨边界语义操作禁裸 `Func/Action`（命名委托或结构化接口）。
- 目录名 = 域声明：新目录先答"这是什么域"，名实不符即红。

## 4. 编译与警告红线

- 全解决方案 `TreatWarningsAsErrors`——**警告即错误，0 错误且 0 警告**，tests/benchmarks/probes
  一体适用。编译警告 = 真 bug 信号（资源泄漏 / ValueTask 误用 / 取消令牌未传播 / 可空违约）。
- `NoWarn` 豁免必须带理由，**理由消失即删**；代码本身能改的一律改代码。
- 分析器配置键必须落在 `.globalconfig`（`is_global = true`）或 `.editorconfig` 的节内——
  节头之前的"无节键"被 Roslyn **静默忽略**（规则会长期不执法而不报错）。
- 新增诊断规则两步：产生包内 `AnalyzerReleases.{Shipped,Unshipped}.md` 登记 + 在 [RULES.md](RULES.md)
  补行声明段位；改名时同步 pragma 里的 ID（漏 → 豁免静默失效）。

## 5. 测试纪律

- **测试与源 1:1**：新增源文件必须同批带测试，否则不许合入。
- **并发/队列/锁必须有契约测试**（计数语义/互斥/唤醒协议/无配对操作绊线）；核心隐蔽场景常设
  `#if DEBUG` 仪器（Release 零开销）；**禁用 SKIP 藏并发测试**。
- **Skip 底线**：功能性测试全进 CI；不得以 Skip/删断言/放宽阈值把红改绿。平台差异 ≠ Skip 理由 →
  分平台写各自断言；唯一例外 = 环境前置门控（端点未配置等），跳过项必须显式可见；
  确定性红立 issue（run 链接 + 断言原文 + 平台矩阵 + 取证方向）。
- **单元测试 5 分钟没跑完 = 已卡死**：先取证（dotnet-stack / dotnet-dump）再杀进程，顺序不可倒置；
  跑测试先 build（`--no-build` 跑的是旧二进制）。
- **测试创建的资源与产品资源同红线**：持专用线程/句柄的资源必须失败路径也释放
  （创建即登记 + 统一收尾；共享夹具 setup 失败先收尾再 rethrow）。
- 纯单元（秒级、内存介质）与对抗性测试（真磁盘、时序敏感）分项目，不混跑。
- **遇失败禁回滚逃避**：先分析根因（读异常与堆栈）→ 定点修复 → 验证。

## 6. 审查与结论纪律

- **任何结论先主体验证**；子代理报告/工具扫描结果只当线索不当证据。
- 下"死代码/无用/可删"结论前必读类型头注释，排除：设计档案/对照实验、`[Experimental]`、
  公共 API 原语、并列双版本；**grep 无引用 ≠ 无用**。
- **用组件前先读该组件使用文档全文与反模式表**——遇"反直觉"行为，第一假设是违反文档契约，
  不是组件缺陷。
- **环境归因必须核实前提**：不得以环境标签（磁盘慢/runner 慢）作挂死·慢·flaky 的结论；
  本地能复现的必须本地解决。
- 审查覆盖测试代码：资源释放面沿成功/失败两条路径核到释放。

## 7. 架构与目录

- 生命周期骨架 = 信任边界：恢复/生命周期走骨架基类，业务层只填钩子，禁裸写生命周期协议。
- 项目拓扑依赖单向；每 csproj 必须入 sln（新建当场 `dotnet sln add`，禁游离项目）；
  改 sln 只用 `dotnet sln` 工具，禁手改/正则替换；新建生成物输出目录必须同步 .gitignore。
- 目录契约：`src/`（源码）/ `tests/`（功能测试）/ `benchmarks/`（性能压测）/ `probes/`
  （临时实验，验证完保留或删）/ `docs/`（开发中设计文档）/ `src/*/docs/`（随包公开文档）。

## 8. 文档纪律

- 公开文档 = 使用指南形态：定位 → 快速上手（可运行）→ 概念 → 怎么选 → 怎么用 → 反模式 →
  深入链接；禁历史背景叙事、禁"优化/维护"过程叙事（性能文档只留环境/配置/当前数字/复现）。
- 写公开文档先对照现行源码核实 API；挪文件回查引用且全文通读（防形态/数字语义漂移）。
- 统一文档站 `docs.kernlab.dev/<product>/`（Cloudflare Pages，聚合仓自动部署）：
  产品仓构建产物上传 artifact（约定名 `docs-site`）→ 触发 `kernlab-dev/kernlab-docs` 聚合。
  **子路径契约：全相对链接，禁根绝对路径资源引用**（favicon/logo/og:image 会断链）。

## 9. 发布纪律

- 发布由**各仓自己的** `release.yml` 执行：nuget.org Trusted Publishing 校验 OIDC workflow 身份，
  要求 owner/repository/workflow **逐字一致**——reusable workflow 会指向载体仓而被拒，不可集中。
- 可集中化的是 **CI**（build/test 门禁）：放本仓的 reusable workflow / workflow-templates。
- 发布腿跑 GitHub-hosted（`windows-latest`）；自建 runner 只跑测试腿。
- preview 线免门禁、正式线全量测试。
- 包元数据 `PackageProjectUrl` / `RepositoryUrl` **随包发出且已发布版本不可改**——域名/组织类
  迁移必须在下次发版前落地。

## 10. CI 与凭据

- CI 触发：main/dev 自动（push + PR）；其他分支手动 `workflow_dispatch`；时序敏感套件重试 ×2。
- 凭据只从环境变量 / GitHub Secrets 读取——永不入库、永不回显；密钥出现在日志/附件 = 视同泄露，
  立即轮换；专用最小权限账号，禁主账号密钥做日常验证。
- 安全漏洞走私有报告渠道（[SECURITY.md](SECURITY.md)），禁在公开 Issue 披露未修复漏洞。
