---
description: Wire idle-guard status indicator into your Claude Code status bar
allowed-tools: Read, Write
---

You are configuring the idle-guard plugin's status bar integration.

## Your task

1. Read `~/.claude/settings.json`.

2. Find the installed path of the idle-guard `status` script:

   ```bash
   ls ~/.claude/plugins/cache/*/idle-guard/*/hooks/status 2>/dev/null \
     || ls ~/.claude/plugins/cache/*/*/idle-guard/*/hooks/status 2>/dev/null
   ```

   Use the path of the most recently installed version (newest directory by
   modification time). Call this `$STATUS_SCRIPT`.

3. Inspect `settings.statusLine.command` (the field may be absent):

   **Case A — claude-hud present** (command contains `claude-hud`):

   Wrap the existing command so it:
   - Reads stdin once into a variable: `INPUT=$(cat)`
   - Extracts transcript_path: `TRANSCRIPT=$(printf '%s' "$INPUT" | jq -r '.transcript_path // empty')`
   - Pipes original stdin to the existing claude-hud command AND appends
     `--extra-cmd "$STATUS_SCRIPT \"$TRANSCRIPT\""` to the claude-hud
     invocation.

   The new command must be a single bash -c '...' string. Write the actual
   resolved path to `$STATUS_SCRIPT`, not a shell variable reference, since
   `settings.json` does not expand environment variables.

   Example of the wrapped command structure (fill in real paths):
   ```
   bash -c 'INPUT=$(cat); TRANSCRIPT=$(printf "%s" "$INPUT" | jq -r ".transcript_path // empty"); printf "%s" "$INPUT" | bun "/real/path/to/claude-hud/src/index.ts" --extra-cmd "/real/path/to/idle-guard/hooks/status \"$TRANSCRIPT\""'
   ```

   **Case B — other custom statusLine present**:

   Read the existing command, determine its output format, and append the
   idle-guard label as an additional line or segment in a way that is
   consistent with the existing output. Preserve all existing output.

   **Case C — no statusLine**:

   Set a minimal statusLine that runs only the `status` script:
   ```json
   {
     "type": "command",
     "command": "bash -c '/real/path/to/idle-guard/hooks/status \"$(cat | jq -r .transcript_path)\"'"
   }
   ```

4. Write the updated `settings.json` back.

5. Report exactly what you changed.
