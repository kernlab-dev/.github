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
| `KLSG` | `KernLab.Loom.*`（家族编译期层） | 001-099 生成器／100-157 各面增量（含 130-139、147-154 规则分析器）／200-242 Ef 扩展（243-249 预留）／250-259 Http | 通用面，全家族复用 |
| `KTSG` | `KernLab.Tier.CodeGen(.Analyzers)` | 010-014 / 020-023 / 040-044 生成器／120-122 分析器 | Tier 域专属 |
| `COH` | `KernLab.Cohort.CodeGen` | 001-005 | Cohort 自有（品牌中性；后续如按 `<K><产品码>SG` 统一可改 `KCSG`） |

**段位分配纪律**：同一包内**生成器段与分析器段分离**——Tier 判例：TierFs 分析器占 1xx 段以避开 RingKeyGenerator 的 020 段（曾撞号）。新增规则前先在本页查空段。

## 逐包清单

### KernLab.Loom（前缀 `KLSG`）

| ID 段 | 包 | 规则 |
|---|---|---|
| KLSG001-013 | `KernLab.Loom.Layout` | `[BinaryLayout]` 标注用法／字段形态／布局／校验声明／嵌套／生成失败（001-006）／位域族（007-013） |
| KLSG045-049 | `KernLab.Loom.Wire` | `[WireArray]` 标注用法／元素类型／声明形态／生成失败（045-048）／**049 `[WireField]`/`[WireMember]` 宿主非法或死旋钮（非消息族、非 Evolvable 上的 Id）** |
| KLSG050-053 | `KernLab.Loom.Wire` | `[WireMessage]` 标注用法／成员形态／tag 冲突／生成失败 |
| KLSG054-067 | `KernLab.Loom.Cli` | 命令路径冲突／非法命令名／参数未标注／不可绑定形态／GET 带体／标注宿主非法／保留前缀／嵌套组未接线／参数类型不在绑定支持集／`loom_cli.*` 配置键非法（fail-fast 回退缺省）／描述资源形态不完整（DescriptionKey 与 ResourceType 只给其一）／ResourceType 缺可访问 ResourceManager（public 或同程序集 internal——帮助本地化回退链）／066 Confirm 自动 `--yes` 与用户选项冲突／067 `[CommandGroup(Options)]` 声明非法 |
| KLSG130-139, 147-154 | `KernLab.Loom.Analyzers` | 分层引用族（130-133）／禁用模式族（134-138 + 152 零拷贝丢失）／配置合法性 fail-fast（139）／**契约真值表族（147-151——[StringContract] 常量串 DSL 五级诊断，TierFsSpecAnalyzer 通用化下沉判例）**／迁移指引族（153-154——Obsolete 过时面家族定向提示） |
| KLSG100-101 | `KernLab.Loom.Display` | `[EnumDisplay]` 解析器（100 标注用法／101 生成失败） |
| KLSG110-112 | `KernLab.Loom.Query` | `[Filterable]` 字段目录（110 标注用法／111 形态／112 生成失败） |
| KLSG140-146、155-157 | `KernLab.Loom.Validation` | T1 校验段（140 标注用法／141-143 形态族／144 生成失败／145-146 增量）／**155-157 交叉字段与条件必填（155 比较目标无效／156 两侧类型不可比较／157 标志无效）** |
| KLSG200-249 | `KernLab.Loom.Ef` | **EF 扩展段（预留 50 位，2026-10-10 迁段判例：原 120-144 与 Analyzers/Validation 三方撞号；250-259 划归 Http）**——已用 200-242：实体声明族（200-208/210）/ 逃逸舱双闸（211/214）/ 投影族（216-219）/ 表名（220）/ 监听（221）/ 动态视图（222-223）/ 壳声明（224）/ 冻结模型与优化（225-242） |
| KLSG250-259 | `KernLab.Loom.Http` | HTTP 客户端生成器：250 空客户端契约／251 路由模板与参数不匹配／252 契约类型不在支持集／253 请求头声明非法／254 请求体绑定非法／255 参数形态不支持／256 表单绑定非法／257 接口方法形态非法（含继承二义）／258 未提供 Service/BaseAddress／259 `[HttpApi]` 记录形态非法 |

