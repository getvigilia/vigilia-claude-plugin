# Vigilia for Claude Code

Fix what breaks in your live app without leaving Claude Code.

[Vigilia](https://getvigilia.com) watches your web app, explains every problem
in plain language and writes a fix prompt for it. This plugin connects Claude
Code to it:

- **`/vigilia:fix`** — takes your worst open incident (or one you name), reads
  the diagnosis and the evidence, fixes the cause in your code, and hands it
  back to Vigilia. Vigilia then watches real traffic and tells you whether the
  fix held.
- **`/vigilia:install`** — adds Vigilia to the project: the script tag in the
  right place for your framework, an existing Content-Security-Policy updated
  (never loosened, never invented), and the live install verified.
- **The `vigilia` MCP server** — so you can also just ask: *"what's broken in
  prod?"*, *"did my last fix hold?"*.

## Install

```
/plugin marketplace add https://getvigilia.com/claude/marketplace.json
/plugin install vigilia@vigilia
```

Then sign in once:

1. Type `/mcp` and choose `plugin:vigilia:vigilia`. If it says disabled,
   choose **Enable** first.
2. Choose **Authenticate**. A Vigilia page opens in your browser.
3. Keep *Read & write* (so Claude can mark incidents fixed) and click **Allow**.

The plugin brings the Vigilia MCP server with it: there is nothing else to
add.

Needs Claude Code 2.1.224 or later.

## Without the plugin

The MCP server works on its own too:

```
claude mcp add --transport http --scope user vigilia https://app.getvigilia.com/mcp
```

It signs in the same way, from `/mcp`. Use one or the other, not both, or
Claude sees every Vigilia tool twice.

## Approval prompts (optional)

Claude Code asks before each Vigilia tool call, including marking an incident
fixed. The plugin doesn't change that; what to auto-approve is your call. We
recommend allowing Vigilia: its tools only read and update your incidents in
Vigilia, never your files. Add this to `~/.claude/settings.json`:

```json
{ "permissions": { "allow": ["mcp__plugin_vigilia_vigilia", "mcp__vigilia"] } }
```

The first entry is the plugin's server, the second a server added with
`claude mcp add … vigilia`. To allow only some tools, name them instead, e.g.
`mcp__plugin_vigilia_vigilia__list_open_incidents`. Change it any time with
`/permissions`.

## What the plugin runs and sends

- **Nothing runs on your machine by itself.** The plugin has no hooks, scripts
  or bundled binaries: two skills (`/vigilia:fix`, `/vigilia:install`, plain
  instructions to Claude) and one MCP server declaration.
- **One network destination:** `https://app.getvigilia.com/mcp`, Vigilia's own
  server. You sign in with OAuth; Claude Code keeps the resulting access token
  in its own credential store, never in a project file. Revoke it any time in
  Vigilia → Settings → Connected AI.
- **Edits happen in your project, in your session:** `/vigilia:install` adds
  Vigilia's script tag (and, only if you already have one, a CSP allowance) to
  your code for you to review and commit. `/vigilia:fix` changes whatever
  your code needs to fix the incident. Nothing is deployed for you.
- The one tool that writes to Vigilia is `mark_fixed`, which marks an incident
  as fixed so Vigilia can watch for it to come back. Claude Code asks before it
  runs.

## Privacy and support

Vigilia never sees your code: the fixing happens in your session. What Vigilia
stores about your account and your apps is in its
[privacy policy](https://getvigilia.com/privacy). Questions or problems:
support@getvigilia.com.

## License

MIT, see [LICENSE](LICENSE).
