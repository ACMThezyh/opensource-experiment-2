# 开源软件实验二：GitHub Actions 和 GitHub Pages 使用

> 课程：开源软件  
> 实验：实验 2 — GitHub Actions 和 GitHub Pages 使用  
> 提交件：`开源软件实验二报告+学号+姓名.docx`

[![Deploy MkDocs to GitHub Pages](https://github.com/ACMThezyh/opensource-experiment-2/actions/workflows/deploy-docs.yml/badge.svg)](https://github.com/ACMThezyh/opensource-experiment-2/actions/workflows/deploy-docs.yml)
[![GitHub Pages](https://img.shields.io/badge/docs-GitHub%20Pages-blue)](https://acmthezyh.github.io/opensource-experiment-2/)

---

## 一、实验目的

### 1.1 GitHub Actions 使用

1. 学会创建基本的 GitHub Actions 工作流（workflow）配置文件；
2. 触发工作流运行并观察其执行过程；
3. 验证工作流是否成功执行。

### 1.2 GitHub Pages 使用

掌握使用 GitHub Actions 实现静态网站（MkDocs 文档）的**自动化构建与部署**，最终通过 GitHub Pages 托管可访问的文档网站。

---

## 二、实验平台

| 项目 | 说明 |
| --- | --- |
| 操作系统 | Windows 10 / Windows 11 / macOS / Linux |
| 代码管理 | Git、GitHub |
| 编辑器 | Visual Studio Code |
| 文档框架 | MkDocs + Material for MkDocs |
| CI/CD | GitHub Actions |
| 静态托管 | GitHub Pages |

**依赖版本参考：** Python 3.9+、Git 2.30+、VS Code 1.80+

---

## 三、快速开始（本地）

```bash
# 1. 克隆仓库
git clone https://github.com/ACMThezyh/opensource-experiment-2.git
cd opensource-experiment-2

# 2. 创建虚拟环境（可选但推荐）
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

# 3. 安装依赖
pip install -r requirements.txt

# 4. 本地预览
mkdocs serve
# 浏览器访问 http://127.0.0.1:8000

# 5. 本地构建（严格模式）
mkdocs build --strict
```

---

## 四、目录结构

```
.
├── CLAUDE.md                     # AI 协作指引（开发规范）
├── README.md                     # 本文件
├── mkdocs.yml                    # MkDocs 站点配置
├── requirements.txt              # Python 依赖
├── docs/                         # 文档源码
│   ├── index.md                  # 首页
│   ├── experiment/
│   │   ├── github-actions.md     # 实验 2-1：GitHub Actions
│   │   └── github-pages.md       # 实验 2-2：GitHub Pages
│   └── assets/screenshots/       # 实验截图
├── .github/workflows/
│   ├── hello-action.yml          # 基础演示工作流
│   └── deploy-docs.yml           # MkDocs 构建部署工作流
└── report/                       # 实验报告提交件
    └── 开源软件实验二报告+学号+姓名.docx
```

---

## 五、实验内容及要求

参照学习通资料中的《实验03-Github Action和GitHub Pages使用》文档，按以下步骤完成。

### 5.1 GitHub Actions 部分

| 步骤 | 操作 | 验证方式 |
| --- | --- | --- |
| 1 | 在 `.github/workflows/` 下创建 `hello-action.yml` | 文件被 GitHub 识别为 workflow |
| 2 | `git push` 触发工作流 | 仓库 **Actions** 页签出现运行记录 |
| 3 | 观察执行过程 | 逐步骤查看日志、Job 依赖关系、耗时 |
| 4 | 验证执行结果 | 运行状态为绿色 ✅，日志输出符合预期 |

基础工作流示例（`hello-action.yml`）：

```yaml
name: Hello GitHub Actions

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
      - name: 查看仓库文件
        run: ls -la
      - name: 输出问候
        run: echo "Hello, GitHub Actions!"

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-python@v5
        with:
          python-version: '3.x'
      - name: 打印 Python 版本
        run: python --version
```

### 5.2 GitHub Pages 部分

| 步骤 | 操作 | 验证方式 |
| --- | --- | --- |
| 1 | 编写 `mkdocs.yml` 与 `docs/` 内容 | `mkdocs serve` 本地预览正常 |
| 2 | 创建 `deploy-docs.yml` 部署工作流 | 文件通过 YAML 语法校验 |
| 3 | 配置仓库 **Settings → Pages → Source = GitHub Actions** | 页面显示 "GitHub Actions" |
| 4 | 推送代码触发部署 | Actions 中 `build` 与 `deploy` 两个 Job 均成功 |
| 5 | 访问站点 | `https://acmthezyh.github.io/opensource-experiment-2/` 可正常打开 |

> ⚠️ **常见坑位**
> - 未设置 `permissions: pages: write` 与 `id-token: write` → 部署报错 `Resource not accessible by integration`
> - Pages Source 未切到 GitHub Actions → 部署后仍显示 404
> - `deploy` Job 缺少 `needs: build` → 在构建产物上传前就开始部署

---

## 六、实验报告说明

实验报告采用 **「文字总结 + 截图」** 形式，包含以下章节：

1. 实验目的
2. 实验平台
3. 实验内容及要求
4. 实验步骤与结果（含截图）
5. 问题与解决
6. 实验总结

**提交文件名（严格遵守）：**

```
开源软件实验二报告+学号+姓名.doc
开源软件实验二报告+学号+姓名.docx
```

> 两种扩展名任选其一，文件名中的 `+` 号与顺序不得改动。

---

## 七、实验结果

| 项目 | 状态 | 链接 |
| --- | --- | --- |
| GitHub Actions 工作流 | ✅ 运行成功 | [Actions](../../actions) |
| MkDocs 文档站点 | ✅ 已上线 | https://acmthezyh.github.io/opensource-experiment-2/ |

---

## 八、参考资料

- [GitHub Actions 官方文档](https://docs.github.com/zh/actions)
- [GitHub Pages 官方文档](https://docs.github.com/zh/pages)
- [MkDocs 官方文档](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)

---

## 九、许可

本项目仅用于课程学习，遵循 [MIT License](LICENSE)。
