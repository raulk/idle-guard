# idle-guard

A Claude Code plugin that blocks resuming conversations whose prompt cache has
expired, preventing surprise cache-resurrection costs.

## How it works

The Anthropic prompt cache has a 5-minute TTL. If you leave a conversation
idle longer than that and then send a message, the entire conversation context
must be re-uploaded to the cache, which can be expensive for long sessions.

idle-guard tracks when Claude last responded. If you submit a new message after
the cache has expired, it blocks the send and tells you:

```
Session idle for 8m 14s (limit: 5m). The prompt cache has expired.
Resuming will trigger cache rewrites for ~45k tokens.
Press up-arrow to resend within the next 5m, or start a new conversation.
```

Press up-arrow to resend your message. You have one cache TTL window (default 5m) to do so before the block triggers again.

## Installation

**Step 1.** Register the marketplace and enable the plugin in
`~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "idle-guard": {
      "source": {
        "source": "github",
        "repo": "raulk/idle-guard"
      }
    }
  },
  "enabledPlugins": {
    "idle-guard@idle-guard": true
  }
}
```

**Step 2.** Run the setup skill in any Claude Code session to wire up the
status bar:

```
/idle-guard:setup
```

That's it. The plugin starts blocking expired-cache resumes immediately.
The setup skill configures the status bar indicator alongside any existing
statusLine plugin you already have.

## Configuration

Create `~/.claude/plugins/data/idle-guard/config.json`:

```json
{ "timeout_minutes": 5 }
```

The default is 5 minutes, matching the Anthropic prompt cache TTL.

## Status bar

After running `/idle-guard:setup`, your status bar will show:

| State | Label |
|-------|-------|
| Fresh (< 80% of timeout) | *(nothing)* |
| Approaching expiry (≥ 80%) | `⏰ 30s left, 45k tokens at risk` |
| Expired | **`⚠ cache expired (3m ago), 45k tokens`** (bold yellow) |

The setup skill works with any existing statusLine configuration. It reads
your current statusLine plugin's source to find a working integration point,
and falls back to appending the idle-guard label as a plain output line if
none exists.

## Requirements

- Claude Code with plugin support
- `jq` (required by Claude Code itself)
- `shasum` or `sha1sum` (standard on macOS and Linux)
