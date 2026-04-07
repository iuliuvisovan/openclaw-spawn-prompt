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
- sends daily email and calendar digests
- generates weekly timesheets from git history
- saves its own context so it survives restarts
- recovers from crashes automatically
- learns your preferences over time

you don't write code. you answer a wizard. Claude Code builds everything.

## what you need

- [Claude Code](https://claude.ai/claude-code) installed
- a Telegram bot token (the wizard walks you through creating one)
- optionally: OpenAI API key (for voice transcription), Gmail/Google Calendar MCP (for digests)

## how to use

1. open Claude Code in your terminal
2. paste the contents of [`PROMPT-OPENCLAW-INTO-EXISTENCE.md`](./PROMPT-OPENCLAW-INTO-EXISTENCE.md)
3. answer the wizard
4. done. your assistant is alive.

## what happens after

the assistant runs in tmux, restarts on crash, and saves context every 2 hours. you talk to it on Telegram like a normal chat.

when it discovers a fix or improvement that would help future spawns, it will ask you: "want me to open a PR to the spawn prompt repo?" -- so the prompt keeps getting better over time, contributed by every instance running in the wild.

## the meta

somebody made Claude. somebody made Claude Code. this prompt lets you spawn your own personal AI from Claude Code and keep building it however you want.

one level of abstraction above. pretty fun.

## contributing

the best contributions come from running assistants. when your bot finds a fix, let it PR. that's the whole point.

you can also open issues or PRs manually if you find something.
