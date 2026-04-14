You are about to help me set up a personal AI assistant that runs 24/7 as a Claude Code daemon, connected to Telegram. This assistant should feel like my own Jarvis -- sharp, proactive, always available, and deeply personalized to me.

## What you will build

A Claude Code session running in tmux that:
- Listens for Telegram messages via the Telegram channel plugin
- Has a persistent identity, personality, and memory
- Can transcribe voice messages (via OpenAI Whisper)
- Auto-saves context so it survives restarts and compacts
- Has a crash-recovery watchdog
- Can be controlled from the terminal with simple commands

## Before you start: OpenClaw conflict check

Before running the wizard, check if OpenClaw is installed by running `which openclaw`. If it is installed, warn the user:

OpenClaw uses the same Telegram bot polling mechanism. Having both running (or even installed) will cause silent message loss -- Telegram dispatches each update to only one poller at random, so messages randomly go to the wrong process.

Ask the user (via AskUserQuestion):
- "OpenClaw is installed on this machine. It conflicts with this assistant's Telegram integration. What would you like to do?"
  - **Uninstall now** (recommended): runs `npm uninstall -g openclaw`, removes the shell completion line from ~/.zshrc, and optionally removes ~/.openclaw
  - **Skip, I will uninstall later**: continue with setup, but warn that Telegram messages WILL be unreliable until OpenClaw is fully removed
  - **I don't use Telegram with OpenClaw**: continue, but still recommend uninstalling to avoid accidental conflicts

If the user chooses to uninstall, run:
1. Check where the binary lives with `ls -la $(which openclaw)` -- it may be installed under nvm's node (`~/.nvm/versions/node/...`) OR under Homebrew's node (`/opt/homebrew/lib/node_modules/`). Use the matching npm to uninstall:
   - If under nvm: `npm uninstall -g openclaw`
   - If under Homebrew: `/opt/homebrew/bin/npm uninstall -g openclaw`
2. Verify removal with `which openclaw` -- if it still exists, check for other installations
3. Remove the `source ".../.openclaw/completions/openclaw.zsh"` line from ~/.zshrc (if present)
4. Remove `~/.openclaw` directory (config + data)
5. Kill any running openclaw processes: `pkill -f openclaw`

If OpenClaw is not installed, skip this step silently.

## How this works

You will walk me through a wizard using AskUserQuestion. Ask one question at a time (or small groups of related questions). After gathering all my answers, you will generate every file needed and guide me through the final setup steps.

## Wizard rules

- Use AskUserQuestion for EVERY question. Never assume answers.
- Allow "skip" as a valid answer for any non-critical question. Use sensible defaults when skipped.
- Allow "back" to revisit the previous question.
- Group related questions together (max 3 per step).
- Show progress like "[Step 3/8]" so I know where I am.
- After the wizard, show a summary of all my answers and ask for confirmation before generating anything.
- If I say "change X", let me modify that specific answer without restarting the wizard.

## Wizard flow

### Step 1: Basics
- What should your assistant be called? (e.g., "Atlas", "Friday", "Nyx" -- or skip for a generic name)
- What vibe/personality should it have? Give me 3-4 options to pick from, plus "custom":
  - Sharp and casual (no fluff, opinionated, gets things done)
  - Warm and supportive (encouraging, patient, thorough)
  - Professional and precise (formal, structured, efficient)
  - Playful and witty (humor, creativity, personality)
  - Custom: describe your own
- What language(s) do you communicate in? (e.g., "English only", "English and Spanish -- switch based on input")

### Step 2: About you
- Your name (for personalization and memory)
- Your timezone (e.g., "US/Eastern", "Europe/Berlin")
- Your role/occupation (helps tailor responses -- e.g., "software engineer", "product manager", "student")
- Anything else the assistant should know about you from day one? (skip if nothing comes to mind)

### Step 3: Safety rules
Explain these defaults and ask which to keep, modify, or remove:
- Email: NEVER send directly, drafts only, always ask before creating
- Calendar: ask before creating/editing/deleting events
- Social media: always ask, no exceptions
- Any folders/paths that should be READ ONLY? (accounting, shared drives, etc.)
- Any other hard rules? (skip if defaults are fine)

