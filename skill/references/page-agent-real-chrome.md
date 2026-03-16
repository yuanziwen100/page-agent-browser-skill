# PageAgent Real Chrome Notes

Use this reference only when the task needs the exact operating model or the current setup is failing.

## Official capability summary

- PageAgent is a page-level AI agent driven from JavaScript in the browser.
- Direct script injection is naturally scoped to the current tab.
- Cross-page or multi-tab behavior depends on the official Chrome extension according to the upstream project documentation.
- PageAgent still requires an OpenAI-compatible model endpoint.
- The Alibaba testing endpoint is for evaluation only and should not be treated as production-safe for sensitive data.

## Preferred Codex integration

Prefer this order:

1. `chrome-devtools-real`
2. User's normal Chrome profile with Page Agent Ext already installed
3. PageAgent execution in the active page context
4. Snapshot-based verification with browser MCP tools

This keeps Codex attached to the same browser state, cookies, extensions, and tabs the user already uses.

## Real Chrome assumptions for this machine

- `chrome-devtools-real` is the MCP entry intended for `--autoConnect`.
- The target browser profile is the user's real Chrome profile, not an isolated automation profile.
- If multiple Chrome profile windows are open, auto-connect may attach to the wrong one. Keep only the intended profile active when precision matters.

## Practical boundaries

- Do not claim that Codex has direct privileged control over the extension internals unless that API is actually visible and verified in the attached page.
- If extension behavior cannot be verified, keep using PageAgent in the current tab and use browser MCP page switching for everything else.
- If CSP blocks the CDN loader, use direct browser automation instead of forcing PageAgent.

## GitHub-specific findings

- GitHub blocks external `page-agent` CDN injection with CSP, so active-tab script injection is not a reliable default there.
- The Chrome extension path works on GitHub when the extension handshake is active and `window.PAGE_AGENT_EXT` is exposed in the page.
- Multi-tab execution is real, but prompt precision matters. If the task names an ambiguous target like `Chinese README`, the agent may infer the wrong path.
- For GitHub tasks, prefer explicit URLs and explicit tab-finish instructions.
- Even when the data extraction is correct, the agent may misreport the final focused tab. Always verify the selected page after completion and repair it with browser MCP tools if needed.

## Prompt-writing pattern

Write one short, concrete instruction at a time.

Good:

- `Open the order details drawer for the first row`
- `Change the language selector to Simplified Chinese`
- `Fill the search box with "发票" and submit`

Bad:

- `Understand this site and do whatever seems right`
- `Handle everything on this page`

## Safety

- Ask before using PageAgent on payment flows, login flows, destructive admin actions, or pages containing private data.
- Verify the post-action DOM state instead of trusting agent text output.
