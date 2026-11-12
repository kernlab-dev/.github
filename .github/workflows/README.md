# Workflows 注册表（组织 reusable）

> **本仓不放任何产品仓的入口**——入口在各产品仓（现行 `ci.yml`，单入口迁移后 `gate.yml`）。
> 本目录只提供被产品仓入口 `uses:` 引用的 reusable 节点实现。
> 命名/分流规范：[CI-GATE.md §3/§4](https://github.com/kernlab-dev/.github/blob/main/CI-GATE.md)

## 现行标准（`reusable-<节点码>.yml`）

| 文件 | 节点码 | 职责 | 消费层 |
|---|---|---|---|
| `reusable-e10-policy.yml` | e10 | 强校验：来源分支矩阵 / Tag↔版本 / CHANGELOG / 分支可达（**唯一合法红**） | 入口层 |
| `reusable-e20-triage.yml` | e20 | 分诊：PR 全量 diff → `docs_only` 路由信号（恒跑恒成功） | 入口层 |
| `reusable-v10-ci.yml` | v10 | 完整门禁：build + 全量测试（缺省 sln 自动发现 + tests glob 顺序跑） | 验证层 |
| `reusable-v30-docs.yml` | v30 | docs 质量门：改动 md/mdx 站内链接解析 | 验证层 |
| `reusable-m10-docs-merge.yml` | m10 | docs 自动合并：竞态复核 + squash + 删临时分支 | 汇合层 |
| `reusable-r10-release.yml` | r10 | 发布管线：build→pack→publish | 交付层 |
| `reusable-w10-docs-deploy.yml` | w10 | 文档站：docfx 构建→聚合→上站 | 交付层 |

## 兼容旧名（各仓 gate 迁移后删除，勿新增引用）

| 旧名 | → 新名 |
|---|---|
| `reusable-policy-check.yml` | `reusable-e10-policy.yml` |
| `reusable-docs-triage.yml` | `reusable-e20-triage.yml` |
| `reusable-ci.yml` | `reusable-v10-ci.yml` |
| `reusable-docs-check.yml` | `reusable-v30-docs.yml` |
| `reusable-docs-merge.yml` | `reusable-m10-docs-merge.yml` |
| `reusable-release.yml` | `reusable-r10-release.yml` |
| `reusable-docs-deploy.yml` | `reusable-w10-docs-deploy.yml` |

> 例外：`kernlab-loom` 的 docs-deploy / release 为钉 SHA 审计引用，不受本表影响。

## workflow-templates/

新建仓模板（现行 ci/policy/docs-deploy 三件套）——单入口迁移后将替换为 `gate.yml` 模板。
