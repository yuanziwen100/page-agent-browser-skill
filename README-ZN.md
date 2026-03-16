# Page Agent Browser Skill

通过真实 Chrome profile 驱动 Alibaba PageAgent 的 Codex 技能仓库。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](./LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/yuanziwen100/page-agent-browser-skill?style=flat-square)](https://github.com/yuanziwen100/page-agent-browser-skill/stargazers)
[![Upstream TypeScript](https://img.shields.io/badge/Upstream-TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://github.com/alibaba/page-agent)
[![Upstream Downloads](https://img.shields.io/npm/dm/page-agent?style=flat-square)](https://www.npmjs.com/package/page-agent)
[![Upstream Bundle Size](https://img.shields.io/bundlephobia/minzip/page-agent?style=flat-square)](https://bundlephobia.com/package/page-agent)

[English](./README.md) | 中文

## 双语简介

`page-agent-browser` is a Codex skill that combines real Chrome attachment, Page Agent Ext, and direct page injection for high-precision browser work.

`page-agent-browser` 是一个把真实 Chrome 连接、Page Agent Ext 扩展模式和页面内直接注入组合起来的 Codex 技能，适合高精度浏览器任务。

It sits one layer above `chrome-devtools-real` and adds a practical PageAgent workflow with explicit verification.

它建立在 `chrome-devtools-real` 之上，并为 PageAgent 增加了可验证的实际工作流。

## 概览

`page-agent-browser` 组合了以下几层能力：

- `chrome-devtools-real`，用于连接真实 Chrome
- `Page Agent Ext`，用于扩展模式下的浏览器控制
- 页面内直接注入 `PageAgent`，作为单页任务的回退方案
- Codex 侧校验，用于确认 DOM 状态和最终焦点标签页

这个 skill 适合那些低层自动化能做，但写起来脆弱、冗长且难维护的浏览器流程。

## 为什么需要它

纯 `click`、`fill`、`press_key` 在简单页面上很好用，但在这些场景会迅速变笨重：

- 页面流程长，步骤多
- 用户已经在自己的真实浏览器里登录
- 任务涉及多个标签页协调
- 扩展状态本身就是流程的一部分
- 站点 CSP 严格，外部脚本注入容易失败

这个 skill 给 Codex 提供了一套更实用的工作模型：

1. Codex 通过 MCP 工具控制浏览器。
2. PageAgent 负责更高层的自然语言网页操作。
3. Codex 负责最终校验，而不是盲信 agent 文本输出。

## 核心特性

- 优先使用 `chrome-devtools-real`，而不是隔离浏览器
- 复用用户真实 Chrome profile、登录态和扩展状态
- 在 GitHub 等强 CSP 站点优先使用 `Page Agent Ext`
- 当前页任务可回退到直接注入 `PageAgent`
- 多标签任务结束后额外核验最终焦点页
- 当 agent 声称成功但停在错误标签页时，由 Codex 纠正最终状态

## 仓库结构

```text
.
|-- README.md
|-- README-ZN.md
|-- .gitignore
|-- LICENSE
`-- skill/
    |-- SKILL.md
    |-- agents/
    |   `-- openai.yaml
    `-- references/
        `-- page-agent-real-chrome.md
```

## 运行前提

- 已安装支持 skill 的 Codex CLI
- 已配置浏览器 MCP
- 最好已经具备 `chrome-devtools-real`
- 目标 Chrome profile 中已经安装 `Page Agent Ext`
- PageAgent 能访问一个 OpenAI-compatible 模型接口

## 安装方式

### 方式 1. 复制打包好的 skill 目录

1. 下载或克隆当前仓库。
2. 将其中的 `skill/` 目录复制到 Codex 的 skills 目录。
3. 如有需要，将复制后的目录命名为 `page-agent-browser`。

典型目标路径：

```text
~/.codex/skills/page-agent-browser
```

当前这台机器上的对应路径：

```text
C:\Users\15790\.codex\skills\page-agent-browser
```

### 方式 2. 从本地发布目录安装

如果你已经把这个仓库放在本地，只需要把：

```text
page-agent-browser-skill/skill
```

复制到：

```text
~/.codex/skills/page-agent-browser
```

### 安装后

1. 重启 Codex CLI，让技能列表重新加载。
2. 启动带有 `chrome-devtools-real` 的 Codex。
3. 打开装有 `Page Agent Ext` 的真实 Chrome profile。
4. 为该 Chrome 会话开启 remote debugging。
5. 在提示词中显式调用这个 skill。

## 相关技能

本仓库依赖的真实浏览器传输层已单独发布为 [chrome-devtools-real-skill](https://github.com/yuanziwen100/chrome-devtools-real-skill)。

## 推荐运行方式

1. 启动带有 `chrome-devtools-real` 的 Codex。
2. 打开用户平时使用的真实 Chrome，而不是隔离自动化浏览器。
3. 为该 Chrome 会话开启 remote debugging。
4. 当任务依赖 PageAgent 或真实浏览器状态时，显式调用这个 skill。

示例提示词：

```text
Use $page-agent-browser to drive PageAgent through chrome-devtools-real in my real Chrome profile.
```

## GitHub 场景说明

GitHub 会通过 CSP 限制外部 CDN 脚本注入，所以这个 skill 在 GitHub 上默认优先使用扩展模式。

这意味着：

- `page-agent.demo.js` 不是 GitHub 上的优先路径
- `Page Agent Ext` 是首选方案
- 多标签任务可以交给扩展执行，但结束后仍由 Codex 校验最终标签页状态

## 许可证

本仓库基于 [MIT License](./LICENSE) 发布。
