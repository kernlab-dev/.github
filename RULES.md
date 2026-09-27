# KernLab 诊断注册表（家族编译期扩展）

> 一页看全家族**编译器扩展产生的诊断**（源生成器 + 规则分析器）：谁产生、ID 段位、怎么配置、在哪登记。
> 本页是**家族级单一事实源**；每个包另有自己的 `AnalyzerReleases.*.md`（随 nupkg 分发到包根，可解包查阅）。

## 怎么知道有哪些规则（四个面）

| 面 | 看什么 | 面向 |
|---|---|---|
| 构建输出 | 编译期直接报 ID 与消息（如 `error KLSG136: 禁止运行时反射…`） | 开发者 |
| 包内登记 | 解包任意 nupkg → `AnalyzerReleases.Shipped.md` / `AnalyzerReleases.Unshipped.md` | 包消费者 |
| 本页 | 全家族 ID 段位与归属（下方两张表） | 所有人 |
| 消费方配置 | `.editorconfig` / `.globalconfig` 里 `dotnet_diagnostic.<ID>.severity` | 仓维护者 |

## ID 命名与段位

格式：**`K<层/产品码>SG`** + 三位数字（SG = 编译期扩展诊断，生成器与分析器共用）。

| 前缀 | 产生方 | 段位 | 备注 |
|---|---|---|---|
| `KLSG` | `KernLab.Loom.*`（家族编译期层） | 001-099 生成器／130-139 规则分析器 | 通用面，全家族复用 |
| `KTSG` | `KernLab.Tier.CodeGen(.Analyzers)` | 010-014 / 020-023 / 040-044 生成器／120-122 分析器 | Tier 域专属 |
| `COH` | `KernLab.Cohort.CodeGen` | 001-005 | Cohort 自有（品牌中性；后续如按 `<K><产品码>SG` 统一可改 `KCSG`） |

**段位分配纪律**：同一包内**生成器段与分析器段分离**——Tier 判例：TierFs 分析器占 1xx 段以避开 RingKeyGenerator 的 020 段（曾撞号）。新增规则前先在本页查空段。

## 逐包清单

### KernLab.Loom（前缀 `KLSG`）

| ID 段 | 包 | 规则 |
|---|---|---|
| KLSG001-006 | `KernLab.Loom.Layout` | `[BinaryLayout]` 标注用法／字段形态／布局／校验声明／嵌套／生成失败 |
| KLSG045-048 | `KernLab.Loom.Wire` | `[WireArray]` 标注用法／元素类型／声明形态／生成失败 |
| KLSG050-053 | `KernLab.Loom.Wire` | `[WireMessage]` 标注用法／成员形态／tag 冲突／生成失败 |
| KLSG054-061 | `KernLab.Loom.Cli` | 命令路径冲突／非法命令名／参数未标注／不可绑定形态／GET 带体／标注宿主非法／保留前缀／嵌套组未接线 |
| KLSG130-139 | `KernLab.Loom.Analyzers` | 分层引用族（130-133）／禁用模式族（134-138）／配置合法性 fail-fast（139） |

`KernLab.Loom.Analyzers` 配置键：`loom_layer.forbidden_reference`、`loom_layer.zero_internal_refs`、`loom_layer.internal_assembly_prefix`、`loom_layer.allowed_reference`、`loom_layer.namespace_prefix`、`loom_layer.allowed_framework_prefix`、`loom_forbidden.pack`（`reflection`／`sync_over_async`／`fire_and_forget`／`bare_threads`／`hotpath_discipline`）。

**预留段位（Loom 待建两面——设计稿 [endpoints-di-design](https://github.com/kernlab-dev/kernlab-loom/blob/main/docs/design/endpoints-di-design.md)）**

| 预留段 | 面 | 规划规则 |
|---|---|---|
| KLSG070-079 | `KernLab.Loom.Endpoints` | 070 契约缺标记接口／071 路由重复／072 契约无端点配对（消费者工程可关）／073 切面未实现 IAspect（此四条自 Cohort `COH001-005` 迁入）／**074 契约结果类型与端点返回类型不一致**／**075 端点方法形态非法**／**076 route 路径参数与契约成员不匹配**／**077 `[FromLastEventId]` 用于非 SSE 端点**；078-079 预留 |
| KLSG080-099 | `KernLab.Loom.Di` | 080 captive dependency／081 依赖未登记／082-083 多实现无 Default·多 Default／084 `[SingleImplementation]` 被违反／085 标注误用／086 严格档注册面越界／087 登记未消费（W）／**088 依赖环**／**089 Transient 实现 IDisposable（W）**／**090 Singleton 注入 IServiceProvider（W）**／**091 约定表违规（W）**；092-099 预留 |

### KernLab.Tier 域专属（前缀 `KTSG`）

| ID 段 | 产生方 | 规则 |
|---|---|---|
| KTSG010-014 | `KernLab.Tier.CodeGen`（TierFsGenerator） | MediumOptions nature／verb、NetworkProtocol key、SpecParam 形态／介质 |
| KTSG020-023 | `KernLab.Tier.CodeGen`（RingKey／KvStoreGenerator） | RingKey 非 unmanaged、KvStore key／value／formatter |
| KTSG040-044 | `KernLab.Tier.CodeGen`（ConstantRegistryGenerator） | 重复值／非法 zone 声明／越界／非 partial／值域歧义 |
| KTSG120-122 | `KernLab.Tier.CodeGen.Analyzers` | spec scheme 非法／参数名未知／参数 × 介质违规 |

Tier 域侧不读配置键（全部 Error 默认，声明即报）。

## 配置纪律（都是踩过的坑）

1. **键必须落在 `.globalconfig`（`is_global = true`）或 `.editorconfig` 的节内。** 写在 `.editorconfig` 任何节头**之前**的"无节键"被 Roslyn **静默忽略**——Tier 的家族规则曾因此长期不执法（2026-09-27 实测：无节键 0 命中／移入 `[*]` 节 2 命中）。
2. **严重级按路径分段声明**：生产代码 `error`，测试／探针／基准可 `none`（同步测试等 Task 完成是测试惯用法）。
3. **受控豁免用 `#pragma warning disable <ID>` 并注明理由**（治理原语内核的同步等待、受控丢弃属此类）。**改名时必须同步 pragma 里的 ID**——否则豁免静默失效、规则重新报警（Tier 判例：22 个文件的 pragma 未随改名更新）。
4. **新增规则两步**：①产生包内 `AnalyzerReleases.{Shipped,Unshipped}.md` 登记（RS2008 门；RS2007 非确定性误报可带理由豁免）；②本页补一行并声明段位。
5. **登记对账自动护栏**：`KernLab.Loom` 测试门校验"源码实际 ID ↔ 登记表"一一对应、跨面不撞号、ID 必为 `KLSG`+三位数字（判例：Loom 早期四份登记被覆盖回 Tier 内容、Cli 段格式化笔误出过 `KLSG0554`）。产品仓域专属诊断建议照建同款门。

## 相关

- 生成器开发 SDK 与约定：[KernLab.Loom](https://github.com/kernlab-dev/kernlab-loom)
- 面向包消费者的工程口径：[SECURITY.md](SECURITY.md) ／ [SUPPORT.md](SUPPORT.md)
