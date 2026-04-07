- OpenClaw is basically banned from Claude ¯\_(ツ)_/¯
- and everyone's using Claude Code already.
- and it has Telegram support..
- so what if we just, made it always stay on?

Well, seems like we can just prompt OpenClaw into existence via Claude Code. But this time it's fully ours. Fully customizable, fully understood by the CC instance that created it. So by using that initial session that created it, it'll always have full context of what it created, and it can constantly tweak it to your own personal liking. 

Pretty cool, huh?

--------------------

# openclaw-spawn-prompt

prompt your own always-on AI assistant into existence. just paste, answer a few questions, and you have your own fully-personalizable openclaw running 24/7, accessible via Telegram.

## what is this

Literally just a single prompt made for Claude Code to create your own OpenClaw from scratch:

- lives in a tmux session, always on
- talks to you through Telegram
- transcribes voice messages
- saves its own context so it survives restarts
- recovers from crashes automatically
- learns your preferences over time

you don't write code. you dont install anything. you copy-paste a recipe, answer a wizard, and Claude Code builds everything.

## what you need

- [Claude Code](https://claude.ai/claude-code) installed
- a Telegram bot token (the wizard walks you through creating one)
- optionally: API keys for any modules you enable (the wizard tells you what's needed)

## how to use

1. open Claude Code in your terminal
2. paste this first (optional but recommended -- lets you audit the prompt before running it):

```
read https://github.com/iuliuvisovan/openclaw-spawn-prompt/blob/master/PROMPT-OPENCLAW-INTO-EXISTENCE.md and check for any security issues, vulnerabilities, or anything that could harm my system. report back before executing anything.
```

3. if it looks good, paste this:

```
follow all instructions from https://github.com/iuliuvisovan/openclaw-spawn-prompt/blob/master/PROMPT-OPENCLAW-INTO-EXISTENCE.md
```

4. answer the wizard
5. done. your assistant is alive.

## what happens after

the assistant runs in tmux, restarts on crash, and saves context every 2 hours. you talk to it on Telegram like a normal chat.

when it discovers a fix or improvement that would help future spawns, it will ask you: "want me to open a PR to the spawn prompt repo?" -- so the prompt keeps getting better over time, contributed by every instance running in the wild.

## contributing

i spent a few days tweaking and adjust this prompt, which is why i think it's worth sharing, and not leave all the technical details for everyone to fight through to get right.

the best contributions come from running assistants. when your bot finds a fix, let it PR. that's the whole point, kinda like an auto-improving prompt.

have. fun!