`KernLab.Loom.Analyzers` 配置键：`loom_layer.forbidden_reference`、`loom_layer.zero_internal_refs`、`loom_layer.internal_assembly_prefix`、`loom_layer.allowed_reference`、`loom_layer.namespace_prefix`、`loom_layer.allowed_framework_prefix`、`loom_forbidden.pack`（`reflection`／`sync_over_async`／`fire_and_forget`／`bare_threads`／`hotpath_discipline`／`zero_copy_loss`）。受控豁免 FQN 前缀：`loom_forbidden.exempt.bare_threads`／`loom_forbidden.exempt.sync_over_async`／`loom_forbidden.exempt.fire_and_forget`。契约真值表族无配置键——`[StringContract]` 标注驱动。

**两面规则（已落地——设计稿 [endpoints-di-design](https://github.com/kernlab-dev/kernlab-loom/blob/main/docs/design/endpoints-di-design.md)）**

| ID 段 | 包 | 规则 |
|---|---|---|
| KLSG070-079 | `KernLab.Loom.Endpoints` | 070 契约缺标记接口／071 路由重复／072 契约无端点配对（消费者工程可关）／073 切面未实现 IAspect（此四条自 Cohort `COH001-005` 迁入）／074 契约结果类型与端点返回类型不一致／075 端点方法形态非法／076 route 路径参数与契约成员不匹配／077 `[FromLastEventId]` 用于非 SSE 端点或参数类型非 string／**078 `LoomEndpoints*` MSBuild 配置键非法（原预留槽改配——配置错误绝静默）**；079 切面声明不可解析（派生切面特性无法确定切面类型/实参非编译期常量——切面参数化配套）；**240 序列化底座不支持类型（越段实况：实配于 Ef 预留块 200-259 内——Endpoints 无空段可取的既成选择，#62 入档；Ef 增量触及时双方对齐）；241 端点参数无法绑定** |
| KLSG080-099 | `KernLab.Loom.Di` | 080 captive dependency／081 依赖未登记／082-083 多实现无 Default·多 Default／084 `[SingleImplementation]` 被违反／085 标注误用／086 严格档注册面越界／087 登记未消费（W）／088 依赖环／089 Transient 实现 IDisposable（W）／090 Singleton 注入 IServiceProvider（W）／091 约定表违规（W）／**092 `loom_di.*` 配置键非法（原预留槽改配——配置错误绝静默）**；093 装饰器构造形态非法（装饰器特性无法解析接口/形态不匹配——def8596 装饰器面）；094 装饰目标生命周期不确定（W）；095-099 预留 |

`KernLab.Loom.Endpoints` 配置键（MSBuild 属性，消费方 csproj 直设）：`LoomEndpointsNamespace`、`LoomEndpointsClassName`、`LoomEndpointsConsumerOnly`、`LoomEndpointsServiceLifetime`。
`KernLab.Loom.Di` 配置键（analyzer config——**必须落 `.globalconfig` 或 `.editorconfig` 节内**，无节键静默失效判例）：`loom_di.registration_face`（`interop`｜`declarative`）、`loom_di.unused_registration`（`off`｜`warn`）、`loom_di.lifetime_conventions`（箭头规则表，语法同 `loom_layer.*`）。
`KernLab.Loom.Di` MSBuild 属性：`LoomDiProfile`（编译期档位，分号分隔；`[ContainerService(Condition = ...)]` 与它有交集才发射——未设 = 无活动档位；经包内 `build/KernLab.Loom.Di.props` 暴露给生成器）。

### KernLab.Tier 域专属（前缀 `KTSG`）

| ID 段 | 产生方 | 规则 |
|---|---|---|
| KTSG010-014 | `KernLab.Tier.CodeGen`（TierFsGenerator） | MediumOptions nature／verb、NetworkProtocol key、SpecParam 形态／介质 |
| KTSG020-023 | `KernLab.Tier.CodeGen`（RingKey／KvStoreGenerator） | RingKey 非 unmanaged、KvStore key／value／formatter |
| KTSG040-044 | `KernLab.Tier.CodeGen`（ConstantRegistryGenerator） | 重复值／非法 zone 声明／越界／非 partial／值域歧义 |
| ~~KTSG120-122~~ | ~~`KernLab.Tier.CodeGen.Analyzers`~~ | **已退役（2026-10-19，kernlab-loom #157 下沉判例）**——契约真值表族通用化至 `KernLab.Loom.Analyzers` KLSG147-149（TierFsSpecAnalyzer 已删除，Tier 消费 Loom 分析器） |

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
