# KernLab 家族文档站 landing 骨架——复制到 docs-site/index.md 后填产品内容。
# 结构 = hero 区（landing-hero）+ 特性卡三张 + 分区入口卡网格。
# 样式钩子在家族统一样式 main.css 中（landing-hero / 卡片柔和色阶 / 禁锚点图标）。

<div class="landing-hero text-center py-5 mb-4">
  <h1 class="display-4 fw-bold mb-3"><产品名></h1>
  <p class="lead text-secondary mb-4"><一句话定位></p>
  <div class="d-flex justify-content-center gap-3 flex-wrap">
    <a class="btn btn-primary btn-lg px-4" href="docs/<层>/<入口页>.html">开始阅读</a>
    <a class="btn btn-outline-secondary btn-lg px-4" href="https://github.com/kernlab-dev/<产品仓名>">GitHub</a>
  </div>
</div>

<div class="row row-cols-1 row-cols-md-3 g-4 mb-5">
  <!-- 三张特性卡：图标用 bootstrap-icons（bi-*），色阶 text-primary/success/warning
       （main.css 已转柔和色阶，勿用原色定制） -->
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

<!-- 分区入口卡网格：指向各章节/COORDINATION，参照 Tier index.md 下半部形态 -->
