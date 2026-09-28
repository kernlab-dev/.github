---
title: <产品名> — <一句话定位>
description: <SEO 描述：产品身份 × 两三个支柱关键词。>
---

<div class="landing-hero text-center py-5 mb-4">
  <h1 class="display-4 fw-bold mb-3"><产品名></h1>
  <p class="lead text-secondary mb-4"><一句话定位——支柱关键词用 · 分隔></p>
  <div class="d-flex justify-content-center gap-3 flex-wrap">
    <a class="btn btn-primary btn-lg px-4" href="docs/<首分类>/<入口页>.html">开始阅读</a>
    <a class="btn btn-outline-secondary btn-lg px-4" href="https://github.com/kernlab-dev/<产品仓名>">GitHub</a>
  </div>
</div>

<div class="row row-cols-1 row-cols-md-3 g-4 mb-5">
  <!-- 三张特性卡：产品三大支柱。图标用 bootstrap-icons（bi-*）；
       色阶 text-primary/success/warning 轮换（main.css 已转柔和色阶，勿用原色定制） -->
  <div class="col">
    <div class="card h-100 border-0 shadow-sm">
      <div class="card-body">
        <div class="h1 text-primary mb-3"><i class="bi bi-<图标>"></i></div>
        <h3 class="h5"><特性一></h3>
        <p class="text-secondary"><一句话说明></p>
      </div>
    </div>
  </div>
  <div class="col">
    <div class="card h-100 border-0 shadow-sm">
      <div class="card-body">
        <div class="h1 text-success mb-3"><i class="bi bi-<图标>"></i></div>
        <h3 class="h5"><特性二></h3>
        <p class="text-secondary"><一句话说明></p>
      </div>
    </div>
  </div>
  <div class="col">
    <div class="card h-100 border-0 shadow-sm">
      <div class="card-body">
        <div class="h1 text-warning mb-3"><i class="bi bi-<图标>"></i></div>
        <h3 class="h5"><特性三></h3>
        <p class="text-secondary"><一句话说明></p>
      </div>
    </div>
  </div>
</div>

<!-- ── 文档导航（必备区块）────────────────────────────────────────────
     顶导全分类各一卡，覆盖完整导航面。形态铁律（与 Tier/Traffic 逐类同款）：
     · 整卡可点：<a class="text-decoration-none" href=...> 包住整张卡
     · 卡体：card h-100 border-start border-4 border-{色} shadow-sm
     · 标题：h3.h5 text-{同色}（bi 图标 + me-2）；描述：p.text-secondary.mb-0（一词一顿 · 分隔）
     · 色轮换序：primary → success → warning → info → secondary → danger（后继续接 primary）
     · 标题固定：<h2 class="h4 border-bottom pb-2 mb-4">文档导航</h2> -->
<h2 class="h4 border-bottom pb-2 mb-4">文档导航</h2>

<div class="row row-cols-1 row-cols-md-2 g-4 mb-5">
  <div class="col">
    <a class="text-decoration-none" href="docs/<分类A>/<入口页>.html">
      <div class="card h-100 border-start border-4 border-primary shadow-sm">
        <div class="card-body">
          <h3 class="h5 text-primary"><i class="bi bi-<图标> me-2"></i><分类A></h3>
          <p class="text-secondary mb-0"><子页 · 子页 · 子页></p>
        </div>
      </div>
    </a>
  </div>
  <div class="col">
    <a class="text-decoration-none" href="docs/<分类B>/<入口页>.html">
      <div class="card h-100 border-start border-4 border-success shadow-sm">
        <div class="card-body">
          <h3 class="h5 text-success"><i class="bi bi-<图标> me-2"></i><分类B></h3>
          <p class="text-secondary mb-0"><子页 · 子页 · 子页></p>
        </div>
      </div>
    </a>
  </div>
  <!-- ↑ 按顶导分类数复制整行 col——每个分类一卡，不遗漏；色按轮换序接续 -->
</div>

<!-- ── 快速链接（必备区块）────────────────────────────────────────────
     三卡形态：最常用入口（上手捷径/对外契约文档/仓库）。卡体同文档导航，
     网格 row-cols-md-3。标题固定：<h2 class="h4 border-bottom pb-2 mb-4">快速链接</h2> -->
<h2 class="h4 border-bottom pb-2 mb-4">快速链接</h2>

<div class="row row-cols-1 row-cols-md-3 g-4 mb-5">
  <div class="col">
    <a class="text-decoration-none" href="docs/<上手捷径页>.html">
      <div class="card h-100 border-start border-4 border-primary shadow-sm">
        <div class="card-body">
          <h3 class="h5 text-primary"><i class="bi bi-lightning-charge me-2"></i><捷径入口></h3>
          <p class="text-secondary mb-0"><步骤词 · 步骤词 · 步骤词></p>
        </div>
      </div>
    </a>
  </div>
  <div class="col">
    <a class="text-decoration-none" href="docs/<契约或参考页>.html">
      <div class="card h-100 border-start border-4 border-success shadow-sm">
        <div class="card-body">
          <h3 class="h5 text-success"><i class="bi bi-braces me-2"></i><契约/参考入口></h3>
          <p class="text-secondary mb-0"><关键词 · 关键词></p>
        </div>
      </div>
    </a>
  </div>
  <div class="col">
    <a class="text-decoration-none" href="https://github.com/kernlab-dev/<产品仓名>">
      <div class="card h-100 border-start border-4 border-warning shadow-sm">
        <div class="card-body">
          <h3 class="h5 text-warning"><i class="bi bi-github me-2"></i>GitHub 仓库</h3>
          <p class="text-secondary mb-0">源码 · Issues · Pull Requests · Releases</p>
        </div>
      </div>
    </a>
  </div>
</div>

<!-- 可选收尾行：家族底座/生态链一句话（Tier 被消费类产品用）——
     <p class="text-center text-secondary">…</p> -->
