---
description: Wire idle-guard status indicator into your Claude Code status bar
allowed-tools: Read, Write, Bash, Glob, Grep
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

   **Case A — an existing statusLine command is present**:

   Investigate what it does before touching it:

   a. Read the command string and identify any plugin or binary being invoked
      (e.g. a bun/node script, a shell script, a compiled binary).

   b. If it invokes a plugin, locate the plugin's installed source and read its
      code to understand how it works — specifically whether it provides a
      documented and **actually implemented** integration point for appending
      extra output (e.g. a CLI flag, a config field, a hook). Do not trust
      documentation or flag names alone; verify the integration point is wired
      up in the code that runs. Dead flags and unimplemented features are common.

   c. If a working integration point exists, use it. If none exists, use the
      universal fallback: run `$STATUS_SCRIPT` as a separate command after the
      existing command and append its label as a plain output line.

   In either case, the final command must:
   - Read stdin once into a variable (`INPUT=$(cat)`) and pass it to any
     command that needs it, since stdin can only be consumed once.
   - Extract transcript_path:
     `TRANSCRIPT=$(printf '%s' "$INPUT" | jq -r '.transcript_path // empty')`
   - End with `; true` so the command always exits 0. A non-zero exit code
     causes Claude Code to hide the entire status bar silently.

   The universal fallback structure (fill in real paths):
   ```
   bash -c 'INPUT=$(cat); TRANSCRIPT=$(printf "%s" "$INPUT" | jq -r ".transcript_path // empty"); printf "%s" "$INPUT" | <existing-command>; "$STATUS_SCRIPT" "$TRANSCRIPT" | jq -r ".label // empty" | grep -v "^$"; true'
   ```

   The new command must be a single `bash -c '...'` string. Write the actual
   resolved path to `$STATUS_SCRIPT`, not a shell variable reference, since
   `settings.json` does not expand environment variables.

   **Case B — no statusLine present**:

   Set a minimal statusLine that runs only the `status` script:
   ```json
   {
     "type": "command",
     "command": "bash -c 'INPUT=$(cat); TRANSCRIPT=$(printf \"%s\" \"$INPUT\" | jq -r \".transcript_path // empty\"); /real/path/to/idle-guard/hooks/status \"$TRANSCRIPT\" | jq -r \".label // empty\" | grep -v \"^$\"; true'",
     "refreshInterval": 10
   }
   ```

   In both cases (A and B), set `"refreshInterval": 10` on the statusLine object.
   idle-guard tracks elapsed idle time, so the status bar must refresh on a
   timer to show the warning as soon as the cache expires — not only when Claude
   responds to a message.

4. Write the updated `settings.json` back.

5. Report exactly what you changed and, if you investigated a plugin, what you
   found about its integration surface.
