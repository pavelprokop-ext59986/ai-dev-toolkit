---
name: playwright-browser-automation
description: Use Playwright from the command line to inspect, test, and automate browser interactions when a task requires reading or validating web UI behavior.
license: Proprietary
---

# Playwright Browser Automation

## Purpose

Use Playwright as a safe command-line browser automation tool for inspecting, testing, and validating web applications.

This skill is useful when Copilot needs to:
- inspect rendered web pages
- verify Angular/React UI behavior
- check forms, buttons, navigation, and visible text
- capture screenshots
- extract DOM/text content
- reproduce browser-side bugs
- validate local frontend behavior

## Core Principle

Do not pretend to directly see the browser.

Instead:
1. create or use a Playwright script
2. run it locally
3. print structured output
4. reason from the output

## Safety Rules

- Do not enter passwords, tokens, bank data, or sensitive personal data unless the user explicitly instructs and approves.
- Do not perform destructive actions such as submitting real orders, deleting data, approving workflows, or changing production data.
- Prefer local/dev/test environments.
- Clearly state which URL/environment is being accessed.
- Prefer read-only inspection unless interaction is explicitly requested.
- Save scripts under a temporary or task-specific location unless the user asks to commit them.
- Do not commit generated Playwright helper scripts unless explicitly requested.

## Preferred Script Location

Use one of:

```text
/tmp/copilot-playwright/
```

or repo-local temporary path:

```text
.copilot/tmp/playwright/
```

Create the directory if needed.

## Basic Inspection Workflow

1. Identify target URL.
2. Create a small Playwright script.
3. Navigate to the page.
4. Wait for page load or specific selector.
5. Extract:
   - page title
   - URL
   - visible text
   - relevant selectors
   - console errors if useful
6. Optionally capture screenshot.
7. Print concise structured output.

## Example Script

```js
const { chromium } = require('playwright');

(async () => {
  const browser = await chromium.launch({ headless: true });
  const page = await browser.newPage();

  page.on('console', msg => {
    if (msg.type() === 'error') {
      console.log('[console:error]', msg.text());
    }
  });

  await page.goto(process.env.TARGET_URL, { waitUntil: 'networkidle' });

  const title = await page.title();
  const bodyText = await page.locator('body').innerText();

  console.log(JSON.stringify({
    url: page.url(),
    title,
    bodyTextPreview: bodyText.slice(0, 5000)
  }, null, 2));

  await page.screenshot({ path: '/tmp/copilot-playwright/screenshot.png', fullPage: true });

  await browser.close();
})();
```

Run with:

```bash
TARGET_URL="http://localhost:4200" node /tmp/copilot-playwright/inspect.js
```

## Local Project Usage

### This environment — Homebrew global install

Playwright is installed globally via Homebrew. Scripts **must** be run with:

```bash
NODE_PATH=/opt/homebrew/lib/node_modules node /tmp/copilot-playwright/script.js
```

Plain `node script.js` will fail with `Cannot find module 'playwright'`. Always use `NODE_PATH`.

Binary location: `/opt/homebrew/bin/playwright` → `/opt/homebrew/lib/node_modules/playwright/`

### Generic fallback

If project-local Playwright is available, prefer:

```bash
npx playwright test
```

or:

```bash
npx playwright node script.js
```

If Playwright is missing, ask the user before installing dependencies.

## Authentication

Authentication is tricky.

Prefer one of:
- user manually logs in and provides a test URL/state
- use test credentials only when explicitly approved
- use existing saved browser profile only when explicitly approved

Do not attempt to bypass SSO/MFA.

### ⚠️ Navigate to a real page, not the app root

Many SPAs (including `http://localhost:4200`) have **no landing page** — navigating to the root may immediately redirect to OIDC/SSO login, or silently fail with a blank page. Always use a **real, known page URL** (e.g. a specific CA or entity URL) when checking authentication status, not just the root `/`. The user should provide such a URL at the start of the task.

## Headed Mode

For debugging visible browser behavior, use headed mode:

```js
const browser = await chromium.launch({ headless: false });
```

Use headed mode when:
- login requires human interaction
- visual behavior matters
- user explicitly wants to observe the browser

## Screenshots

When useful, save screenshots to:

```text
/tmp/copilot-playwright/screenshot.png
```

or task-specific names.

Always mention screenshot path in final output.

## Output Expectations

After running Playwright, report:
- script used
- URL inspected
- key findings
- console errors if any
- screenshot path if captured
- whether the observed behavior matches expectations

## When Not To Use

Do not use Playwright for:
- simple static code questions
- backend-only logic
- destructive production workflows
- tasks that can be solved more safely by reading code/tests

## Authentication With Manual Login and Saved Session

Prefer saved browser session state over storing credentials.

Default auth state path:

```text
.copilot/playwright/auth-state.json
```

This file may contain active session cookies or tokens and must not be committed.

Add to `.gitignore`:

```gitignore
.copilot/playwright/auth-state.json
```

### Authentication workflow

1. Check whether `.copilot/playwright/auth-state.json` exists.
2. If it exists, create the browser context with this storage state.
3. Navigate to the **target page URL** (not just the app root — see warning above).
4. Detect whether the user is already authenticated (URL still on app, body has real content).
5. If authenticated, continue automation.
6. If redirected to login or authentication is expired:
   - Open browser in **headed mode** (`headless: false`)
   - Tell the user to log in — the browser will stay open
   - **Use a flag-file pattern** to wait for the user to signal completion (avoids stdin/readline issues in async shells)
   - After the flag appears, save storage state and continue

