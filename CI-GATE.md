# KernLab CI 单入口分流设计（Workflow Gate）

> **版本**：0.1.0（设计稿——待评审，未生效）
> **日期**：2026-11-12
> **适用范围**：组织内全部代码仓（产品仓、支撑仓）
> **关联**：[BRANCHING.md](https://github.com/kernlab-dev/.github-private/blob/main/BRANCHING.md) §6/§6.2 · [CI-INFRA.md](https://github.com/kernlab-dev/.github-private/blob/main/CI-INFRA.md) §1.5/§2/§10 · 判例链 §9
> **变更记录**：0.1.0 初稿。回应三判例：双入口语义混淆、success() 传递 skip 空壳绿（tier#1417）、非法来源照烧重门禁。

---

## 1. 背景与问题

| # | 问题 | 实锤 |
|---|---|---|
| P1 | PR 触发两个 workflow（CI + Policy）→ 双状态检查、运行列表噪音、"为什么跑两个 CI"的语义混淆 | tier PR 面板：CI#1230 + Policy#513 |
| P2 | `success()` 对 needs DAG **传递求值**——入口 job 被 skip 后下游整链空壳绿 | tier#1417（build-test-pack skipped 但 run 绿）、traffic#382（编译/单测/SDK 契约/kind E2E 全空壳） |
| P3 | Policy 独立于重门禁——非法来源分支的 PR 照烧 30 分钟 linux-h | 设计缺口，非判例 |

## 2. 三个概念问题的回答（本设计的地基）

**Q1：一个入口怎么做分流？**
Job 级 `if` 条件路由。入口 job 产出路由信号（事件类型、来源分支、内容分诊 `docs_only`），下游每个 job 用 `if` 声明自己为什么跑、为什么不跑。分流逻辑单点可见，不再散落在多个 workflow 文件的触发器里。

**Q2：多 job 能并行/串行吗？**
`needs` DAG 原生支持。无 `needs` 的 job 并行 fan-out（如 policy ∥ triage 秒级并行）；`needs` 串行收口（如 build → test → e2e）；`needs` 多父 fan-in（如 docs-merge 需要 [policy, triage, docs-check] 全绿）。并行不占同一台机器——并行度只受资源池并发约束（§5）。

**Q3：多台 CI 服务器怎么参与？**
`runs-on` 标签 → 资源池（CI-INFRA §1.5 分派表）：`linux-h` = pve 三机轮转（12G 重门禁）、`linux-l` = ci-h-3/腾讯云（轻量）、`ubuntu-latest`/`windows-latest` = GitHub 托管（秒级/matrix/文档）。同一个 workflow 里不同 job 天然落不同机器——单入口不改变分派语义。

## 3. 设计目标

1. **单入口**：每仓 PR 门禁只有一个 workflow、一个状态检查、一次运行。
2. **强校验前置**：非法来源分支秒级红，重门禁短路（不烧资源池）。
3. **规则分流**：docs 快速通道 / 完整 CI / main 重线 / nightly 专项由事件+条件路由，规则单表可见（§4）。
4. **关键路径隔离**：release（tag 发布）与 docs-deploy（文档站）保持独立文件——门禁文件的语法错误不波及发版能力。
5. **单源收口**：全部分流与门禁逻辑在 dotgithub reusable；仓内只剩薄桩（触发块 + 参数 + 仓专属扩展 job）。
6. **传递 skip 防线**（CI-INFRA §9 判例③的制度化）：入口 job 恒跑；扩展 job 一律显式条件。

## 4. 分流规则总表（唯一权威表）

| 事件 | 来源分支 | 内容分诊 | 路由 | runner |
|---|---|---|---|---|
| PR → dev | `docs/*` | 纯文档 | policy ✓ → docs-check → **自动 squash 合并 + 删分支** | ubuntu-latest |
| PR → dev | `docs/*` | 夹代码 | 完整 CI（triage 判 false） | linux-h 等 |
| PR → dev | `feat/* fix/* chore/* hotfix/* main` | 任意 | 完整 CI（双条件门缺分支条件，不进快速通道） | linux-h 等 |
| PR → dev | 非法来源 | — | **policy 红 → 全部短路**（run 秒级红，不占池） | ubuntu-latest |
| PR → main | `dev` / `hotfix/*` | 任意 | 完整 CI + 发布检查单 + release owner 人工批准（永不自动合并） | linux-h 等 |
| push main | — | — | 重线后置防线（AOT 探针等仓扩展）+ docs-deploy（paths 独立文件） | linux-l/linux-h |
| tag `v*` | — | — | `release.yml` 独立文件（自带 tag policy 前置 → build/pack/publish） | 见 release 配置 |
| schedule | — | — | nightly 专项（同文件内 `if: event == schedule` 的 job：模板 smoke/AOT） | linux-h/linux-l |
| dispatch | — | — | 手动 opt-in（win 认证腿、专项验证） | 按输入 |

**双条件门定义**（docs 快速通道，三者同时满足）：来源分支 `docs/*` ∧ 目标 `dev` ∧ diff 纯文档（`.github/**` 恒非 docs）。

## 5. 单入口拓扑（needs DAG）

```
PR / push / dispatch / schedule
        │
        ├─ ① policy   入口强校验（恒跑；唯一合法的红；红 → ②③④⑤ 全短路）
        │     └─ reusable-policy-check（PR 段：来源分支矩阵）
        │
        ├─ ② triage   内容分诊（恒跑恒成功；非 PR 上下文 → docs_only=false）
        │     └─ reusable-docs-triage
        │
        │   ┌── 并行 fan-out（同一 workflow 内，按标签落不同资源池）──┐
        ├── ③ docs-check   if: 快速通道全条件            ubuntu-latest
        ├── ④ ci           if: 非 schedule 且非快速通道  linux-h（reusable-ci 内部 matrix）
        ├── ⑤ 扩展 job…    if: 显式条件（仓专属，逐个声明） linux-h/linux-l/托管
        │   └──────────────────────────────────────────────┘
        │
        └─ ⑥ docs-merge  if: ①②③ 全绿 + 快速通道全条件 + 非 draft
              └─ reusable-docs-merge（竞态复核 + squash + 删分支）
```

并行语义：①②并行秒过；④⑤在③不该跑时并行铺开；③⑥只占 ubuntu 托管秒级。串行语义：⑥等③；Traffic 的 build→test→kind-e2e 链保持串行收口。

## 6. 传递 skip 防线（制度化三原则）

1. **入口 job 恒跑**：`policy` 与 `triage` 不挂任何 `if`。triage 对非 PR 上下文在步骤内返回 `docs_only=false` 成功；policy 的"红"是唯一合法的失败语义（红=非法来源，本来就该全红）。
2. **policy 红即短路**：重门禁条件显式含 `needs.policy.result == 'success'`——非法来源的 PR 不烧 30 分钟 linux-h（**决策点 D1**：短路 vs 照跑，本稿取短路，理由=注定不能合的 PR 无测试价值，修复来源后重触发即得全量）。
3. **下游扩展 job 禁用隐式 `success()`**：凡 `needs` 了入口 job 的自定义 job，一律显式 `if: >- ${{ !cancelled() && needs.<直接父>.result == 'success' && <业务条件> }}`。隐式传递求值是空壳绿根因（判例 CI-INFRA §9 ③①）。

## 7. 关键路径隔离（什么不并入单入口）

| 文件 | 触发 | 不并入的理由 |
|---|---|---|
| `release.yml` | push tag `v*` | 生产发布路径：门禁文件语法错误不得波及发版能力；自带 tag 侧 policy 前置（Tag↔props/CHANGELOG/分支可达） |
| `docs-deploy.yml` | push main + `paths:` | paths 过滤让非文档 push 零 run；并入单入口反而每次 main push 多一个空跑 job |
| 仓专项（Loom bench-nightly / platform-matrix / provider-matrix / watchdog…） | 各自独立语义 | nightly/手动/外部巡检，与 PR 门禁无耦合 |

## 8. 单源 reusable 清单（dotgithub）

| reusable | 消费方 | 状态 |
|---|---|---|
| `reusable-ci.yml` | 单入口 ci job | 已有 |
| `reusable-policy-check.yml` | 入口 policy job（PR 段）+ release.yml（tag 段） | 已有 |
| `reusable-docs-triage.yml` | 入口 triage job | 已有（无条件常跑形态） |
| `reusable-docs-check.yml` | docs-check job | 已有 |
| `reusable-docs-merge.yml` | docs-merge job | 已有 |
| `reusable-docs-deploy.yml` / `reusable-release.yml` | 独立文件 | 已有 |

本设计**不新增 reusable**——只改仓侧桩的拓扑（policy 从独立文件并入入口）。

## 9. 仓内桩形态（目标模板全文）

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main, dev]
  workflow_dispatch:   # 仓专属 inputs（如 Tier include_windows）保留在此

concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read

jobs:
  # ① 入口强校验（恒跑；唯一合法的红；红→下游全短路）
  policy:
    uses: kernlab-dev/.github/.github/workflows/reusable-policy-check.yml@main

  # ② 内容分诊（恒跑恒成功——传递 skip 防线，勿挂 if）
  triage:
    uses: kernlab-dev/.github/.github/workflows/reusable-docs-triage.yml@main

  # ③ docs 快速通道质量门（双条件门收在此处）
  docs-check:
    needs: [policy, triage]
    if: >-
      ${{ needs.policy.result == 'success' && needs.triage.outputs.docs_only == 'true' &&
          startsWith(github.head_ref, 'docs/') && github.base_ref == 'dev' }}
    uses: kernlab-dev/.github/.github/workflows/reusable-docs-check.yml@main

  # ④ 完整 CI（policy 红短路；快速通道让路；schedule 由专项 job 承担）
  ci:
    needs: [policy, triage]
    if: >-
      ${{ !cancelled() && needs.policy.result == 'success' && github.event_name != 'schedule' &&
          !(needs.triage.outputs.docs_only == 'true' &&
            startsWith(github.head_ref, 'docs/') && github.base_ref == 'dev') }}
    uses: kernlab-dev/.github/.github/workflows/reusable-ci.yml@main
    with:
      runner: ubuntu-latest            # 仓按需切 linux-h / runners_matrix
      build_config: Release
      test_projects: 'tests/*/*.csproj'
      test_retries: '1'

  # ⑤ 仓专属扩展 job（模板：一律显式条件，禁用隐式 success()）
  # extension:
  #   needs: [policy, triage, ci]
  #   if: >-
  #     ${{ !cancelled() && needs.policy.result == 'success' && needs.ci.result == 'success' }}
  #   runs-on: linux-h

  # ⑥ docs 自动合并终段（PAT 未配置显式跳过不假绿）
  docs-merge:
    needs: [policy, triage, docs-check]
    if: >-
      ${{ !cancelled() &&
          needs.policy.result == 'success' && needs.triage.outputs.docs_only == 'true' &&
          startsWith(github.head_ref, 'docs/') && github.base_ref == 'dev' &&
          needs.docs-check.result == 'success' &&
          github.event.pull_request.draft == false }}
    uses: kernlab-dev/.github/.github/workflows/reusable-docs-merge.yml@main
    secrets:
      DOCS_MERGE_PAT: ${{ secrets.DOCS_MERGE_PAT }}
