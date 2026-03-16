# 2026-03-16 Snake Contribution Heatmap (GitHub Profile README) — Design

## 1. Goal / 背景
为 GitHub Profile README（仓库：`nanfangns/nanfangns`）新增“贪吃蛇贡献热力图（Snake）”效果，用于展示贡献活跃度，并通过 GitHub Actions **每天自动更新**。

用户明确偏好：
- 输出形式：**SVG 动画**
- 产物存储：**`output` 分支存图**（避免污染 `main` 历史）
- 更新频率：**每天**
- README 展示位置：**在“📊 GitHub Insights”区块下方**
- 主题适配：**自动切换 light/dark（推荐）**

成功标准：
- `main` 分支 README 中能正常显示 snake 动画
- `output` 分支自动生成并更新 SVG（light/dark 各一份）
- Actions 定时运行成功（可手动触发）

## 2. Non-goals / 不做什么
- 不修改其他仓库，不做跨仓库资产托管
- 不引入额外站点（如 Pages）作为强依赖（除非后续需要）
- 不追求每小时级更新（避免不必要的资源与队列占用）

## 3. Recommended Approach / 方案选择
选择 **方案 A（推荐）**：使用开源 action 生成 snake SVG，并推送到 `output` 分支。

实现基于社区主流方案（Platane/snk 系列方案）。

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

引用 URL 形式（raw）：
- `https://raw.githubusercontent.com/nanfangns/nanfangns/output/dist/github-contribution-grid-snake.svg`
- `https://raw.githubusercontent.com/nanfangns/nanfangns/output/dist/github-contribution-grid-snake-dark.svg`

## 5. GitHub Actions 设计

### 5.1 Workflow 文件
- 路径：`.github/workflows/snake.yml`

### 5.2 触发
- `schedule`：每天一次（建议固定时间点，如 UTC 0:00；后续可按需微调）
- `workflow_dispatch`：允许手动触发

### 5.3 权限
需要能向 `output` 分支 push：
- `permissions: contents: write`

### 5.4 关键步骤
1) Checkout（main 分支）
2) 生成 snake（指定 `github_user_name: nanfangns`，输出到 `dist/`）
3) 推送产物到 `output` 分支（保留 `dist/`）

## 6. README 修改点
在 `README.md` 的 `## 📊 GitHub Insights` 区块现有两张统计卡片之后、`---` 分隔线之前插入：

- `Snake` 标题或保持无标题（按你现有风格，建议加一行小标题或直接插图）
- `<picture>` block（根据主题切换 light/dark）

## 7. Edge cases / 风险与处理
- 首次运行前 `output` 分支不存在：workflow 需要能创建并推送。
- raw 链接缓存：GitHub 对 raw 有缓存，更新可能延迟几分钟，属正常。
- Actions 权限不足：确保 workflow permissions 允许写入内容。

## 8. Test plan / 验证方式
- 本地（可选）：不需要本地运行。
- GitHub：
  1) 手动触发 workflow（workflow_dispatch）
  2) 确认 `output` 分支出现 `dist/` 与两个 svg
  3) 打开 `https://github.com/nanfangns/nanfangns` 查看 README 是否显示蛇图
  4) 切换 GitHub 主题（dark/light）确认自动切换

## 9. Rollback / 回滚
- 删除 README 中 snake 引用块
- 删除 `.github/workflows/snake.yml`
- 可选：删除 `output` 分支（如果不再需要）
