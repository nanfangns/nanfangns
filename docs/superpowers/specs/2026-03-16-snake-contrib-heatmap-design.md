# 2026-03-16 Snake Contribution Heatmap (GitHub Profile README) — Design

## 1. Goal / 背景
为 GitHub Profile README（仓库：`nanfangns/nanfangns`）新增“贪吃蛇贡献热力图（Snake）”效果，用于展示贡献活跃度，并通过 GitHub Actions **每天自动更新**。

用户明确偏好：
- 输出形式：**SVG 动画**
- 产物存储：**`output` 分支存图**（避免污染 `main` 历史）
- 更新频率：**每天**
- README 展示位置：**在“📊 GitHub Insights”区块下方**
- 主题适配：**自动切换 light/dark**

成功标准：
- `main` 分支 README 中能正常显示 snake 动画
- `output` 分支自动生成并更新 SVG（light/dark 各一份）
- Actions 定时运行成功（可手动触发）

## 2. Non-goals / 不做什么
- 不修改其他仓库，不做跨仓库资产托管
- 不引入额外站点（如 Pages）作为强依赖（除非后续需要）
- 不追求每小时级更新（避免不必要的资源与队列占用）

## 3. Recommended Approach / 方案选择
选择 **方案 A（推荐）**：使用开源 GitHub Action 生成 snake SVG，并推送到 `output` 分支。

## 4. Architecture / 架构与数据流

### 4.1 分支策略
- `main`：只存 README 与工作流配置。
- `output`：只存生成产物（SVG）。该分支由 Actions 自动更新。

### 4.2 产物路径（在 output 分支）
- `dist/github-contribution-grid-snake.svg`
- `dist/github-contribution-grid-snake-dark.svg`

### 4.3 README 引用策略（自动切换主题）
使用 `<picture>` + `prefers-color-scheme`：
- dark 主题展示 `...-dark.svg`
- 其他展示 `... .svg`

引用 URL（raw）：
- `https://raw.githubusercontent.com/nanfangns/nanfangns/output/dist/github-contribution-grid-snake.svg`
- `https://raw.githubusercontent.com/nanfangns/nanfangns/output/dist/github-contribution-grid-snake-dark.svg`

**README 可直接粘贴片段：**

```html
<!-- Snake contribution graph (auto light/dark) -->
<picture>
  <source media="(prefers-color-scheme: dark)"
          srcset="https://raw.githubusercontent.com/nanfangns/nanfangns/output/dist/github-contribution-grid-snake-dark.svg" />
  <img alt="github contribution grid snake animation"
       src="https://raw.githubusercontent.com/nanfangns/nanfangns/output/dist/github-contribution-grid-snake.svg" />
</picture>
```

## 5. GitHub Actions 设计

### 5.1 Workflow 文件
- 路径：`.github/workflows/snake.yml`

### 5.2 触发
- `schedule`：每天一次，**建议避开 :00 / :30**（减少全网拥堵排队）。示例：`23 1 * * *`（UTC 01:23）。
- `workflow_dispatch`：允许手动触发（用于首次初始化/排障）

### 5.3 权限
需要能向 `output` 分支 push：
- `permissions: contents: write`

并要求仓库 Settings → Actions → General → Workflow permissions 允许 “Read and write”（否则 push 会失败）。

### 5.4 使用的 action（锁版本）与输入/输出
使用 Platane/snk 系列方案生成 SVG（实现时需锁定到一个明确的 major 版本），并确保输出文件名与第 4.2 节一致：
- 输出（light）：`dist/github-contribution-grid-snake.svg`
- 输出（dark）：`dist/github-contribution-grid-snake-dark.svg`

> 注：具体 `uses:` 与 inputs 字段名以实现时选定的 action 为准，但必须满足上述输出路径与文件名约束。

### 5.5 output 分支创建与分支保护
- 首次运行需要能创建并 push `output` 分支（由 workflow 的 push 步骤负责）。
- `output` 分支不建议启用严格保护规则（例如禁止直接 push / 必须走 PR），否则自动更新会失败。

## 6. README 修改点
在 `README.md` 的 `## 📊 GitHub Insights` 区块中：
- 放在现有两张统计卡片之后
- 放在下一条 `---` 分隔线之前
- 插入第 4.3 节的 `<picture>` 片段

## 7. Edge cases / 风险与处理
- output 分支不存在：首次运行由 workflow 自动创建并推送。
- raw 链接缓存：GitHub 对 raw 有缓存，更新可能延迟几分钟，属正常。
- Actions 权限不足：检查 workflow permissions 是否为 read/write。
- output 分支保护：若启用保护导致 push 失败，需要放宽或为 Actions 开例外。

## 8. Test plan / 验证方式
- GitHub：
  1) 手动触发 workflow（workflow_dispatch）
  2) 确认 `output` 分支出现 `dist/` 与两个 svg
  3) 直接打开 raw 链接确认 200：
     - `.../output/dist/github-contribution-grid-snake.svg`
     - `.../output/dist/github-contribution-grid-snake-dark.svg`
  4) 打开 `https://github.com/nanfangns/nanfangns` 查看 README 是否显示蛇图
  5) 切换 GitHub 主题（dark/light）确认自动切换

## 9. Rollback / 回滚
- 删除 README 中 snake 引用块
- 删除 `.github/workflows/snake.yml`
- 可选：删除 `output` 分支（如果不再需要）
