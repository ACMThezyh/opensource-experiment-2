# 实验 2-2：GitHub Pages 实验记录

本实验使用 GitHub Actions 实现 MkDocs 文档站点的**自动化构建与部署**，最终通过 GitHub Pages 托管可访问的文档网站。

## 实验目的

掌握使用 GitHub Actions 实现静态网站（MkDocs 文档）的自动化构建与部署，通过 GitHub Pages 托管可访问的文档网站。

## 步骤一：编写 MkDocs 站点

创建 `mkdocs.yml` 与 `docs/` 文档源码：

```yaml
site_name: 开源软件实验二：GitHub Actions 和 GitHub Pages
site_url: https://acmthezyh.github.io/opensource-experiment-2/
repo_url: https://github.com/ACMThezyh/opensource-experiment-2

theme:
  name: material
  language: zh

nav:
  - 首页: index.md
  - 实验 2-1：GitHub Actions: experiment/github-actions.md
  - 实验 2-2：GitHub Pages: experiment/github-pages.md
```

依赖通过 `requirements.txt` 声明（`mkdocs`、`mkdocs-material`）。本地预览与构建：

```bash
pip install -r requirements.txt
mkdocs serve     # 本地预览 http://127.0.0.1:8000
mkdocs build --strict   # 严格模式构建，警告即错误
```

<!-- 2-2-04: 本地 mkdocs serve 预览 -->

## 步骤二：创建部署工作流

创建 `.github/workflows/deploy-docs.yml`：

```yaml
name: Deploy MkDocs to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.x'
      - run: pip install -r requirements.txt
      - run: mkdocs build --strict
      - uses: actions/upload-pages-artifact@v3
        with:
          path: site

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

**关键要点：**

- `permissions` 中必须声明 `pages: write` 与 `id-token: write`，否则部署报错 `Resource not accessible by integration`；
- `concurrency: group: pages` 防止并发部署互相覆盖；
- `build` Job 用 `mkdocs build --strict` 构建并通过 `upload-pages-artifact` 上传产物；
- `deploy` Job 必须 `needs: build`，在产物上传完成后才调用 `deploy-pages` 发布。

## 步骤三：配置 Pages 来源

在仓库 **Settings → Pages** 中将 **Source** 设置为 **GitHub Actions**（也可用 `gh api` 完成：`gh api --method POST repos/{owner}/{repo}/pages -f build_type=workflow`）。若 Source 保持默认的"Deploy from a branch"，部署后站点仍会显示 404。

<!-- 2-2-01: Pages 设置 Source 为 GitHub Actions -->

## 步骤四：推送触发自动部署

推送 `main` 分支后，Actions 中 `Deploy MkDocs to GitHub Pages` 工作流自动运行，`build` 与 `deploy` 两个 Job 依次执行成功：

<!-- 2-2-02: deploy 工作流 build+deploy 两 Job 成功 -->

## 步骤五：访问验证站点

打开 `https://acmthezyh.github.io/opensource-experiment-2/`，文档站点已上线，可正常浏览首页与两篇实验记录：

<!-- 2-2-03: 浏览器访问线上文档站点 -->

## 问题与解决

| 问题 | 排查与解决 |
| --- | --- |
| Pages 站点部署后 404 | 原因是仓库未开启 Pages 或 Source 不是 GitHub Actions。通过 `gh api` 以 `build_type=workflow` 创建 Pages 站点，等价于网页中把 Source 设为 GitHub Actions，问题解决 |
| 部署工作流报 `Resource not accessible by integration` | 由未声明 `pages: write` / `id-token: write` 引起。本仓库的 `deploy-docs.yml` 从一开始就按规范声明了最小 `permissions`，未触发此问题 |

## 小结

本实验跑通了"推送代码 → Actions 构建 MkDocs → 发布 Pages"的完整 CI/CD 闭环：此后每次修改文档，只需 `git push`，站点即在数十秒后自动更新，体现了持续集成对文档发布效率的提升。