### Step 4: Communication style
- How should the assistant communicate on Telegram? Pick or customize:
  - Message length: short and punchy vs detailed and thorough
  - Emoji usage: often / sometimes / never
  - Should it react to your messages with emoji? (e.g., thumbs up for acknowledgments instead of text replies)
  - Should it use lowercase casually or proper capitalization?
  - Should it always confirm before acting, or just do it for simple tasks?
  - Anything it should NEVER say or do? (e.g., "don't summarize what you just did", "don't use em dashes")

### Step 5: Telegram setup
- Guide me through creating a Telegram bot via @BotFather if I don't have one yet
- Ask for the bot token
- Walk me through the Telegram channel plugin setup:
  1. Run: `claude /telegram:configure` and paste the token
  2. Run: `claude /telegram:access` to approve the pairing
  3. Explain how pairing works (send any message to the bot, then approve in terminal)

### Step 6: Modules (all optional, recommend all)
Present each module with a short description and ask yes/no:

1. **Voice transcription** (recommended)
   - Transcribes voice messages using OpenAI Whisper (gpt-4o-transcribe)
   - Requires: OpenAI API key
   - Ask for the key if they want this module

2. **Web search** (recommended)
   - Search the web and summarize results from Telegram
   - Built into Claude Code, no extra setup needed

3. **Context persistence** (strongly recommended)
   - Auto-saves conversation context every 2 hours
   - Survives compacts and restarts
   - No extra setup needed, just creates the skill and cron

4. **Proactive check-ins** (optional)
   - Heartbeat every 4 hours -- checks for pending reminders, upcoming events, anything to flag
   - Ask what interval they want (default: 4 hours)

### Step 7: Telegram commands
Based on selected modules, show what commands will be registered:
- Always: /stop, /help, /status, /restart
- If memory: /remember, /forget
- If search: /search
- Ask if they want to add any custom commands

### Step 8: Final review
Show complete summary:
- Assistant name and personality
- Language(s)
- User profile
- Safety rules
- Communication style
- Modules enabled
- Telegram commands
- File paths that will be created

Ask: "Look good? Say 'go' to build everything, or tell me what to change."

## File generation

After confirmation, generate ALL files. Here is the complete architecture:

### 1. Working directory
Create `~/repos/{assistant-name-lowercase}/` (or let user pick)

### 2. CLAUDE.md (the soul)
Generate `{workdir}/CLAUDE.md` with:
- Identity section (name, vibe, personality description)
- Soul section (core principles -- be helpful not performative, have opinions, be resourceful, earn trust)
- Language rules
- Hard rules (from Step 3)
- Communication style (from Step 4)
- Proactive assistance rules (process files immediately: voice=transcribe, photo=analyze, PDF=summarize)
- Telegram commands list (from Step 7)
- Voice message handling (if enabled): transcribe first, respond second, never mix
- Progress updates: use reactions as indicators, send new message when done (for push notification)
- Token efficiency: ~230K per 5h rolling window, never dump massive datasets
- Context persistence rules (if enabled): save every 2h, after compacts, read daily file on startup
- Session startup checklist: recreate crons that expired, read today's context
- Contributing back section (see below)

#### Contributing back to the spawn prompt

The CLAUDE.md must include this section:

```
## Contributing Back

This assistant was spawned from the openclaw-spawn-prompt repository:
https://github.com/{user-github-handle}/openclaw-spawn-prompt

When you discover a fix, workaround, or improvement that would benefit future spawns:
- After confirming the fix works, ask the user: "This fix could help others spawning their own assistant. Want me to open a PR to the spawn prompt repo?"
- If yes: clone the repo, update setup-prompt.md with the new lesson or fix, and open a PR with a clear title and description
- PR should target the "Important implementation notes" section for operational lessons, or the relevant wizard step / file generation section for structural changes
- Keep PRs atomic -- one fix per PR
- Never PR personal preferences or user-specific config -- only universal improvements
```

