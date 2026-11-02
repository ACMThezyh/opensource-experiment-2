# 开源软件实验二：GitHub Actions 和 GitHub Pages 使用

本站点是《开源软件》课程**实验二**的配套文档，由 [MkDocs](https://www.mkdocs.org/) + [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) 构建，并通过 **GitHub Actions** 自动部署到 **GitHub Pages**。

## 实验内容

1. **[实验 2-1：GitHub Actions](experiment/github-actions.md)** —— 创建基础工作流，触发、观察并验证其执行过程；
2. **[实验 2-2：GitHub Pages](experiment/github-pages.md)** —— 使用 GitHub Actions 实现 MkDocs 文档站点的自动化构建与部署。

## 本站点如何部署

每次向 `main` 分支推送代码时，[`deploy-docs.yml`](https://github.com/ACMThezyh/opensource-experiment-2/blob/main/.github/workflows/deploy-docs.yml) 工作流会自动执行：

```bash
pip install -r requirements.txt
mkdocs build --strict
```

构建产物 `site/` 随后由 `actions/deploy-pages@v4` 发布到 GitHub Pages，无需任何手动操作。

## 相关链接

- 仓库地址：<https://github.com/ACMThezyh/opensource-experiment-2>
- Actions 运行记录：<https://github.com/ACMThezyh/opensource-experiment-2/actions>
- [GitHub Actions 官方文档](https://docs.github.com/zh/actions)
- [GitHub Pages 官方文档](https://docs.github.com/zh/pages)
