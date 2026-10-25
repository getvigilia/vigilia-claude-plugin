---
name: fix
description: Fix what is broken in the user's live app using Vigilia. Finds the worst open incident (or the one named), reads Vigilia's plain-language diagnosis, evidence and fix prompt, fixes the cause in this codebase, and hands it back to Vigilia to verify with real traffic. Use when the user asks to fix Vigilia incidents, asks what is broken in production, or pastes a Vigilia incident id or link.
when_to_use: "Examples: \"fix my Vigilia incidents\", \"what's broken in prod?\", \"fix incident abc123\", \"Vigilia emailed me about checkout — fix it\"."
argument-hint: "[incident-id]"
---

# Fix a Vigilia incident

Vigilia watches this person's live app. Its MCP tools (server `vigilia`) give you the diagnosis and the evidence for each problem. Your job is to fix the cause in this codebase and hand the incident back to Vigilia, which then confirms with real traffic whether the fix held.

## Before you start

This needs Vigilia's tools (`list_open_incidents`, `get_incident`, `mark_fixed`), which come from this plugin's `vigilia` server. If they aren't available, Vigilia isn't signed in yet. Don't look for config files, tokens or other servers. Tell the user:

> Vigilia isn't connected yet. Type `/mcp`, choose `plugin:vigilia:vigilia` (choose **Enable** first if it says disabled), then **Authenticate** and click **Allow** on the Vigilia page that opens. Then ask me again.

Then stop.

## 1. Pick the incident

- If an incident id or a Vigilia incident link was given (`$ARGUMENTS`), use that id. A link ends in `/incidents/<id>`.
- Otherwise call `list_open_incidents` and take the first one. It is sorted worst first. If several apps are listed and the one this repo builds is obvious, use `list_open_incidents` with that `appId`.
- Tell the user in one line which incident you're taking and why.

If there are no open incidents, say so and stop. Don't go looking for problems Vigilia didn't report.

## 2. Read it

Call `get_incident`. Read all of it before touching code:

- **What's broken / who's affected**: the symptom, in the user's words.
- **Evidence**: the error message, the page the visitor was on, and, for a failed request, the endpoint that failed. Also the stack frames, and what the visitor did right before.
- **Vigilia's fix prompt**: a strong lead written by a model that never saw this code. Verify it against the code instead of applying it blindly.

Two details in the evidence are easy to misread:

- The page is where the visitor was standing, not necessarily where the fault is. For a failed request, the fault is at the request target.
- `HTTP 0` means the request never completed. That is network-level, not an application bug.

## 3. Find the cause

- Search for the stack frames' file and function names first. Minified frames still carry component names and string literals.
- For a failed request, find where the app calls that endpoint, and whether this repo also serves it.
- If the fault is outside this repo (a third-party API that is down, a backend in another repo, a database rule such as a Supabase RLS policy), say so plainly. Fix what is fixable here, such as handling the failure gracefully, and tell the user what must change elsewhere.
- If the evidence contradicts the diagnosis, trust the evidence and tell the user.

## 4. Fix it

- Make the smallest change that removes the cause. Don't refactor unrelated code.
- The fix should also leave the user-facing behaviour sane: a visitor should see a helpful message, not a frozen screen.
- If the project has tests, add a regression test that fails without the fix and passes with it.
- Run the project's own checks (typecheck, lint, tests) where they exist.

## 5. Hand it back

- Explain in plain language what broke, what you changed, and how to see that it works. The user may not read code.
- Vigilia's verification only means something once real visitors run the new code. Call `mark_fixed` **after the fix is deployed**:
  - If the user deploys by pushing (Vercel, Netlify, Lovable…) and asked you to commit and push, push first, then call `mark_fixed`.
  - Otherwise ask: "Once this is deployed, shall I mark it fixed in Vigilia so it can confirm with real traffic?"
- If several open incidents share this root cause, mark each of them fixed.
- Afterwards, `get_fix_status` tells whether the fix held. After about a day of traffic it reads confirmed; if the error comes back, the incident reopens.

Never print, log or commit the Vigilia API token.
