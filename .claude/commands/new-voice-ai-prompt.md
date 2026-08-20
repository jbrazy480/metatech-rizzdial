# New Voice AI Prompt

**This command is a pointer. All rules live in the skill.**

Invoke the `new-voice-ai-prompt` skill and follow it exactly:

- Canonical source: `~/.claude/skills/new-voice-ai-prompt/SKILL.md`
- In the repo: `skills/core/new-voice-ai-prompt/SKILL.md` (EverythingAI-Pro/metatech-skills)

Do not add rules to this file. Two copies of the same skill with different rules is
how the team ended up running different versions of the generator for months — Eric
on a 9-question command file, everyone else on a 7-question skill. If a rule needs to
change, change it in `SKILL.md`.

The skill asks 12 questions and outputs a complete 12-section prompt, pre-sliced for
the RizzDial builder UI.
