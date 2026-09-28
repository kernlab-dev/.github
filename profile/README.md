# KernLab

**可组合的 .NET 底层家族** —— 从存储内核到流量平台，全部自研、零黑盒依赖、可嵌可组合。

*Composable .NET infrastructure family — from a storage kernel to a full traffic platform.
Self-built, zero black-box dependencies, embeddable and composable.*

## 产品矩阵

| 产品 | 定位 | 文档 | 包 |
|---|---|---|---|
| **KernLab.Tier** | 字节存储内核：KV / WAL / Blob / Queue / TimeSeries——零反射、NativeAOT 就绪；编译期序列化消费 KernLab.Loom，域专属生成器（RingKey / KvStore / TierFs）仓内自持 | [docs.kernlab.dev/tier](https://docs.kernlab.dev/tier/) | [nuget.org · 预览版已发布](https://www.nuget.org/packages?q=KernLab.Tier) |
| **KernLab.Synod** | Raft 强一致分布式 K-V：多副本表决，统一状态视图 | 筹备中 | 筹备中 |
| **KernLab.Cohort** | 分布式开发框架底座：注册中心 + 配置中心，编程模型＝MiniAPI + CQRS + 源生成端点 + 编译期切面 | 筹备中 | 筹备中 |
| **KernLab.Claim** | 统一认证中心：验证身份、签发身份声明，全平台唯一信任源 | 筹备中 | [nuget.org · 预览版已发布](https://www.nuget.org/packages?q=KernLab.Claim) |
| **KernLab.Traffic** | 全协议统一流量平台：单端口多协议复用、指令流编排管线、原生多租户、CP/DP 分离集群 | 筹备中 | 筹备中 |
| **KernLab.Loom** | 编译期层：源生成器 + 规则分析器 + 构建骨架（协议/布局/端点/装配编译期织入）——Tier 为 0 号消费者 | [docs.kernlab.dev/loom](https://docs.kernlab.dev/loom/) | [nuget.org · 预览版已发布](https://www.nuget.org/packages?q=KernLab.Loom) |
| **KernLab.Cell** | 运行时容器：一租户一容器的进程隔离与池化，OS 级资源边界 | — | 筹备中 |

产品门户：[kernlab.dev](https://kernlab.dev) · 统一文档站：[docs.kernlab.dev](https://docs.kernlab.dev)

## 工程立场

- **纯托管自研内核**：零 Native 黑盒依赖，无第三方存储引擎与中间件。
- **编译期优先**：协议、内存布局、端点、DI 装配由源生成器在编译期织出；运行期零反射、稳态零分配、AOT 就绪。
- **零警告红线**：全解决方案 `TreatWarningsAsErrors`——警告即错误，`NoWarn` 豁免必须带理由。
- **可审计**：每个仓一份 `AGENTS.md` 作为唯一规则源；判例与缺陷台账在 GitHub Issues。

## 交流渠道

唯一渠道是 GitHub Issues / Discussions——无微信、无邮箱、不接受私下咨询。
答疑为自愿行为，不承诺回复时效，部分问题可能直接忽略或关闭。

## 安全

漏洞请走私有报告渠道，见 [SECURITY.md](SECURITY.md)。**不要在公开 Issue 里披露未修复的安全问题。**

## License

MIT © 2026 kernlab.dev