### 3. start.sh (daemon launcher)
```bash
#!/bin/zsh
SESSION="{assistant-name-lowercase}"
WORKDIR="$HOME/repos/{assistant-name-lowercase}"

tmux kill-session -t "$SESSION" 2>/dev/null
pkill -f "external_plugins/telegram.*start" 2>/dev/null
sleep 1
tmux new-session -d -s "$SESSION" -c "$WORKDIR"
tmux send-keys -t "$SESSION" \
  "while true; do CLAUDE_CONFIG_DIR=$HOME/.claude command claude --dangerously-skip-permissions --continue --effort medium --channels plugin:telegram@claude-plugins-official; echo \"\$(date): Claude exited, restarting in 5s...\" >> $WORKDIR/daemon.log; sleep 5; done" Enter

echo "$(date): Started session $SESSION" >> "$WORKDIR/daemon.log"
```

Key features:
- Kills stale Telegram plugin processes before start (prevents duplicate polling)
- Auto-restart loop (when Claude exits from usage limits or crashes, restarts in 5s)
- Uses --continue to resume previous conversation (preserves context)
- Uses --effort medium to save tokens (recommend medium for always-on, high burns fast)
- CLAUDE_CONFIG_DIR ensures correct auth profile

### 4. watchdog.sh (crash recovery)
```bash
#!/bin/bash
SESSION="{assistant-name-lowercase}"
WORKDIR="$HOME/repos/{assistant-name-lowercase}"
LOG="$WORKDIR/watchdog.log"

# 1. If tmux session doesn't exist at all, restart
if ! tmux has-session -t "$SESSION" 2>/dev/null; then
  echo "$(date): Session not found, restarting..." >> "$LOG"
  bash "$WORKDIR/start.sh"
  exit 0
fi

# 2. Session exists but Claude might not be running inside it
# (e.g., double Ctrl-C killed Claude AND the restart loop, leaving an idle shell)
PANE_CMD=$(tmux display-message -t "$SESSION" -p '#{pane_current_command}' 2>/dev/null)

if [[ "$PANE_CMD" == "zsh" || "$PANE_CMD" == "bash" ]]; then
  echo "$(date): Session exists but Claude not running (pane_cmd=$PANE_CMD), restarting..." >> "$LOG"
  pkill -f "plugins/cache/claude-plugins-official/telegram" 2>/dev/null
  sleep 1
  tmux kill-session -t "$SESSION" 2>/dev/null
  sleep 1
  bash "$WORKDIR/start.sh"
  exit 0
fi

# 3. Kill orphaned telegram bun processes not belonging to the active session
PANE_PID=$(tmux display-message -t "$SESSION" -p '#{pane_pid}' 2>/dev/null)
if [[ -n "$PANE_PID" ]]; then
  TELEGRAM_PIDS=$(pgrep -f "plugins/cache/claude-plugins-official/telegram" 2>/dev/null)
  for PID in $TELEGRAM_PIDS; do
    PARENT=$PID
    IS_CHILD=false
    for i in $(seq 1 10); do
      PARENT=$(ps -o ppid= -p "$PARENT" 2>/dev/null | tr -d ' ')
      if [[ -z "$PARENT" || "$PARENT" == "1" ]]; then break; fi
      if [[ "$PARENT" == "$PANE_PID" ]]; then IS_CHILD=true; break; fi
    done
    if [[ "$IS_CHILD" == "false" ]]; then
      echo "$(date): Killing orphaned telegram process PID=$PID" >> "$LOG"
      kill "$PID" 2>/dev/null
    fi
  done
fi
```

### 5. LaunchAgent plist (macOS auto-recovery)
Generate `~/Library/LaunchAgents/com.{assistant-name-lowercase}.watchdog.plist` that runs watchdog.sh every 60 seconds. Include proper PATH environment variable.

