---
name: page-agent-browser
description: Drive Alibaba PageAgent through Codex using `chrome-devtools-real` attached to a real Chrome profile. Use when the user asks to integrate or operate PageAgent, wants Codex to work through Page Agent Ext in their normal browser profile, needs more precise natural-language browser control than raw click/fill scripting, or wants PageAgent-backed actions in the current tab with a fallback to direct script injection.
---

# Page Agent Browser

Use this skill to let Codex control a rendered webpage through Alibaba PageAgent while staying attached to the user's real Chrome whenever possible.

Treat PageAgent as a browser-side helper layered on top of the browser MCP tools. It is not a standalone Codex plugin.

Read [references/page-agent-real-chrome.md](./references/page-agent-real-chrome.md) when you need the exact setup assumptions, official capability boundaries, or prompt-writing patterns.

## Overview

This skill exists to make Codex better at browser work that is too brittle or too verbose for raw `click` and `fill` scripting.

Use it to:

- drive the user's real Chrome through `chrome-devtools-real`
- reuse Page Agent Ext when it is already installed in the active Chrome profile
- fall back to direct in-page `PageAgent` injection for active-tab tasks
- keep Codex responsible for verification instead of trusting agent output blindly

This skill is optimized for real-user browsing contexts, especially sites where:

- the user is already logged in
- multiple tabs matter
- extension state matters
- selectors are cumbersome
- the site is hostile to third-party script injection

## Preconditions

- Prefer `chrome-devtools-real`. Use `chrome-devtools` only if the user explicitly wants the isolated browser or the real-browser attachment is unavailable.
- Confirm a browser tab is already open on the target page.
- Confirm the page is one where in-page JavaScript execution is acceptable.
- Do not assume PageAgent works without an LLM configuration.
- Ask before using PageAgent on login, payment, destructive admin, or privacy-sensitive workflows.

## Default Strategy

1. Attach to the user's real Chrome through `chrome-devtools-real`.
2. Keep the active browser profile stable. If the user says Page Agent Ext is installed in a specific profile, avoid switching to another profile window.
3. Inspect the current page before invoking PageAgent.
4. Prefer extension-backed usage when the attached browser already has Page Agent Ext and the task may span multiple steps or tabs.
5. Fall back to direct page injection when extension-backed behavior is unavailable, unverifiable, or unnecessary.
6. Verify the resulting DOM with browser snapshots instead of trusting the agent output alone.

## Tooling Assumptions

- Browser transport is provided by Codex browser MCP tools.
- `chrome-devtools-real` is the preferred target when available.
- `Page Agent Ext` may expose `window.PAGE_AGENT_EXT` on supported pages after its handshake is active.
- Model access is still required because PageAgent is not self-contained.

## Workflow

### 1. Inspect the page first

- Open or select the target page with browser tools.
- Take a snapshot before injection so you understand the current state and can verify the result afterward.

### 2. Choose the model configuration

PageAgent needs an OpenAI-compatible `baseURL`, `apiKey`, and `model`.

Use a concrete configuration that the user already trusts:

- Personal or local model:
  - `baseURL: 'http://localhost:11434/v1'`
  - `apiKey: 'NA'`
  - `model: 'qwen3:14b'`
- OpenAI-compatible cloud endpoint:
  - `baseURL: '<provider-compatible-endpoint>'`
  - `apiKey: '<secret>'`
  - `model: '<tool-capable-model>'`
- Alibaba testing API, only for technical evaluation:
  - `baseURL: 'https://page-ag-testing-ohftxirgbn.cn-shanghai.fcapp.run'`
  - `apiKey: 'NA'`
  - `model: 'qwen3.5-plus'`

Do not present the testing API as production-safe.

### 3. Decide execution mode

Use extension-backed mode first when all of the following are true:

- `chrome-devtools-real` is attached to the user's real Chrome
- the user says Page Agent Ext is installed in that profile
- the task benefits from PageAgent rather than direct DOM actions

Force extension-backed mode when any of the following are true:

- the page has a strict CSP that blocks external script injection
- the site is GitHub or another major app that restricts third-party scripts
- the task needs multiple tabs, cross-page coordination, or tab switching
- the task requires preserving the user's real login state and extension state across pages

Use direct injection mode when:

- the task is limited to the current page
- extension availability is uncertain
- the page must be handled immediately and a normal in-page agent is sufficient

Do not invent a private extension API. If extension internals are not visible from the page, continue with normal browser tools plus direct PageAgent injection for the active tab.

### 4.5. Use extension mode on GitHub-like sites

On GitHub and similar CSP-heavy sites, assume CDN injection will fail unless proven otherwise.

Use this sequence:

1. Check whether `window.PAGE_AGENT_EXT` is already exposed on the page.
2. If not, confirm the extension handshake token is present for that origin before assuming extension mode is unavailable.
3. If the extension becomes available, call `window.PAGE_AGENT_EXT.execute(...)`.
4. If the extension still does not expose an API, fall back to ordinary browser MCP actions instead of retrying blocked CDN injection repeatedly.

When using extension mode, give more specific prompts than you would for manual browser control. Include:

- exact URLs when known
- whether to open a new tab or reuse the current tab
- which tab should be focused when done
- the expected end state

### 4. Inject the script for active-tab mode

```javascript
async () => {
  if (!window.PageAgent) {
    await new Promise((resolve, reject) => {
      const script = document.createElement('script');
      script.src = 'https://cdn.jsdelivr.net/npm/page-agent@1.5.7/dist/iife/page-agent.demo.js';
      script.crossOrigin = 'anonymous';
      script.onload = resolve;
      script.onerror = () => reject(new Error('Failed to load PageAgent'));
      document.head.appendChild(script);
    });
  }
  return { hasPageAgent: typeof window.PageAgent !== 'undefined' };
}
```

Verify `hasPageAgent` is `true` before continuing.

### 5. Create or reuse one agent instance

Store the instance on `window` so later calls can reuse it:

```javascript
async () => {
  window.__pageAgent = window.__pageAgent || new window.PageAgent({
    baseURL: 'http://localhost:11434/v1',
    apiKey: 'NA',
    model: 'qwen3:14b',
    language: 'zh-CN'
  });
  return { ready: !!window.__pageAgent };
}
```

Adapt the config to the user's actual model endpoint.

### 6. Execute one concrete instruction

Call `execute` from `evaluate_script`:

```javascript
async () => {
  const result = await window.__pageAgent.execute('Click the submit button and confirm the dialog');
  return result;
}
```

Keep instructions concrete and page-local unless the extension-backed workflow is already confirmed. Good examples:

- `Open the profile menu and switch language to Chinese`
- `Fill the search box with "invoice" and submit`
- `Create a new row with quantity 3 and save`

Prefer one goal per call. If the task is long, break it into checkpoints and verify between them.

For extension-backed multi-tab tasks, prefer URL-explicit prompts such as:

- `Open https://github.com/alibaba/page-agent/blob/main/docs/README-zh.md in a new tab, ensure Releases is open in another tab, then return focus to the repository tab`
- `Open the issues page in a new tab, collect the top 3 issue titles, then switch back to the code tab`

### 7. Verify with browser tools

- Take another snapshot.
- Confirm the page state actually changed as intended.
- If the action partially succeeded, report exactly what changed and what remains unresolved.
- For multi-tab tasks, verify the final focused tab with `list_pages` plus a page snapshot or `location.href` check.
- If the agent claims it returned to a target tab but the actual selected tab or URL does not match, correct it manually with browser MCP actions and report the mismatch as an agent limitation.

### 8. Enforce final tab state for multi-page work

When the task specifies which tab should be focused at the end:

1. Record the intended end tab before execution when possible.
2. After `window.PAGE_AGENT_EXT.execute(...)` finishes, immediately verify the selected page and current URL.
3. If the intended tab is missing, say so explicitly instead of claiming success.
4. If the intended tab still exists but is not focused, switch back with browser MCP tools.
5. Only report success after the final tab state is externally verified.

## When To Skip PageAgent

- Skip it for simple deterministic actions that `click`, `fill`, `press_key`, or direct DOM inspection can handle more reliably.
- Skip it when the page blocks script injection and the task does not justify extra setup.
- Skip it on sensitive workflows unless the user explicitly wants PageAgent and accepts the tradeoff.

## Failure Handling

- If script injection fails, fall back to normal `chrome-devtools` actions.
- If a site blocks CDN injection with CSP, prefer extension-backed mode instead of retrying other CDN mirrors.
- If PageAgent reports missing config, stop and supply a valid `baseURL`/`apiKey`/`model`.
- If the page blocks script execution or CSP prevents loading the CDN asset, explain that the page cannot be driven this way and use ordinary browser tools instead.
- If multi-page execution returns correct data but leaves the wrong tab focused, treat that as partial success and repair the tab state manually.
- If the task is sensitive, prefer manual deterministic steps over autonomous natural-language execution.

## Integration Guidance For Codex

Translate the official PageAgent integration model into Codex like this:

1. Use `chrome-devtools-real` as the preferred browser transport.
2. Reuse the user's real Chrome profile when Page Agent Ext is already installed there.
3. Invoke PageAgent from page context for the active tab.
4. Use ordinary browser MCP actions to select pages, gather state, and verify outcomes.
5. Fall back to plain `chrome-devtools` only when the real-browser route is unavailable.

Treat this skill as a browser-side adapter pattern, not as a standalone MCP server.