```

Tier/Traffic 等带扩展链的仓：扩展 job 组挂在 ④⑤ 形态上（Tier：matrix → build-test-pack；Traffic：关键字/ui-dist → build → test → kind-e2e），条件全部显式化后进入单入口。

## 10. 迁移方案

| 阶段 | 内容 | 状态 |
|---|---|---|
| 0 | 传递 skip 根治（triage 无条件常跑 + 消费端条件） | ✅ 已落地（.github#12 + 7 仓 PR，tier build-test-pack 实证恢复） |
| 1 | 本设计稿评审定稿（决策点 D1-D3） | 🔵 本文 |
| 2 | Tier 试点：policy 并入 ci.yml + 删 policy.yml，验证单 run 形态 | 待 D1-D3 定 |
| 3 | 其余 6 仓机械推广（同 diff 形状）+ workflow-templates 更新 | 待阶段 2 |
| 4 | 将来升级 Team 配 rulesets：required check 仅挂 `CI` 一个 | 远期 |

**回滚**： revert 仓侧桩 PR 即回双文件形态（reusable 层无破坏性变更）。
**Cohort**：随其 #49（unify-ci，在建）落地后按本模板直接接线，不走旧形态。

## 11. 兼容性影响

- PR 状态检查从两个（CI + Policy）变一个（CI，内含 policy job）——将来 rulesets required check 更简单；
- 非法来源 PR 的 CI run 变为秒级红（原为 policy 红照跑全量）；
- 运行列表噪音减半（PR 事件 2 run → 1 run）；
- 并发组不变（`ci-${{ github.workflow }}-${{ github.ref }}`）；单文件后同 ref 单 run；
- `policy.yml` 文件退役：各仓删除（release.yml 的 policy job 不受影响）。

## 12. 决策点（待定）

| # | 决策 | 本稿立场 | 备选 |
|---|---|---|---|
| D1 | policy 红时重门禁短路？ | **短路**（省池；非法来源注定不能合） | 照跑（作者同时看测试信号，烧满全程） |
| D2 | 单入口命名 | 保持 `CI`（状态检查名不变，迁移零心智成本） | 改 `Gate`（语义更准，检查名变化） |
| D3 | docs-deploy 长期归属 | 保持独立（paths 触发零空跑） | 并入（main push 每次多一个守卫 job） |