### 6. Shell function
Add to ~/.zshrc (or ~/.bashrc):
```bash
{assistant-name-lowercase}() {
  case "${1:-start}" in
    start)
      if tmux has-session -t {assistant-name-lowercase} 2>/dev/null; then
        echo "Already running."
      else
        zsh "$HOME/repos/{assistant-name-lowercase}/start.sh"
        echo "Started."
      fi
      ;;
    stop)
      tmux kill-session -t {assistant-name-lowercase} 2>/dev/null && echo "Stopped." || echo "Not running."
      ;;
    restart)
      tmux kill-session -t {assistant-name-lowercase} 2>/dev/null
      sleep 1
      zsh "$HOME/repos/{assistant-name-lowercase}/start.sh"
      echo "Restarted."
      ;;
    status)
      tmux has-session -t {assistant-name-lowercase} 2>/dev/null && echo "Running." || echo "Not running."
      ;;
    attach)
      tmux attach -t {assistant-name-lowercase} 2>/dev/null || echo "Not running."
      ;;
    *)
      echo "Usage: {assistant-name-lowercase} [start|stop|restart|status|attach]"
      ;;
  esac
}
```

### 7. Maker shell function
Add a second shell function to ~/.zshrc (or ~/.bashrc) that resumes the Claude Code session used to spawn and manage the assistant. This is the "control room" -- for editing the assistant's CLAUDE.md, skills, memory, start.sh, watchdog, etc.

```bash
{assistant-name-lowercase}-maker() {
  claude --dangerously-skip-permissions --resume {SESSION_ID}
}
```

Replace `{SESSION_ID}` with the actual session ID from the current Claude Code conversation (the one running the wizard). Use whatever `claude` alias/function the user already has for the correct profile (e.g., if they use `claudet` for their work profile, use that instead of bare `claude`).

Tell the user: "Run `source ~/.zshrc`, then use `{assistant-name-lowercase}-maker` anytime you want to tweak the assistant's config, skills, or infrastructure."

### 8. Skills (based on module selection)

#### Voice transcription skill (.claude/skills/voice-transcribe/SKILL.md)
Whisper pipeline: react with eye emoji, download attachment, ffmpeg convert OGA to MP3, call gpt-4o-transcribe API, reply with blockquote transcription, follow up if it was a question.

#### Save context skill (.claude/skills/save-context/SKILL.md)
Save timestamped context summary to daily memory file. Only meaningful stuff: decisions, learnings, config changes, preferences. Skip routine activity.

### 9. Memory structure
Create the memory directory and seed files:
- `memory/MEMORY.md` (index)
- `memory/{user-name-lowercase}_personal.md` (user profile from Step 2)
- `memory/interaction_preferences.md` (communication preferences from Step 4)

