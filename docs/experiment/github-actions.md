# 实验 2-1：GitHub Actions 实验记录

本实验完成 GitHub Actions 工作流的**创建 → 触发 → 观察 → 验证**全流程。

## 实验目的

1. 学会创建基本的 GitHub Actions 工作流（workflow）配置文件；
2. 触发工作流运行并观察其执行过程；
3. 验证工作流是否成功执行。

## 步骤一：创建工作流文件

在仓库的 `.github/workflows/` 目录下创建 `hello-action.yml`：

```yaml
name: Hello GitHub Actions

on:
  push:
    branches: [main]
  pull_request:
  workflow_dispatch:

jobs:
  build:
    name: 构建 Job
    runs-on: ubuntu-latest
    steps:
      - name: 检出代码
        uses: actions/checkout@v4

      - name: 查看仓库文件
        run: ls -la

      - name: 输出问候
        run: echo "Hello, GitHub Actions!"

  test:
    name: 测试 Job
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: 检出代码
        uses: actions/checkout@v4

      - name: 配置 Python 环境
        uses: actions/setup-python@v5
        with:
          python-version: '3.x'

      - name: 打印 Python 版本
        run: python --version

      - name: 安装 pytest
        run: pip install pytest

      - name: 编写并运行单元测试
        run: |
          cat > test_hello.py <<'EOF'
          def test_add():
              assert 1 + 2 == 3

          def test_greeting():
              assert "Hello, GitHub Actions!" == "Hello, GitHub Actions!"
          EOF
          pytest -v
```

**配置要点：**

- `on` 定义了三种触发方式：`push`（推送）、`pull_request`（拉取请求）、`workflow_dispatch`（手动触发）；
- 工作流包含 `build` 与 `test` 两个 Job，通过 `needs: build` 体现依赖关系——`test` 只有在 `build` 成功后才会运行；
- 使用了官方 Action：`actions/checkout@v4`（检出代码）与 `actions/setup-python@v5`（配置 Python 环境）；
- 每个关键步骤都设置了 `name`，便于在 Actions 页面阅读执行过程；
- 测试 Job 使用 `pytest -v` 运行单元测试，输出可观测的 `PASSED` 信息。

## 步骤二：触发工作流

将工作流文件推送到 `main` 分支：

```bash
git add .github/workflows/hello-action.yml
git commit -m "feat: 添加基础演示工作流"
git push origin main
```

推送完成后，仓库的 **Actions** 页签即出现该工作流的运行记录。

<!-- 2-1-01: Actions 工作流列表 -->

## 步骤三：观察执行过程

点开 **Actions** 页签中的运行记录，可以逐步骤查看日志：

- `build` Job 中"查看仓库文件"步骤打印了 `ls -la` 的输出；
- "输出问候"步骤打印了 `Hello, GitHub Actions!`；
- `test` Job 依赖 `build` 完成，页面上的依赖图（DAG）清晰展示了 `build → test` 的先后关系；
- `pytest -v` 输出了两个测试的 `PASSED` 结果。

<!-- 2-1-02: hello-action 运行成功 -->

### 故意制造失败，观察错误日志

为了观察工作流失败时的表现，将测试断言临时改为错误版本：

```python
def test_add():
    assert 1 + 2 == 4   # 故意写错
```

推送后工作流以红色 ✗ 结束，日志中可见 `FAILED` 与 `AssertionError` 的详细堆栈，证明 GitHub Actions 能如实反馈测试失败。

<!-- 2-1-03: 故意失败的红色 ✗ 与 FAILED 日志 -->

随后还原断言并以 `fix:` 提交推送，工作流恢复绿色 ✅。

<!-- 2-1-04: 修复后恢复绿色 -->

## 步骤四：验证执行结果

- 工作流运行状态为绿色 ✅；
- 日志输出符合预期（问候语、Python 版本、测试 `PASSED`）；
- 也可以在 **Actions** 页面通过 `workflow_dispatch` 手动触发一次，验证手动触发通道。

## 问题与解决

| 问题 | 排查与解决 |
| --- | --- |
| 本机向 GitHub 推送时 SSH 报 `Permission denied (publickey)` | 本机私钥未绑定 GitHub 账号。改用 `gh auth login` 完成 CLI 登录，由 gh 自动配置 SSH 密钥与凭证，之后 `git push` 走 SSH 通道成功 |
| 需要观察测试失败的真实日志 | 通过"故意写错断言 → 推送观察红 ✗ → 还原代码 fix: 修复"的方式完成，无需长期保留坏代码 |

## 小结

通过本实验掌握了 workflow / job / step 三层结构、`needs` 依赖、`on` 触发器以及官方 Action 的使用；并通过一次"故意失败"完整观察了 Actions 对错误的如实反馈。
