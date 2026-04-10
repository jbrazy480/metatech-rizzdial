# MetaTech AI — Master Prompt Library

Everything we've learned building RizzDial voice AI agents that actually close deals at scale (100k+ calls/day).

This repo is the **single source of truth** for how we write voice AI prompts at MetaTech AI. If you're building a new agent, read `MASTER_PROMPT_GUIDE.md` first, then grab the closest match from `templates/` and adapt it.

## Structure

```
.
├── MASTER_PROMPT_GUIDE.md     # THE rules — read this first, every time
├── templates/                 # Production-ready agent templates by use case
│   ├── 00-RIZZDIAL-SECTION-STRUCTURE.md   # The 12-section Retell builder format
│   ├── 01-speed-to-lead-live-transfer.md
│   ├── 02-speed-to-lead-booking.md
│   ├── 03-speed-to-lead-both.md
│   ├── 04-no-show-recovery.md
│   ├── 05-360-nurture.md
│   ├── 06-annual-reengagement.md
│   ├── 07-appointment-reminder-2hr.md
│   ├── 08-post-sale-welcome.md
│   ├── 09-appointment-no-show-v2.md
│   ├── 10-database-reactivation.md
│   ├── BUILD-A-BOT-DEV-SPEC.md
│   └── README.md
└── examples/
    └── marketing_strategist_kickoff.md   # Canonical reference — the gold standard
```

## How to use this with Claude Code

When you want Claude to build a new voice agent for you, say:

> "Read `MASTER_PROMPT_GUIDE.md` and `examples/marketing_strategist_kickoff.md`, then build me a [role] agent in the 12-section format."

Claude will follow the rules, structure, and voice every time.

## Golden rules (the short version)

1. **One question at a time.** Ask → stop → wait → react → next. Never stack.
2. **Never speak a variable name out loud.** If `{{client_name}}` is empty, skip the sentence.
3. **Complete every section. No skipping.** Especially creative/Section F.
4. **Dig on insights.** When a business-changing detail drops, break script and ask 2-3 follow-ups.
5. **Silence is your friend.** After a question, shut up and wait.
6. **Sound human.** Disfluencies, contractions, real reactions. Never "as an AI."
7. **12-section Retell format** for every prompt, every time.

Full detail in `MASTER_PROMPT_GUIDE.md`.
