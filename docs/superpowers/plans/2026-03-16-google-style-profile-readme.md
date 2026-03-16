# Google-Style Profile README Redesign Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 让 `nanfangns/nanfangns` 的 Profile README 呈现更“Google 产品风”（蓝为主、四色点缀），整体更统一、更高级；保留少量动效（typing + snake），其余模块做版式与配色统一。

**Architecture:** 只改 `README.md` 的信息架构与展示组件（图片卡片、徽章、表格布局）。不改 workflow 逻辑，不引入额外外部服务。通过统一模块顺序、统一标题与分隔、减少冗余徽章与卡片来实现“好看”。

**Tech Stack:** GitHub Profile README（Markdown + 少量 HTML）、现有 GitHub Actions（snake 已存在）。

---

## File Map

**Modify:**
- `README.md`

**Do NOT modify (unless explicitly requested later):**
- `.github/workflows/snake.yml`
- `docs/**`

---

## Chunk 1: Establish Visual System + Layout Skeleton

### Task 1: Capture current README and identify sections

**Files:**
- Inspect: `README.md`

- [ ] **Step 1: Snapshot current structure**
  - Read `README.md` and list current top-level sections (H1/H2), and what embedded images/cards exist.

- [ ] **Step 2: Define target section order (no content changes yet)**
  - Target order (Google-style, low motion):
    1. Hero (banner / title / short tagline)
    2. `## 📊 GitHub Insights` (stats + streak + snake)
    3. `## 🧰 Tech Stack` (精选徽章)
    4. `## 🚀 Featured Projects`
    5. `## 🤝 Connect`

- [ ] **Step 3: Commit only if you changed nothing**
  - No commit in this task unless a change is required.

### Task 2: Implement Google-style layout skeleton in README

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Write a “failing check” (manual) definition**
  - Because this is Markdown-only, define a manual check list that currently fails:
    - [ ] README sections match the target order
    - [ ] Insights uses a 2-column table for stats + streak
    - [ ] Snake block remains under Insights and links to `/output/...` (no `dist/`)
    - [ ] Tech Stack badges reduced to a curated set (≤ 12)

- [ ] **Step 2: Apply section order and consistent separators**
  - Add/adjust `---` separators so each major section is clearly separated.
  - Ensure headings are consistent: `## <emoji> <Title>`.

- [ ] **Step 3: Build Insights as a clean 2-column table**
  - Keep existing stats + streak cards, but align in a Markdown table:

```md
| Stats | Streak |
| --- | --- |
| <img ...> | <img ...> |
```

- [ ] **Step 4: Keep snake as the only animation inside Insights (besides typing in hero)**
  - Ensure snake snippet remains:

```html
<picture>
  <source media="(prefers-color-scheme: dark)"
          srcset="https://raw.githubusercontent.com/nanfangns/nanfangns/output/github-contribution-grid-snake-dark.svg" />
  <img alt="github contribution grid snake animation"
       src="https://raw.githubusercontent.com/nanfangns/nanfangns/output/github-contribution-grid-snake.svg" />
</picture>
```

- [ ] **Step 5: Commit**

```bash
git add README.md
git commit -m "docs: restructure README into google-style sections"
```

---

## Chunk 2: Google Primary Color Badges + Curated Tech Stack

### Task 3: Curate Tech Stack badges (Google Blue primary)

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Choose curated badge list (≤12)**
  - Prefer stable, core stack. Example categories:
    - Languages: TypeScript, Python, Go
    - Backend: Node.js, FastAPI
    - Infra: Docker, Cloudflare
    - Data: PostgreSQL, Redis
    - Frontend: React, Next.js
  - (Actual list should match the user’s real stack; remove random/unrelated badges.)

- [ ] **Step 2: Normalize badge style**
  - Use consistent shields parameters:
    - `style=flat` or `style=flat-square`
    - consistent logo color and label casing
  - Primary color: Google Blue `4285F4` as the badge color where applicable.

- [ ] **Step 3: Manual verification**
  - Open README preview (GitHub page) and check:
    - badge heights consistent
    - not overly colorful

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs: curate tech stack badges with google primary color"
```

---

## Chunk 3: Featured Projects + Connect Cleanup

### Task 4: Featured Projects - make it look like a product list

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Define project card format**
  - Use a simple table for 2 projects per row (or 1 per row if long descriptions).
  - Each project entry:
    - Project name (link)
    - One-line value proposition
    - 2-4 compact tags (optional)

- [ ] **Step 2: Remove clutter**
  - Remove redundant widgets that duplicate Insights.
  - Keep only what supports “portfolio”.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: refine featured projects layout"
```

### Task 5: Connect - unify CTA style

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Normalize social links**
  - Keep 3-5 maximum links.
  - Ensure consistent icon/badge style.

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "docs: tidy connect section"
```

---

## Final Verification

- [ ] **Step 1: Validate snake raw URLs return 200**

```bash
curl -I "https://raw.githubusercontent.com/nanfangns/nanfangns/output/github-contribution-grid-snake.svg"
curl -I "https://raw.githubusercontent.com/nanfangns/nanfangns/output/github-contribution-grid-snake-dark.svg"
```

Expected: `200`.

- [ ] **Step 2: Preview README on GitHub (manual)**
  - Check mobile rendering (narrow width): tables stack acceptably.
  - Check dark mode: snake switches to dark SVG.

- [ ] **Step 3: Push and merge**
  - If working on feature branch:

```bash
git push origin <branch>
```

  - Create PR and merge to `main`.

---

## Notes / Constraints
- GitHub Profile README 的“玻璃拟态”无法用 CSS 真正实现，所以用“统一布局 + 克制色彩 + 分组”实现类似质感。
- 保持动效克制：只留 typing + snake，避免显得杂。
