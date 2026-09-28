# KernLab docfx 标准模板

> 本目录是 KernLab 家族**文档站标准模板**，从 KernLab.Tier 文档站（首个踩坑走通的实例）提炼。
> 目标：新产品文档站**照抄即对**——样式、导航、构建、发布与 docs.kernlab.dev 家族站完全一致，
> 不再重踩 Tier 踩过的坑。
>
> 配套：家族内部发布规则 `.github-private/DOC-PUBLISH.md`（接入五步/校验清单）、
> 编写规则 `.github-private/DOC-GUIDE.md`、模板 `templates/`。

## 文件清单

| 文件 | 用途 | 怎么用 |
|---|---|---|
| [`docfx.json`](docfx.json) | metadata + build 配置骨架 | 复制到产品仓 `docs-site/docfx.json`，按注释改项目清单与产品名 |
| [`main.css`](main.css) | 全家族统一样式（navbar 收紧/landing hero/卡片柔和色阶/TOC 宽度/footer） | 原样复制到 `docs-site/main.css`，**不要改**——家族一致性优先 |
| [`toc.yml`](toc.yml) | 顶部导航骨架 | 复制后按产品章节改名改链接 |
| [`index.md`](index.md) | landing 页骨架（hero + 特性卡 + 文档导航 + 快速链接——Tier/Traffic 同款实态） | 复制后填产品名/简介/卡片，文档导航按顶导分类补全 |
| [`../workflow-templates/docs-deploy.yml`](../workflow-templates/docs-deploy.yml) | CI 模板（构建→artifact→触发聚合） | 新建仓可在 Actions 页直接选用；存量仓复制到 `.github/workflows/` |

## 快速开始（新产品接入）

1. 产品仓建 `docs-site/`，放入上面四个文件并按占位符填写。
2. 产品各发布项目 **Release 构建**（workflow 已含）——docfx metadata 靠 dll 旁的 XML 注释。
3. 配置 `DOCS_SYNC_TOKEN` secret（对 kernlab-docs 有 `actions:write` 的 PAT）。
4. push main 触发 → artifact `docs-site` → kernlab-docs 聚合 → docs.kernlab.dev/`<product>`/ 上线。
5. 按 DOC-PUBLISH.md §8 校验清单过一遍（本地子路径预览必做）。

## 踩坑清单（每一条都真踩过——模板已内置对策）

| # | 坑 | 后果 | 模板对策 |
|---|---|---|---|
| 1 | **metadata 项目清单不全**（曾只构建 Runtime 一个） | 缺哪个项目，哪个产品的 API 参考整层消失，且构建不报错 | docfx.json 注释列全清单；workflow for 循环逐项目构建 `|| exit 1` |
| 2 | **COORDINATION 链接深镜像形态**（`coordination/src/<项目名>/`） | URL 带项目层级，#510 摊平后全变 404 死链 | coordination 固定短名映射（core/core-net/…），对外链接只用短名 |
| 3 | **根绝对路径资源引用**（favicon/logo/og:image 写 `/xxx`） | 部署在 `docs.kernlab.dev/<product>/` 子路径下断链 | 契约：全相对链接；模板全部相对 |
| 4 | **指望 `_appBasePath` 解决子路径** | docfx 2.78 modern 模板下是 no-op（A/B 实测零差异），白配 | 不配该键；子路径兼容靠全相对链接 |
| 5 | **根路径 serve 一眼当验证** | 子路径断链问题只在子路径下暴露，上线才发现 | 验证必须本地挂 `/<product>/` 前缀 + 真浏览器（DOC-PUBLISH §7） |
| 6 | **默认样式未调**（TOC 360px 偏宽/navbar 挤压/卡片原色刺眼） | 阅读体验差，逐像素调过一轮才定型 | main.css 原样带过来，零调整 |
| 7 | **TOC 引用了不存在的文件** | 构建警告刷屏，真问题被淹没 | toc.yml 只引用确实生成的页面；构建警告有已知基线，新增警告必须清零 |
| 8 | **workflow paths 过滤漏文档源** | 文档改了站不更新，排查半天 | workflow `paths` 含 `src/**/docs/**`、`COORDINATION.md`、站点目录、README |
| 9 | **DOCS_SYNC_TOKEN 未配** | dispatch 静默失败或假绿 | workflow 对空 token 显式打日志跳过（不 fail，但日志可见） |
| 10 | **perf 文档平铺**（`docs/runtime/` 根下） | 与使用指南混排，导航混乱 | perf 族统一 `docs/<层>/perf/` 子目录，对外链接对齐该布局 |
| 11 | **首页缺「文档导航/快速链接」区块**（模板旧骨架只有 hero+特性卡，产品自行发挥出弱观感） | 首页与家族站不一致，导航面不全 | index.md 骨架已升级为完整实态（Tier/Traffic 同款彩色描边卡）；footer 同步家族实态 `kernlab.dev · 文档门户` |
| 12 | **聚合仓裸路径 200 重写**（`/product` 无斜杠直吐页面） | 相对 CSS 解析到根 404＝整页裸排 | 聚合仓 `_redirects` 裸路径一律 **301 到带斜杠**（kernlab-docs 判例，全产品生效） |

## 版本与升级

- 样式（main.css）家族统一：改样式 = 先改本模板，再各产品仓同步，禁止产品私自分叉。
- docfx 版本：CI 用 `dotnet tool install -g docfx`（最新 stable）；升级时先在 Tier 仓试跑，
  全家族构建绿后再升模板（跨版本渲染差异真实存在）。
