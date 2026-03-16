# Profile Snake Contribution Heatmap Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在 `nanfangns/nanfangns` 的 Profile README 中展示“贪吃蛇贡献热力图（SVG 动画）”，通过 GitHub Actions 每天自动生成并将产物推送到 `output` 分支。

**Architecture:** GitHub Actions 定时生成 `dist/*.svg`（light/dark），并 push 到 `output` 分支；`main` 分支 `README.md` 使用 raw.githubusercontent 链接引用产物，并通过 `<picture>` 自动适配深浅色主题。

**Tech Stack:** GitHub Actions, Platane/snk 生成器（snake SVG）, git（分支与提交）

---

## File structure / 变更清单

**Create:**
- `.github/workflows/snake.yml`
- `docs/superpowers/plans/2026-03-16-profile-snake-contrib-heatmap.md`（本文件）

**Modify:**
- `README.md`（插入 snake `<picture>` 片段到 “## 📊 GitHub Insights” 区块下）

**Generated (in output branch):**
- `dist/github-contribution-grid-snake.svg`
- `dist/github-contribution-grid-snake-dark.svg`

---

## Chunk 1: Add workflow to generate + publish snake SVG

### Task 1: Add `snake.yml` workflow

**Files:**
- Create: `.github/workflows/snake.yml`

- [ ] **Step 1: Create workflow file**

Create `.github/workflows/snake.yml` with the following full contents:

```yaml
name: Generate snake contribution graph

on:
  schedule:
    # GitHub Actions cron uses UTC.
    # Daily at 01:23 UTC (avoid :00/:30 to reduce queue contention)
    - cron: "23 1 * * *"
  workflow_dispatch:

permissions:
  contents: write

concurrency:
  group: snake-generation
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate snake SVGs
        # NOTE: If you see "Unexpected input(s)" in logs, check the action README
        # and adjust input names accordingly.
        uses: Platane/snk@v3
        with:
          github_user_name: nanfangns
          outputs: |
            dist/github-contribution-grid-snake.svg
            dist/github-contribution-grid-snake-dark.svg?palette=github-dark

      - name: Publish to output branch
        uses: crazy-max/ghaction-github-pages@v4
        with:
          target_branch: output
          build_dir: dist
          keep_history: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

- [ ] **Step 2: Commit workflow**

Run:
```bash
git add .github/workflows/snake.yml
git commit -m "feat: add daily snake contribution graph workflow"
```
Expected: commit succeeds.

- [ ] **Step 3: Push main branch**

Run:
```bash
git push origin main
```
Expected: `main -> main` pushed.

- [ ] **Step 4: Verify repository Actions permissions (manual check)**

In GitHub repo settings:
- Settings → Actions → General → Workflow permissions
- Ensure **Read and write permissions** is enabled.

Expected: future runs can push to `output`.

- [ ] **Step 5 (only if needed): Initialize `output` branch manually**

If the first workflow run fails to create/push `output` (e.g. branch protection or publish step errors), initialize it once:

```bash
git checkout --orphan output
git rm -rf .
echo "# output artifacts" > .gitkeep
git add .gitkeep
git commit -m "chore: init output branch"
git push origin output
git checkout main
```

---

## Chunk 2: Update README to embed snake graph

### Task 2: Insert `<picture>` snippet under GitHub Insights

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Insert snippet**

In `README.md`, under:
- Section header `## 📊 GitHub Insights`
- After the existing two `<img ...>` stats cards block
- Before the next `---`

Insert:

```html
<!-- Snake contribution graph (auto light/dark) -->
<picture>
  <source media="(prefers-color-scheme: dark)"
          srcset="https://raw.githubusercontent.com/nanfangns/nanfangns/output/dist/github-contribution-grid-snake-dark.svg" />
  <img alt="github contribution grid snake animation"
       src="https://raw.githubusercontent.com/nanfangns/nanfangns/output/dist/github-contribution-grid-snake.svg" />
</picture>
```

- [ ] **Step 2: Commit README change**

Run:
```bash
git add README.md
git commit -m "docs: embed snake contribution graph in profile README"
```
Expected: commit succeeds.

- [ ] **Step 3: Push main branch**

Run:
```bash
git push origin main
```
Expected: `main -> main` pushed.

---

## Chunk 3: Verification (evidence before completion)

### Task 3: Trigger workflow and verify output + rendering

**Verification commands (local) + expected results:**

- [ ] **Step 1: Trigger workflow manually**

Run:
```bash
gh workflow run "Generate snake contribution graph" -R nanfangns/nanfangns
```
Expected: exit code 0 and a confirmation message indicating the dispatch event was created.

- [ ] **Step 2: Confirm the workflow run succeeds**

Run:
```bash
gh run list -R nanfangns/nanfangns --workflow snake.yml -L 1
```
Expected: latest run shows status `completed` and conclusion `success`.

(Optional details):
```bash
gh run view -R nanfangns/nanfangns --log
```
Expected: steps show generation + publish succeeded.

- [ ] **Step 3: Confirm `output` branch exists**

Run:
```bash
git ls-remote --heads origin output
```
Expected: one line ending with `refs/heads/output`.

- [ ] **Step 4: Confirm the SVGs are accessible (HTTP 200)**

Run:
```bash
curl -I "https://raw.githubusercontent.com/nanfangns/nanfangns/output/dist/github-contribution-grid-snake.svg"
curl -I "https://raw.githubusercontent.com/nanfangns/nanfangns/output/dist/github-contribution-grid-snake-dark.svg"
```
Expected: both return `HTTP/2 200` (or `HTTP/1.1 200 OK`).

- [ ] **Step 5: Confirm README renders snake graph**

Open:
- `https://github.com/nanfangns/nanfangns`

Expected:
- 在 “📊 GitHub Insights” 区块下能看到蛇图。
- 切换 GitHub 主题（Dark/Light）后，snake 图随主题切换。

---

## Notes / 常见故障排查
- 如果生成步骤报 `Unexpected input(s)`：说明 snk action 的 inputs 名称与计划不一致，以该 action 的 README/错误提示为准调整（常见是 `outputs` vs `output`）。
- 如果 publish 步骤报 403 / permission denied：优先检查仓库 Actions 的 workflow permissions 是否是 Read/Write。
- 如果 output 分支启用了保护（必须 PR / 禁止 push）：需要放开或给 Actions 开例外，否则自动更新会失败。
- raw 链接更新有缓存：通常延迟几分钟属正常，可先用 `curl -I` 看是否已更新。
