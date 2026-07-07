---
name: terrarium
description: Share HTML you build with a human for visual review on Terrarium — publish an artifact, send the share link, pull their anchored comments, push fixed versions, and repeat until they approve. Use whenever the user wants human eyes on generated HTML, asks to "put this on Terrarium", or shares a tryterrarium.com/d/ link with feedback to address.
---

# Terrarium — human review for the HTML you build

Terrarium is where your human reviews the HTML you make. You publish an
artifact, they open the share link in a browser, select text, and leave
comments anchored to exact passages; they approve or request changes. You pull
that feedback over MCP and close the loop.

## One-time setup (needs the human once)

The bundled MCP server reads a personal access token from the `TERRARIUM_PAT`
environment variable. If Terrarium calls fail with 401/unauthorized (or the
token is empty):

1. Ask the human to visit **https://www.tryterrarium.com/connect**, sign in,
   and generate a token (it is shown exactly once).
2. Have them export it where you run — e.g. add
   `export TERRARIUM_PAT="samus_pat_…"` to their shell profile — then restart
   the session.

You then act *as them*: artifacts you publish belong to their account.

## The loop

1. **Publish** — `publish_artifact` with the full HTML and a clear title.
   Give the human the returned share link; review happens there.
2. **Wait for feedback** — the human comments in the browser and may
   **approve** or **request changes** (their note travels to you verbatim).
3. **Pull** — `pull_comments` on the share URL. You get a structured
   `terrarium-feedback/1` payload plus a markdown rendering:
   - `review.state`: `awaiting_review` | `approved` | `changes_requested`
     (per-version — your next push resets it to awaiting_review).
   - `threads[]`, in document order, each with a `kind`
     (`change_request` | `question` | `bug`), an anchor quoting the exact
     passage it is about, and its replies.
   - `expected_version`: the freshness token for step 5.
   - Options: `status: "open" | "resolved" | "all"` (default open).
4. **Address each thread** — `get_artifact` for the current HTML, find each
   anchored quote, make the change it asks for. Anchor `status` tells you how
   much to trust the quote: `anchored` (exact), `needs-review` (text drifted),
   `orphaned` (passage gone — use judgment).
5. **Push** — `push_version` with the updated HTML **and
   `expectedVersion: <expected_version>`**. A "Stale push:" error means someone
   pushed meanwhile: re-pull, re-apply, push again. Open comments carry
   forward, re-anchored to your new text.
6. **Close out threads** — for each thread you addressed:
   `reply_to_comment` (one sentence on what changed), then `resolve_comment`.
   If you deliberately don't act on one, reply explaining why and leave it
   open — never resolve feedback you didn't address.
7. **Repeat** — pull again until no open threads remain and the human
   approves. Questions (`kind: "question"`) usually want a reply, not an edit.

## Rules of thumb

- Artifacts are self-contained HTML (max 4 MB) — inline your CSS/JS.
- Never invent thread ids; use the ids from `pull_comments`.
- Don't push cosmetic rewrites of untouched sections — comment anchors survive
  best when unrelated text stays put.
- The share link (`…/d/<id>`) and the raw document id are interchangeable in
  every tool argument.
