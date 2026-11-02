# CLAUDE.md

本文件为 Claude Code（claude.ai/code）在本仓库中工作时提供指引。
所有代码、文档、提交信息均应遵守本文件的约定。

---

## 1. 项目背景

本仓库是《开源软件》课程**实验二**的配套工程与实验报告载体，主题为：

> **GitHub Actions 与 GitHub Pages 使用**

实验需完成两件事：

1. **GitHub Actions**：创建基本的 Actions 工作流配置文件 → 触发工作流 → 观察执行过程 → 验证执行结果。
2. **GitHub Pages**：使用 GitHub Actions 实现 MkDocs 静态文档站点的**自动化构建与部署**，最终通过 GitHub Pages 托管可访问的文档网站。

---

## 2. 目录结构约定

```
.
├── CLAUDE.md                     # 本文件：AI 协作指引
├── README.md                     # 仓库说明与实验操作手册
├── mkdocs.yml                    # MkDocs 站点配置文件
├── requirements.txt              # Python 依赖（mkdocs、mkdocs-material）
├── docs/                         # 文档源码目录
│   ├── index.md                  # 首页
│   ├── experiment/
│   │   ├── github-actions.md     # 实验2-1：GitHub Actions 实验记录
│   │   └── github-pages.md       # 实验2-2：GitHub Pages 实验记录
│   ├── assets/                   # 图片资源（截图）
│   │   └── screenshots/          # 报告用截图统一存放于此
│   └── stylesheets/
│       └── extra.css             # 自定义样式（可选）
├── .github/
│   └── workflows/
│       ├── hello-action.yml      # 演示用基础工作流
│       └── deploy-docs.yml       # MkDocs 构建并部署到 GitHub Pages
└── report/                       # 实验报告最终提交件
    └── 开源软件实验二报告+学号+姓名.docx
```

**新增文件时必须落到上面对应目录**，不要随意在仓库根目录堆砌文件。

---

## 3. 常用命令

```bash
# 安装依赖
pip install -r requirements.txt

# 本地预览（http://127.0.0.1:8000）
mkdocs serve

# 本地构建（输出到 site/）
mkdocs build

# 严格模式构建（警告即错误，用于 CI 前自检）
mkdocs build --strict
```

> `site/` 为构建产物，**不要提交到仓库**，应由 `.gitignore` 排除。

---

## 4. 工作流规范

### 4.1 基础演示工作流（`.github/workflows/hello-action.yml`）

要求能体现实验目的中「创建 → 触发 → 观察 → 验证」四步：

- 触发条件：`push`、`pull_request`，以及手动触发 `workflow_dispatch`
- 至少包含两个 Job（如 `build` 与 `test`），体现 Job 依赖关系 `needs`
- 使用 `actions/checkout@v4`、`actions/setup-python@v5` 等官方 Action
- 步骤中输出可观测信息（如 `echo`、运行测试、打印 Python 版本）
- 为关键步骤设置 `name`，便于在 Actions 页面阅读执行过程

### 4.2 部署工作流（`.github/workflows/deploy-docs.yml`）

部署 MkDocs 到 GitHub Pages 的推荐写法：

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

**要点提示（修改此文件时务必保持）：**

- 必须声明 `permissions: pages: write` 与 `id-token: write`，否则部署失败
- 仓库 **Settings → Pages → Source** 需设置为 **GitHub Actions**
- 使用 `concurrency` 防止并发部署互相覆盖
- `deploy` Job 必须 `needs: build`

---

## 5. 文档与代码规范

- 所有 Markdown 文档使用**简体中文**书写，技术名词保留英文原文（如 workflow、job、step）。
- 标题层级从 `#` 开始，不跳级；文件内只允许一个一级标题。
- 命令、路径、配置项统一使用反引号包裹；多行命令使用带语言标识的代码块。
- 截图统一放在 `docs/assets/screenshots/`，命名格式：`{章节号}-{序号}-{内容}.png`，例如 `2-1-01-actions-workflow-list.png`。
- 图片引用必须使用相对路径：`![描述](../assets/screenshots/xxx.png)`。
- 提交信息遵循 Conventional Commits：`feat:` / `fix:` / `docs:` / `chore:`。

---

## 6. 实验报告写作规范

实验报告采用 **「文字总结 + 截图」** 的形式，结构固定为：

1. **实验目的** —— 对应 GitHub Actions 与 GitHub Pages 两部分目标
2. **实验平台** —— 操作系统（Windows 10/11、macOS、Linux）、Git、VS Code 版本
3. **实验内容及要求** —— 说明参照《实验03-Github Action和GitHub Pages使用》文档完成
4. **实验步骤与结果** —— 每一步骤写「做了什么 + 关键命令/配置 + 结果截图 + 结果说明」
5. **问题与解决** —— 记录实际遇到的问题及排查过程（如 Pages 403、workflow 权限不足）
6. **实验总结** —— 收获与体会，需结合具体操作，避免空话

**撰写要求：**

- 截图必须真实来自本人操作过程，**严禁使用占位图或伪造截图**。
- 每张截图下方须有图注说明该图证明了什么。
- 代码片段与仓库中实际文件保持一致，修改工作流后需同步更新报告。

**最终提交件命名（严格遵守）：**

```
开源软件实验二报告+学号+姓名.doc
开源软件实验二报告+学号+姓名.docx
```

> 两种格式任选其一（`.doc` 或 `.docx`），文件名中的 `+` 号保留。

---

## 7. 禁止事项

- ❌ 不要在仓库中提交 `site/` 构建产物、`.venv/`、`__pycache__/`
- ❌ 不要在公开仓库中提交任何个人敏感信息（学号以外的身份证号、手机号、Token 等）
- ❌ 不要硬编码 GitHub Token；如需认证请使用 `secrets.GITHUB_TOKEN`
- ❌ 不要修改 `.github/workflows/` 中的 `permissions` 为 `write-all`
- ❌ 不要在未运行 `mkdocs build --strict` 通过的情况下推送文档变更

---

## 8. 验证清单（提交前自检）

- [ ] `mkdocs serve` 本地预览正常，无死链
- [ ] `mkdocs build --strict` 无警告
- [ ] GitHub Actions 页面显示最新 workflow 运行状态为 ✅
- [ ] GitHub Pages 站点可通过 `https://<用户名>.github.io/<仓库名>/` 访问
- [ ] 报告文件名符合 `开源软件实验二报告+学号+姓名.docx` 格式
- [ ] 报告中所有截图均有图注，且与正文描述一致
