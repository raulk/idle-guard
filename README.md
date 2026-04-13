# idle-guard

A Claude Code plugin that blocks resuming conversations whose prompt cache has expired, **preventing surprise cache-resurrection costs**.
Kinda sad we need this, but we do.

## How it works

The Anthropic prompt cache has a 5-minute TTL. If you leave a conversation
idle longer than that and then send a message, the entire conversation context
must be re-written to the cache, which can be expensive, especially for long sessions!

idle-guard tracks when Claude last responded and blocks your next message if you
submit it after the cache has expired:

```
Session idle for 8m 14s (limit: 5m). The prompt cache has expired.
Resuming will trigger cache rewrites for ~45k tokens.
Press up-arrow to resend within the next 5m, or start a new conversation.
```

You can always smile, open up your (token) wallet, and press up-arrow to forcibly resend your message.

## Installation

```sh
❯ /plugin marketplace add raulk/idle-guard
❯ /plugin install idle-guard@idle-guard
❯ /reload-plugins
❯ /idle-guard:setup     # this will wire in a status
```

## Customize

If you want to override the TTL, you can update it here:

Create `~/.claude/plugins/data/idle-guard/config.json`:

```json
{ "timeout_minutes": 5 }
```

The default is 5 minutes, matching the Anthropic prompt cache TTL.

I will track future TTL changes and update the plugin accordingly.

## Status bar

After running `/idle-guard:setup`, your status bar will show:

| State                      | Label                                                    |
| -------------------------- | -------------------------------------------------------- |
| Fresh (< 80% of timeout)   | _(nothing)_                                              |
| Approaching expiry (≥ 80%) | `⏰ 30s left, 45k tokens at risk`                        |
| Expired                    | **`⚠ cache expired (3m ago), 45k tokens`** (bold yellow) |

The setup skill works with any existing statusLine configuration. It reads
your current statusLine plugin's source to find a working integration point,
and falls back to appending the idle-guard label as a plain output line if
none exists.

## Requirements

- Claude Code with plugin support
- `jq` (required by Claude Code itself)
- `shasum` or `sha1sum` (standard on macOS and Linux)

## License

MIT.
