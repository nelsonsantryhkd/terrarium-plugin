# Terrarium plugin for Claude Code

Give your agent a place to show its work to humans. [Terrarium](https://www.tryterrarium.com)
hosts the HTML your agent builds; humans review it in the browser with
comments anchored to exact passages; the agent pulls that feedback over MCP,
pushes fixes, and repeats until approved.

## Install

```
/plugin marketplace add nelsonsantryhkd/terrarium-plugin
/plugin install terrarium@terrarium
```

Then mint a token at [tryterrarium.com/connect](https://www.tryterrarium.com/connect)
and make it available as `TERRARIUM_PAT`:

```sh
export TERRARIUM_PAT="samus_pat_…"
```

(Or just ask your agent to set Terrarium up — the bundled skill walks it
through the whole flow, including sending you to /connect for the token.)

## What it ships

- **MCP server** (`.mcp.json`) — Terrarium's remote server over Streamable
  HTTP with 7 tools: `list_artifacts`, `publish_artifact`, `get_artifact`,
  `pull_comments`, `push_version`, `reply_to_comment`, `resolve_comment`.
- **Skill** (`skills/terrarium/`) — teaches the publish → review → fix loop,
  including the `terrarium-feedback/1` payload, `expectedVersion` freshness,
  and comment-thread etiquette.

## The protocol

`pull_comments` returns a versioned, structured payload
(`terrarium-feedback/1`) designed to be stable enough to build agents against:
review state, typed comment threads (`change_request` / `question` / `bug`),
text anchors with live resolution status, and reply chains. Evolution within
`/1` is additive-only.
