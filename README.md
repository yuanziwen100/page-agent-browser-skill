# Page Agent Browser Skill

The Codex skill for driving Alibaba PageAgent through a real Chrome profile.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](./LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/yuanziwen100/page-agent-browser-skill?style=flat-square)](https://github.com/yuanziwen100/page-agent-browser-skill/stargazers)
[![Upstream TypeScript](https://img.shields.io/badge/Upstream-TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://github.com/alibaba/page-agent)
[![Upstream Downloads](https://img.shields.io/npm/dm/page-agent?style=flat-square)](https://www.npmjs.com/package/page-agent)
[![Upstream Bundle Size](https://img.shields.io/bundlephobia/minzip/page-agent?style=flat-square)](https://bundlephobia.com/package/page-agent)

English | [中文](./README-ZN.md)

## Bilingual Summary

`page-agent-browser` is a Codex skill that combines real Chrome attachment, Page Agent Ext, and direct page injection for high-precision browser work.

`page-agent-browser` is also presented in Chinese here: it combines real Chrome attachment, Page Agent Ext, and direct page injection into one verified browser workflow for Codex.

It sits one layer above `chrome-devtools-real` and adds a practical PageAgent workflow with explicit verification.

## Overview

`page-agent-browser` combines:

- `chrome-devtools-real` for real Chrome attachment
- `Page Agent Ext` for extension-backed browser control
- direct in-page `PageAgent` injection as a fallback
- Codex-side verification for DOM state and final tab state

This skill is designed for browser workflows where low-level automation is too brittle or too verbose.

## Why This Skill Exists

Raw browser scripting works for simple actions, but it becomes awkward when:

- the site has long multi-step flows
- the user is already logged in inside their normal browser
- multiple tabs must be coordinated
- extension state matters
- strict CSP blocks third-party script injection

This skill gives Codex a practical operating model:

1. Codex controls the browser through MCP tools.
2. PageAgent handles higher-level natural-language interaction.
3. Codex verifies the result instead of trusting agent output blindly.

## Key Features

- Prefer `chrome-devtools-real` over isolated browser sessions
- Reuse the user's real Chrome profile and login state
- Prefer `Page Agent Ext` on GitHub and other CSP-heavy sites
- Fall back to direct in-page `PageAgent` injection for current-tab work
- Verify the final focused tab after multi-page execution
- Repair incorrect end-tab state when the agent reports success but leaves the wrong tab selected

## Repository Layout

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

## Requirements

- Codex CLI with skill support
- Browser MCP configured
- Preferably `chrome-devtools-real`
- Chrome with `Page Agent Ext` installed in the profile Codex should use
- An OpenAI-compatible model endpoint for PageAgent

## Installation

### Option 1. Copy the packaged skill directory

1. Download or clone this repository.
2. Copy the `skill/` folder into your Codex skills directory.
3. Rename the copied folder to `page-agent-browser` if needed.

Typical target:

```text
~/.codex/skills/page-agent-browser
```

On this machine:

```text
C:\Users\15790\.codex\skills\page-agent-browser
```

### Option 2. Install from the local publish folder

If you already have this repository on disk, copy:

```text
page-agent-browser-skill/skill
```

to:

```text
~/.codex/skills/page-agent-browser
```

### After installation

1. Restart Codex CLI so it reloads the skill list.
2. Start Codex with `chrome-devtools-real` available.
3. Open your real Chrome profile with `Page Agent Ext` installed.
4. Enable remote debugging for that Chrome session.
5. Invoke the skill explicitly in your prompt.

## Related Skill

The transport layer used by this repository is published separately as [chrome-devtools-real-skill](https://github.com/yuanziwen100/chrome-devtools-real-skill).

## Recommended Runtime Setup

1. Start Codex with `chrome-devtools-real` enabled.
2. Open the user's normal Chrome profile, not an isolated automation profile.
3. Enable remote debugging for that Chrome session.
4. Keep the intended Chrome profile active when precision matters.
5. Invoke the skill explicitly when the task benefits from PageAgent.

Example:

```text
Use $page-agent-browser to drive PageAgent through chrome-devtools-real in my real Chrome profile.
```

## GitHub Notes

GitHub blocks external CDN script injection with CSP. On GitHub, this skill is extension-first by design.

That means:

- direct `page-agent.demo.js` injection is not the preferred path
- `Page Agent Ext` is preferred
- multi-tab tasks are supported, but Codex still verifies final selected tab state

## License

This repository is released under the [MIT License](./LICENSE).
