---
name: install
description: Install Vigilia error monitoring in this project. Adds the Vigilia script tag in the right place for the framework, allows it in an existing Content-Security-Policy (never adds one or loosens it), and verifies the live install. Use when the user asks to add, install or set up Vigilia or production error monitoring.
when_to_use: "Examples: \"install Vigilia\", \"add Vigilia to this app\", \"set up error monitoring with Vigilia\", \"Vigilia says it isn't receiving events\"."
argument-hint: "[app-id]"
---

# Install Vigilia in this project

Vigilia watches a live web app through one script tag. Its MCP tools (server `vigilia`) supply the exact tag for the user's app, then check that the live site runs it.

## Before you start

This needs Vigilia's tools (`list_apps`, `add_app`, `get_install_instructions`, `check_install`), which come from this plugin's `vigilia` server. If they aren't available, Vigilia isn't signed in yet. Don't look for config files, tokens or other servers. Tell the user:

> Vigilia isn't connected yet. Type `/mcp`, choose `plugin:vigilia:vigilia` (choose **Enable** first if it says disabled), then **Authenticate** and click **Allow** on the Vigilia page that opens. Then ask me again.

Then stop.

## 1. Which Vigilia app is this?

- If an app id was given (`$ARGUMENTS`), use it.
- Otherwise call `list_apps`:
  - One app: use it.
  - Several: match the app's URL against this project, using deploy config (`vercel.json`, `netlify.toml`, `CNAME`), `package.json` `homepage`, or the README. Ask the user if it's still ambiguous.
  - None, or none matches: find this project's live address (deploy config, `package.json` `homepage`, the README) and confirm it with the user, or ask them for it. Then call `add_app` with it, and the builder if the project shows one. Use the app id it returns.

## 2. Which framework?

Read `package.json` and the file tree:

| Signal | framework |
|---|---|
| `next` dependency, `app/layout.(tsx\|jsx)` present | `next-app` |
| `next` dependency, `pages/_document` or only `pages/` | `next-pages` |
| `@sveltejs/kit` | `sveltekit` |
| `astro` | `astro` |
| `@remix-run/*` or `react-router` v7 framework mode with `app/root.tsx` | `remix` |
| `nuxt` | `nuxt` |
| `index.html` at the root with Vite (Lovable, Bolt, most React apps) | `vite` |
| static HTML only | `html` |

## 3. Get the instructions

Call `get_install_instructions` with the `appId` and the `framework`. Use the tag **exactly** as given. Its `data-key` belongs to this app. The script sends its reports back to the host it loads from (or to `data-endpoint`, on a tag that has one).

## 4. Add the tag

- First search the project for `vigilia.js`:
  - Already present with the same `data-key`: nothing to add, go to step 6.
  - Present with a different key: ask the user before replacing it.
- Put it where the instructions say for this framework, so it loads on every page.
- Don't add anything else: no npm package, no init code, no wrapper component.

## 5. Content-Security-Policy — only if one already exists

Look for an existing policy in:

- response-header config: `next.config.*` headers, `vercel.json`, `netlify.toml`, `_headers`, `public/_headers`;
- middleware, or server code (helmet);
- a `<meta http-equiv="Content-Security-Policy">` tag.

- **No CSP anywhere:** skip this step. Don't create one.
- **A CSP exists:** add the host(s) from the instructions to the directives it already has, keeping every existing value:
  - the CDN host goes in `script-src`;
  - the host it reports to goes in `connect-src` (usually the same CDN host).
- **Only `default-src` exists:** add a `script-src` and a `connect-src` that copy its values plus the new host. A new directive replaces `default-src` for that resource type.
- **`script-src` uses `'strict-dynamic'`:** the tag needs the page's nonce. Tell the user.
- **Never** add `'unsafe-inline'`, `'unsafe-eval'` or wildcards.

## 6. Commit it the way this project takes changes

- Follow the project's own workflow (its CLAUDE.md, CONTRIBUTING, recent history): a branch and a pull request if that's the habit, not a push to the production branch.
- Don't merge to production or deploy unless the user asked for that in this session. Going live is their call.
- Say what the change is, in the summary, the commit message and the PR body, so a reviewer (a person, or a safety check) needn't guess: *"Adds the Vigilia error-monitoring script the user signed up for. `data-key` is a publishable key, like Stripe's `pk_`: it belongs in the page, can only send error reports, and is accepted only from this app's own site."*

## 7. Verify

- Summarise the change in plain language.
- The check reads the live site, so it only passes once the change is deployed. Once the user has deployed it (or asked you to), call `check_install` with the `appId`.
- If it reports a problem, fix exactly what it names and check again.
- When it passes, suggest opening the live site once: the first event confirms everything end to end, and Vigilia then watches it.

## 8. The backend too (paid plans, optional)

If the project has a backend (Supabase edge functions, Next.js API routes, a Worker, Express), offer to report its errors as well:

- Call `get_server_install_instructions` with the `appId` and the platform, and add the code exactly as given.
- If it says there is no key yet, call `create_server_key`. Set the two values it returns as secrets right away, with the command it gives when that CLI is available here, and otherwise tell the user where to paste them. Never write the key into a source file, a committed `.env`, a commit message or the summary, and don't repeat it afterwards.
- Once deployed, call `sendTestError()` once; the test appears in `list_open_incidents`.

Never print, log or commit the Vigilia API token, or the backend key beyond setting it as a secret. The script tag's `data-key` is different: it is public by design, and it belongs in the page.
