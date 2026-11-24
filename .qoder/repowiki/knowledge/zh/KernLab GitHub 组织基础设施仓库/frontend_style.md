## 概述

本仓库在前端样式方面的产出是一个 **DocFx 文档站点模板**（`docfx-template/`），为 KernLab 家族各产品仓库提供统一的静态文档站骨架与视觉风格。它不是业务 Web 应用，而是用于生成 API 参考与引导页面的 DocFx 产物。

## 技术栈与设计决策

- **基础框架**：DocFx v2.x `default` + `modern` 双模板链（`docfx.json` 中 `"template": ["default", "modern"]`），借助其内建的 Bootstrap 5 运行时与 Bootstrap Icons（通过 `bi bi-*` class 引用）。
- **样式语言**：纯 CSS，无 SCSS / PostCSS / Tailwind；主题色值直接硬编码在 CSS。
- **Dark Mode**：基于浏览器/系统自动切换，利用 Bootstrap 的 `[data-bs-theme="dark"]` 选择器在深色下用 200 级亮阶色替换 card 边框和标题颜色。
- **设计意图**：注释反复标注“设计决策”，强调 Landing Page 卡片禁用 anchorjs 锚点、禁止外部链接伪元素图标以避免视觉干扰；左侧 TOC 侧栏从默认 360px 收紧到 280px；footer 居中排版符合中文站习惯。

## 核心文件

| 文件 | 职责 |
|---|---|
| `docfx-template/main.css` | 所有品牌化样式：响应式导航、landing hero、card 柔和色阶（light/dark）、TOC 宽度、footer 布局 |
| `docfx-template/index.md` | Landing 页面模板桩，定义 hero、特性三卡、文档导航区、快速链接区的 Bootstrap 5 结构与文案占位符 |
| `docfx-template/docfx.json` | DocFx 构建配置，注册 `main.css → public/main.css`（资源注入点），声明模板链与全局元数据 |

## 架构约定

1. **样式注入路径**：在 `docfx.json` 的 `build.resource.files` 中将 `main.css` 复制到 `_site/public/main.css`，页面以相对路径加载。注释要求“子站入口使用固定注入 public/main.css，禁根绝对路径”（坑 #3）。
2. **颜色体系**：未引入设计令牌（tokens）或变量集；Light 模式采用 Bootstrap 调色板 300 级柔化（如 `#748ffc`、`#69db7c`、`#ffa94d`、`#99e9f2`、`#ffa8a8`、`#adb5bd`），Dark 模式改用 200 级亮色以保证深色对比度（如 `#a5d8ff`、`#b2f2bb`、`#ffec99`、`#c5f6fa`、`#ffd5d5`、`#dee2e6`）。卡片标题 `.h5.text-{color}` 与其同规则表一一对应。
3. **断点策略**：仅两组媒体查询——`@media (max-width: 1399.98px)` 缩小搜索框至 6rem，`@media (max-width: 1279.98px)` 隐藏搜索表单给菜单腾空间（配合 navbar 最多 8 项一级菜单的设计决策）。
4. **Landing 页面结构**：`index.md` 规定固定的区块顺序与 DOM 类名：
   - hero 使用 `.landing-hero` 渐变背景 + 圆角。
   - 特性卡统一 `<i class="bi bi-<图标>">` + `text-primary/success/warning` 轮换。
   - “文档导航”、“快速链接”区块要求完整复制列出的 HTML 模式，颜色按 primary → success → warning → info → secondary → danger 次序接续。
5. **Dark Mode 钩子**：所有品牌化规则同时提供 `[data-bs-theme="dark"]` 变体，不依赖 JS 切换逻辑。
6. **组件库限制**：仅使用 Bootstrap 5 的 Utility/Component class 及 Bootstrap Icons；不引用 React/Vue/Angular 等组件库。

## 观察到的约束（规则层级）

- **约束来源**：`docfx.json` 的注释明确标记多处“坑 #N”为协作契约（坑 #1 构建清单同步、坑 #3 样式注入路径、坑 #4 禁止 `_appBasePath`）。
- 必须使用 `bootstrap-icons` 的 `bi` 前缀类来插入图标（由 `index.md` 注释中的 `bi bi-<图标>` 占位与配色说明强制）。
- Landing 卡片标题与边框颜色必须遵循主 CSS 中已定义的柔和色阶（`#748ffc` 等），而非直接使用 Bootstrap 原色，注释写明“避免 Light 下饱和度过高刺眼”。
- TOC 侧栏宽度被 CSS 锁定为 `max-width: 280px !important`，不得随意改宽。
- Footer 内容必须居中对齐，链接继承主体色且 hover 才显示下划线。
- Dark/Light 两套卡片左边框与标题色必须成对维护（见 `main.css` 中每行浅色后紧跟一行 `[data-bs-theme="dark"]` 对应规则）。

## 适用性判断

本仓库作为组织基础设施，不含业务 Web UI，前端样式全部集中在 `docfx-template/` 这一个 DocFx 模板站点中，覆盖范围小而清晰。