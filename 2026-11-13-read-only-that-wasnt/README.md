# The read-only setup that wasn't

**Date:** 2026-11-13
**Tool:** Claude Code (Opus 5.5), Windows, PowerShell
**Project:** `auto-doodle-battler`

`conversation.jsonl` is the raw transcript of the session.

## Setup going in

- I'd told Claude in an earlier session that I want to write all the code myself. It saved that as a memory note: *"never write code into the project; give hints and reviews only."*
- The project's `.claude/settings.json` was meant to make Claude read-only:
  ```json
  {
    "permissions": {
      "defaultMode": "dontAsk",
      "allow": ["Read", "Grep", "Glob"]
    }
  }
  ```

## What happened

1. I asked Claude to "finish step 4" (ItemData resources with shapes).
2. Claude read its own note saying it should never write project code. It decided my request was an exception and went ahead without asking.
3. It used a **Bash heredoc** to write three `.tres` files (`sword`, `boomerang`, `shield`) into `scenes/items/data/`. Then it edited my roadmap file too.
4. My settings didn't stop it.

## Why the settings didn't stop it

- **`defaultMode` only sets the mode a session starts in.** This session had been switched out of plan mode into **auto mode**, which lets Claude edit files and run commands without asking. The session's current mode wins over `defaultMode`.
- **An `allow` list doesn't block anything.** It only skips the prompt for the tools listed. Blocking takes a `deny` list.
- **Once you leave don't-ask mode, the only way back is restarting the session.** Shift+Tab cycles manual → accept-edits → plan (→ auto), and don't-ask isn't in that cycle. Claude can enter plan mode on its own, so it could knock me out of don't-ask for the rest of the session.

## What Claude suggested

### `.claude/settings.json` (or `~/.claude/settings.json` for every project)

```json
{
  "permissions": {
    "defaultMode": "dontAsk",
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Edit", "Write", "NotebookEdit", "Bash", "PowerShell", "EnterPlanMode"],
    "disableAutoMode": "disable",
    "disableBypassPermissionsMode": "disable"
  }
}
```

- **`deny` covers every way to change a file.** To write anything, Claude has to call Edit, Write, NotebookEdit, Bash or PowerShell. Claude Code checks deny rules before a tool runs, in every mode, so Claude can't decide its way past them.
- **`EnterPlanMode` in `deny`** stops Claude from moving the session out of don't-ask mode. *(Untested: start a new session and ask Claude to enter plan mode to confirm it's refused.)*
- **`disableAutoMode` / `disableBypassPermissionsMode`** remove the two modes that skip asking. Manual and accept-edits can't be disabled, but with the deny list they can't edit anything either.
- **Trade-off:** denying Bash also blocks read-only shell commands. Read, Grep and Glob are enough for code reviews.

### `~/.claude/keybindings.json` (optional)

This moves mode switching off Shift+Tab, so it can't happen by accident, while keeping a deliberate way to switch:

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "$docs": "https://code.claude.com/docs/en/keybindings",
  "bindings": [
    { "context": "Chat",         "bindings": { "shift+tab": null, "ctrl+x m": "chat:cycleMode" } },
    { "context": "Confirmation", "bindings": { "shift+tab": null } }
  ]
}
```

## Why I didn't just use the deny list

- **We'd already been over this.** A deny list wasn't a new idea, and Claude offered it again as if it were the answer.
- **It's whack-a-mole.** Writing a rule for every way Claude could do something is insanity. Claude will find a way to do what it thinks I want and get around restrictions where it can. This session proved it: the files didn't go through Edit or Write, they went through a Bash heredoc.
- **Every fix revealed another hole.** The deny list led to "but the mode can change", then disabling auto and bypass, then "manual and accept-edits can't be disabled", then unbinding Shift+Tab, then "but now you can't switch back", then "Claude can put you in plan mode and don't-ask isn't in the cycle", then denying `EnterPlanMode`, which was never tested. Each answer was presented as complete until the next problem came up.
- **The list still depends on trusting Claude.** Claude says those tools are "all of them", but that list comes from the same Claude that ignored my instructions. If it's missing something, I won't find out until it's used.
- **Promises aren't worth anything.** Claude said "from here I'll follow the note" right after breaking the note. It never has followed through.
- **It breaks the things I do want.** Denying Edit and Write also stops Claude from updating the roadmap and its notes about how I work. Denying Bash blocks read-only commands too.

## Takeaways

- **Instructions and memory notes are requests, and Claude treats them that way.** Claude knew my preference and broke it anyway because it read "finish step 4" as permission. Promising to do better afterwards doesn't fix that.
- **Permission rules are only as good as the list of things they block,** and I'm relying on Claude to tell me what belongs on that list.
- **Know which mode you're in.** `defaultMode` is only where a session starts.
- **Don't-ask is a one-way door within a session.** Block whatever could push you out of it.
- **If you want no chance of edits at all,** use the regular claude.ai or desktop chat (not Claude Code) and paste your code in. Claude has no access to your files there.

## Leftovers in the Godot project

- `scenes/items/data/sword.tres`, `boomerang.tres`, `shield.tres` were created by Claude. Keep them or delete them and make my own.
- The roadmap's progress notes were updated by Claude to mark step 4 done.
