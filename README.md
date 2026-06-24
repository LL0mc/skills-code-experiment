# code-experiment / 代码实验

A skill for running safe, structured code experiments on an isolated Git branch. Works with any AI coding agent that supports skill installation (opencode, Claude Code, etc.).

一个在独立 Git 分支上进行安全、结构化代码实验的 skill。适用于支持 skill 安装的任何 AI 编码助手（opencode、Claude Code 等）。

> This skill was inspired by **Auto Research** and summarized from hands-on experience doing code experiments and optimization iterations.
>
> 本 skill 受 **Auto Research** 启发，从我自己的代码实验和优化迭代实践中总结而成。

## What it does / 功能

When you want to try a risky optimization or prototype different implementations, this skill guides you through:

当你想要尝试有风险的优化或对不同的实现方案进行原型验证时，本 skill 引导你完成：

1. **Setup** — create an experiment branch, define success criteria, measure baseline
   **设置阶段** — 创建实验分支、定义成功标准、测量基线
2. **Iterate** — make one change at a time, evaluate, rollback on failure, track everything
   **迭代阶段** — 每次只改一处、评估效果、失败则回滚、全程记录
3. **Wrap up** — present results, let you decide whether to merge or discard
   **收尾阶段** — 呈现结果、由你决定合并还是丢弃

No copies, no feature flags, no scaffolding. The Git branch IS the sandbox.
无需副本、无需功能开关、无需额外脚手架。Git 分支本身就是沙箱。

**Not for:** pure refactoring (which needs regression test coverage, not iterative experiments), documentation, deployment, CI setup, writing brand-new modules, or simple bug fixes.
**不适合：** 纯重构（需要回归测试覆盖而非迭代实验）、文档、部署、CI 配置、从头写新模块、简单修 Bug。

## Install / 安装

Clone or download this repository, then copy the `code-experiment` folder to your agent's skills directory:

克隆或下载本仓库，然后将 `code-experiment` 文件夹复制到你所用 agent 的 skill 目录：

```bash
# For opencode / 适用于 opencode
cp -r code-experiment ~/.config/opencode/skills/

# For Claude Code / 适用于 Claude Code
cp -r code-experiment ~/.claude/skills/

# For a single project / 单个项目
cp -r code-experiment .opencode/skills/
```

Restart your agent. The skill will be available automatically.
重启你的 agent，skill 会自动生效。

## Requirements / 依赖

- Git
- An agent that supports skill installation (opencode, Claude Code, etc.)

## Usage / 使用方式

Just tell the agent you want to try something experimental:

直接告诉 agent 你想做实验性的改动：

Chinese:
- "帮我试试能不能把这个冒泡排序换成快速排序，看看能快多少"
- "我想做个实验，把这三个函数合并成一个，测一下代码量变化"
- "帮我 prototype 一下用缓存优化这个查询的方案"

English:
- "Try replacing this bubble sort with quicksort and measure the difference"
- "I want to experiment with merging these three functions into one, compare the LOC"
- "Prototype a caching optimization for this query and benchmark it"

The agent will propose the experiment framework and walk through the workflow.
agent 会提出实验框架并引导你完成整个流程。

## Contributing / 欢迎贡献

Have ideas to improve the workflow? Found a edge case not covered? PRs and issues are welcome.

如果你有改进工作流的想法，或者发现了没有覆盖到的场景，欢迎提 PR 或 Issue。

## License / 许可证

MIT
