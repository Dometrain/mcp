# Dometrain MCP Server

Connect your coding agent to [Dometrain](https://dometrain.com) course content. The
Dometrain MCP server lets agents like Claude Code, Codex, GitHub Copilot, and OpenCode
search Dometrain video-course lessons and pull curated lesson documents — summaries,
cleaned lesson notes, and the exact code shown on screen, each with a timestamped deep
link into the video — so your agent implements things the way your courses teach them.

- **Endpoint:** `https://mcp.dometrain.com/mcp` (streamable HTTP)
- **Auth:** Dometrain account API key as a Bearer token
- **Access:** requires an active [Dometrain Pro](https://dometrain.com/pro/) subscription
- **Server card:** https://dometrain.com/.well-known/mcp/server-card.json

## 1. Get an API key

1. Sign in at [dometrain.com](https://dometrain.com) and open
   [Account settings](https://dometrain.com/dashboard/account/).
2. In **MCP API keys**, create a key and copy it (it is shown exactly once).
3. Export it where your agent runs:

```bash
export DOMETRAIN_API_KEY="dmt_live_..."
```

## 2. Install

### Claude Code

Quickest — add the server directly:

```bash
claude mcp add --transport http dometrain https://mcp.dometrain.com/mcp \
  --header "Authorization: Bearer ${DOMETRAIN_API_KEY}"
```

Or install the plugin (adds the server **plus** a skill that teaches Claude when to
consult Dometrain and how to cite lessons):

```bash
claude plugin marketplace add dometrain/mcp
claude plugin install dometrain@dometrain
```

### Codex

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.dometrain]
url = "https://mcp.dometrain.com/mcp"
bearer_token_env_var = "DOMETRAIN_API_KEY"
```

### GitHub Copilot (VS Code)

Add to your workspace `.vscode/mcp.json` (or user-level `mcp.json`):

```json
{
  "servers": {
    "dometrain": {
      "type": "http",
      "url": "https://mcp.dometrain.com/mcp",
      "headers": {
        "Authorization": "Bearer ${env:DOMETRAIN_API_KEY}"
      }
    }
  }
}
```

### OpenCode

Add to `opencode.json`:

```json
{
  "mcp": {
    "dometrain": {
      "type": "remote",
      "url": "https://mcp.dometrain.com/mcp",
      "headers": {
        "Authorization": "Bearer {env:DOMETRAIN_API_KEY}"
      }
    }
  }
}
```

## Tools

| Tool | What it does |
|---|---|
| `search_dometrain(query, tech?, max_results?)` | Search lessons for implementation guidance; hybrid keyword + semantic ranking, so conceptual queries work well — ranked excerpts with deep links (a note flags partial-term or semantic-only matches) |
| `search_code(query, language?, max_results?)` | Search the code shown on screen in lessons; snippets with a deep link to the exact moment the code appears |
| `get_lesson(lesson_id)` | Full curated lesson document: summary, key concepts, notes, on-screen code |
| `get_course(course_id_or_slug)` | Course overview + chapter/lesson tree |
| `list_courses(topic?)` | Published courses, optionally filtered by topic |
| `get_usage()` | Your monthly request usage, limit, and reset date |

## Notes

- Requests are limited per calendar month per account (all your keys share the pool) with
  a short per-minute burst cap; `get_usage` shows where you stand.
- Keys can be revoked any time from account settings; treat them like passwords.
- Content covers **video lessons** of published courses. Hands-on exercises, quizzes, and
  their solutions are intentionally not exposed.

## Troubleshooting

- **401** — missing/invalid/revoked key. Create a new key in account settings.
- **403** — your account has no active Dometrain Pro subscription.
- **429** — monthly limit reached (response includes the reset date) or per-minute burst
  cap hit (retry shortly).

## License

This plugin (manifests, configuration, skill, and docs) is [MIT licensed](LICENSE).
The Dometrain course content served by the MCP server is **not** covered by this
license — it remains Dometrain's proprietary content, accessible under your
Dometrain Pro subscription and the [Dometrain terms of service](https://dometrain.com/terms/).