### Do not store credentials

Do not store username, password, tokens, or MFA secrets in files.

The user should enter credentials manually in the browser when authentication is required.

### Flag-file pattern for manual login (recommended)

This is the most reliable way to wait for manual login. Copilot creates a flag file when the user says they're done, so the browser stays open as long as needed.

```js
const { chromium } = require('playwright');
const fs = require('fs');

const AUTH_STATE_PATH = '.copilot/playwright/auth-state.json';
const FLAG_FILE = '/tmp/copilot-playwright/ready.flag';
// Use a real known page URL, not just the app root
const TARGET_URL = process.env.TARGET_URL;

(async () => {
  // Always remove stale flag before starting
  if (fs.existsSync(FLAG_FILE)) fs.unlinkSync(FLAG_FILE);

  const browser = await chromium.launch({ headless: false });
  let context;

  if (fs.existsSync(AUTH_STATE_PATH)) {
    context = await browser.newContext({ storageState: AUTH_STATE_PATH });
  } else {
    context = await browser.newContext();
  }

  const page = await context.newPage();
  await page.goto(TARGET_URL, { waitUntil: 'domcontentloaded', timeout: 15000 }).catch(() => {});

  // Check if login is needed (redirected away from app, or body is empty)
  const needsLogin = !page.url().startsWith('http://localhost:4200') ||
    (await page.locator('body').innerText().catch(() => '')).trim().length < 100;

  if (needsLogin) {
    console.log('Please log in in the browser window. Tell Copilot when done.');
    // Wait for flag file (Copilot touches it when user says "done")
    for (let i = 0; i < 600; i++) {
      await new Promise(r => setTimeout(r, 1000));
      if (fs.existsSync(FLAG_FILE)) { console.log('Flag detected, continuing.'); break; }
    }
  }

  // Save auth state
  fs.mkdirSync('.copilot/playwright', { recursive: true });
  await context.storageState({ path: AUTH_STATE_PATH });
  console.log('Auth state saved.');

  // Now navigate to target and inspect
  await page.goto(TARGET_URL, { waitUntil: 'domcontentloaded', timeout: 30000 }).catch(() => {});
  await page.waitForTimeout(5000);

  // ... inspect page here ...

  await browser.close();
})();
```

**When the user says "done" (logged in), Copilot runs:**
```bash
touch /tmp/copilot-playwright/ready.flag
```

### Checking if saved auth state is still valid

After loading saved auth state, navigate to a **real deep-link URL** (not `/`), wait 3s, then check:
- URL is still on `localhost:4200` (not redirected to ADFS/SSO)
- `body` inner text has substantial content (> 100 chars)

If either fails, discard the saved state and prompt for fresh login.

### ⚠️ Auth state is short-lived (ADFS/OIDC)

The app uses ADFS OIDC (`https://fsidp.cs-test.cz/adfs/...`). Saved session tokens expire quickly — even within the same working session. **Always verify auth before any headless run.** If the check fails, delete the saved state and re-run the login flow.

```js
// Delete stale auth and re-trigger login flow
fs.unlinkSync(AUTH_STATE_PATH);
```

### Use absolute paths for auth-state.json

Scripts run from `/tmp` — relative paths like `.copilot/playwright/auth-state.json` resolve incorrectly. Always use the absolute path:

```js
const AUTH_STATE_PATH = '/Users/ext59986/projects/.copilot/playwright/auth-state.json';
```

### Notes

- **Never navigate to app root to test auth** — many SPAs have no landing page and will immediately redirect to SSO even with valid tokens. Use a deep-link URL.
- Authentication detection should prefer content-length check over URL-only checks.
- `readline/promises` stdin prompts do not work reliably in Copilot's async shell — always use the flag-file pattern instead.
- `waitForFunction` with options must pass `null` as the second arg: `page.waitForFunction(fn, null, { timeout: 60000 })`.
- If login state expires, delete `auth-state.json` and repeat the manual login flow.
- Never commit `auth-state.json`.

## Angular SPA — Rendering

Angular apps need extra time to fully render after navigation. Use `networkidle` + an explicit extra wait:

```js
await page.goto(TARGET_URL, { waitUntil: 'networkidle', timeout: 30000 });
await page.waitForTimeout(5000); // wait for Angular change detection
```

`domcontentloaded` is not enough — components may not have rendered yet.

## DOM Extraction — Finding Field Values in Angular

Playwright's `:has-text()` locator and CSS sibling selectors are **unreliable** in Angular apps due to shadow DOM and dynamic component structure. Instead, use a `TreeWalker` inside `page.evaluate()` to find exact text nodes, then walk up the DOM to find the nearest `input` or `textarea`:

```js
const result = await page.evaluate((labelText) => {
  const walker = document.createTreeWalker(document.body, NodeFilter.SHOW_TEXT);
  let node;
  while ((node = walker.nextNode())) {
    if (node.textContent.trim() === labelText) {
      let container = node.parentElement;
      for (let i = 0; i < 8; i++) {
        const field = container.querySelector('input, textarea');
        if (field) return { value: field.value, tag: field.tagName };
        container = container.parentElement;
        if (!container) break;
      }
    }
  }
  return null;
}, 'My Field Label');
```

This is the most reliable pattern for Angular form field inspection.
