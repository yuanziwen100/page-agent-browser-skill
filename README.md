# Page Agent Browser Skill

一个面向 Codex 的浏览器技能，用来通过真实 Chrome 驱动 Alibaba PageAgent。

这个仓库打包了一套可复用的 Codex skill，核心目标是让 Codex 在真实浏览器里进行更高精度的自然语言网页操作，而不是只依赖低层级的 `click`、`fill`、`press_key` 脚本。

## 这个 Skill 能做什么

它主要解决下面几类问题：

- 页面流程很长，手写选择器和点击步骤太繁琐
- 用户已经在真实 Chrome 里登录，必须复用现有登录态
- 任务涉及多个标签页，需要跨页协同
- 站点有严格 CSP，外部脚本注入容易失败
- 需要让 Codex 调用 `Page Agent Ext`，而不是只控制隔离浏览器

这套 skill 的工作方式是：

1. Codex 通过浏览器 MCP 连接浏览器
2. PageAgent 负责高层自然语言网页交互
3. Codex 负责最终校验页面状态和标签页状态

## 主要特性

- 优先使用 `chrome-devtools-real` 连接用户真实 Chrome
- 优先复用真实 Chrome profile 里的 `Page Agent Ext`
- 当前页可回退到直接注入 `PageAgent`
- GitHub 和其他强 CSP 站点默认优先走扩展模式
- 多标签任务结束后，Codex 会额外校验最终焦点页，而不是盲信 agent 返回结果

## 仓库结构

```text
skill/
  SKILL.md
  agents/
    openai.yaml
  references/
    page-agent-real-chrome.md
```

## 运行前提

- 已安装并可使用 Codex CLI
- 已配置浏览器 MCP
- 最好已经配置 `chrome-devtools-real`
- 真实 Chrome 中已经安装 `Page Agent Ext`
- PageAgent 可访问一个 OpenAI-compatible 模型接口

## 推荐使用方式

1. 启动带有 `chrome-devtools-real` 的 Codex
2. 打开你平时使用的真实 Chrome，而不是隔离自动化浏览器
3. 确保该 Chrome 会话已开启 remote debugging
4. 确保 Page Agent Ext 安装在当前实际使用的 profile 中
5. 在 Codex 中显式调用这个 skill

示例提示词：

```text
Use $page-agent-browser to drive PageAgent through chrome-devtools-real in my real Chrome profile.
```

## 关于 GitHub 场景

GitHub 对外部脚本注入限制较强，因此这个 skill 在 GitHub 上默认优先使用扩展模式，而不是优先尝试 CDN 注入。

这意味着：

- `page-agent.demo.js` 不是 GitHub 上的默认方案
- `Page Agent Ext` 是优先路径
- 多标签操作可以做，但结束后仍要由 Codex 复核最终标签页状态

## 本地安装方式

把 `skill/` 目录复制到你的 Codex skills 目录中，并保持目录名为 `page-agent-browser`。

典型目标路径：

```text
~/.codex/skills/page-agent-browser
```

在当前这台机器上，对应路径是：

```text
C:\Users\15790\.codex\skills\page-agent-browser
```

## 发布到 GitHub

这个仓库已经按 GitHub 发布结构整理好。创建远端仓库后，可以直接把当前目录推上去。

典型命令：

```powershell
cd C:\Users\15790\skills\page-agent-browser-skill
git init
git add .
git commit -m "Add page-agent-browser Codex skill"
git branch -M main
git remote add origin https://github.com/<your-username>/page-agent-browser-skill.git
git push -u origin main
```

如果你的本机没有 `gh`，也没有现成的 GitHub 凭据，可以直接通过浏览器网页方式创建仓库并上传这些文件。
