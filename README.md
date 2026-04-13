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
Session idle for 8m 14s (limit: 5m). The prompt cache has expired:
resuming will trigger cache rewrites for ~45k tokens.
Press up-arrow to send anyway, or start a new conversation.
```

Press up-arrow to resend your message if you want to proceed anyway.

## Installation

### From GitHub

Add to `~/.claude/settings.json`:

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

Then run `/idle-guard:setup` in any Claude Code session to configure the
status bar integration.

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
| Fresh (< 4m idle) | *(nothing)* |
| Approaching expiry (4-5m) | `⏰ 30s left, 45k tokens at risk` |
| Expired (> 5m) | `⚠ cache expired (3m ago), 45k tokens to rewrite` |

## Requirements

- Claude Code with plugin support
- `jq` (required by Claude Code itself)
- `shasum` or `sha1sum` (standard on macOS and Linux)