### 10. Permission configuration
Add to `~/.claude/settings.json` (merge, don't overwrite):
- Allow: Telegram MCP tools (reply, react, edit_message, download_attachment)
- Allow: Bash for specific tools only (curl, ffmpeg, date)
- Deny: Write/Edit to any read-only paths from Step 3

### 11. tmux configuration
If `~/.tmux.conf` does not exist, create it. If it exists, append only missing settings. Ensure it includes:
```
set -g mouse on
```
This enables mouse scrolling, text selection, and clicking inside the tmux session -- essential for navigating the assistant's terminal output.

### 12. Register Telegram bot commands
Use curl to call Telegram Bot API setMyCommands with the selected command list.

## Post-setup

After generating everything:
1. Tell the user to run: `source ~/.zshrc` (to load the shell function)
2. Tell them to start with: `{name} start`
3. Tell them to send a test message from Telegram
4. If they enabled the watchdog: give them the launchctl bootstrap command
5. Remind them that crons auto-expire after 7 days -- the CLAUDE.md startup section handles re-creation automatically

## Important implementation notes (from battle-testing)

These are hard-won lessons. Follow them exactly:

1. **Stale Telegram plugin processes**: When Claude Code exits, the Telegram plugin process (bun) may keep running. Multiple instances fight over bot polling, causing messages to silently disappear. ALWAYS pkill before starting.

2. **--continue vs fresh start**: --continue resumes the last conversation in the working directory. This preserves context but also preserves effort level. The --effort flag only applies to new sessions. If you need to change effort after resume, type /effort directly in the tmux session.

3. **Auth profile conflicts**: If the user has multiple Claude accounts (e.g., personal + team org), tmux may pick up the wrong one. CLAUDE_CONFIG_DIR forces the correct profile. Team/org accounts may block --dangerously-skip-permissions.

4. **Telegram reactions**: Only 73 specific emoji work with the Bot API. Common safe picks: thumbs up, fire, heart, ok hand, grin, rofl, party, star eyes, eyes, salute, 100, trophy. Notably, the laughing-crying emoji does NOT work -- use rofl instead.

5. **Reply first, work second**: When a Telegram message arrives, the very first tool call should be the reply (or at minimum a reaction). Never do background work (saving context, setting up crons) before acknowledging the user. They can't see your terminal.

6. **Post-compact context loss**: After auto-compact, context is compressed. The save-context skill captures important state before it's lost. The CLAUDE.md instruction to run /save-context after every compact is critical.

7. **Message delivery during processing**: Telegram messages sent while Claude is mid-processing a previous message may be silently dropped by the plugin. This is a known limitation.

8. **Diacritics and special characters**: If the user's language uses diacritics or special characters, explicitly instruct the assistant to always use them. Models sometimes drop them unless told not to.

9. **Token budget**: A Claude Pro subscription gives roughly 230K tokens per 5-hour rolling window. Medium effort is recommended for always-on assistants to stretch this budget. High effort burns through it fast.

10. **launchd watchdog**: The plist needs proper PATH to find tmux and claude binaries. Copy PATH from an existing working plist or set it explicitly.

11. **Zombie plugin processes accumulate silently**: Stale Telegram bun processes don't just come from restarts -- they also spawn when you run claude with the telegram channel from other terminals or profiles. Over time you can end up with 5-6 bun processes all polling the same bot. Telegram dispatches each update to only one poller at random, so most messages get swallowed by zombie processes. The pkill in start.sh only helps on restart. For ongoing health, the watchdog should also kill orphaned telegram plugin processes that aren't children of the active tmux session.

12. **Auth expiry is invisible to the user**: When the Anthropic API token expires or becomes invalid, Claude can't process messages AND can't notify via Telegram -- the error happens at the API level before Claude gets to execute any code. Messages just silently go unanswered. Solution: the restart loop in start.sh should capture the last N lines from the tmux pane after Claude exits, grep for error patterns (authentication, rate_limit, expired, crash), and send a notification directly via curl to the Telegram Bot API. This is the only way to surface fatal errors. See the start.sh error notification pattern.

13. **Login is per config dir, not global**: If running multiple profiles (e.g., ~/.claude for personal, ~/.claude-work for work), logging in on one does NOT fix the other. The daemon's CLAUDE_CONFIG_DIR determines which auth token is used. If the personal token expires, you must login specifically with CLAUDE_CONFIG_DIR pointing to ~/.claude -- logging in from a different session that uses a different config dir won't help.

14. **/logout breaks --continue**: After /logout + auto-restart, --continue can't resume because the session is invalidated. Claude starts completely fresh (theme picker, onboarding). This requires manual intervention (attach to tmux, complete login). Avoid /logout for testing error notifications -- instead, temporarily rename/move the auth credentials file, which produces an auth error without destroying the session.

15. **Restart loop error notification pattern**: Add this to the while loop in start.sh -- after Claude exits, capture tmux pane, grep for errors, send via Bot API:
    ```
    EXIT_CODE=$?
    LAST_OUTPUT=$(tmux capture-pane -t $SESSION -p -S -10 | tail -10)
    if echo "$LAST_OUTPUT" | grep -qiE 'error|authentication|rate.limit|expired|crash'; then
      curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" \
        -d chat_id="$CHAT_ID" -d parse_mode="Markdown" \
        --data-urlencode "text=$(printf 'warning {assistant-name} crashed (exit %s). Restarting...\n\n```\n%s\n```' "$EXIT_CODE" "$LAST_OUTPUT")" > /dev/null
    fi
    ```
    Store BOT_TOKEN and CHAT_ID at the top of start.sh, read from the channel .env file.

16. **Watchdog must check process, not just session**: The basic `tmux has-session` check is insufficient. When the user double Ctrl-C's (or the restart loop exits for any reason), the tmux session stays alive with an idle shell prompt, but Claude is not running. The watchdog sees the session and thinks everything is fine. Fix: check `#{pane_current_command}` -- if it's "zsh" or "bash", Claude is dead and needs a full restart (kill session + start.sh). This is already implemented in the watchdog.sh template above.
