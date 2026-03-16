# Page Agent Browser Skill

The Codex skill for driving Alibaba PageAgent through a real Chrome profile.

🌐 English | [中文](./README-ZN.md)

## Overview

`page-agent-browser` is a Codex skill that combines:

- `chrome-devtools-real` for real Chrome attachment
- `Page Agent Ext` for extension-backed browser control
- direct in-page `PageAgent` injection as a fallback
- Codex-side verification for DOM state and final tab state

This skill is designed for high-precision browser workflows where low-level automation is too brittle or too verbose.

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
- Verify final focused tab after multi-page execution
- Repair incorrect end-tab state when the agent reports success but leaves the wrong tab selected

## Repository Layout

```text
.
├─ README.md
├─ README-ZN.md
├─ .gitignore
└─ skill/
   ├─ SKILL.md
   ├─ agents/
   │  └─ openai.yaml
   └─ references/
      └─ page-agent-real-chrome.md
```

## Requirements

- Codex CLI with skill support
- Browser MCP configured
- Preferably `chrome-devtools-real`
- Chrome with `Page Agent Ext` installed in the profile Codex should use
- An OpenAI-compatible model endpoint for PageAgent

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

## Local Installation

Copy `skill/` into your Codex skills directory and keep the folder name as `page-agent-browser`.

Typical target:

```text
~/.codex/skills/page-agent-browser
```

On this machine:

```text
C:\Users\15790\.codex\skills\page-agent-browser
```
