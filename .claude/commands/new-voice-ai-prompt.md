# New Voice AI Prompt

**This command is a pointer. All rules live in the skill.**

Invoke the `new-voice-ai-prompt` skill and follow it exactly:

- Canonical source in the repo: `skills/core/new-voice-ai-prompt/SKILL.md`
  (EverythingAI-Pro/metatech-skills)
- After `install-all.sh`: `~/.claude/skills/new-voice-ai-prompt/SKILL.md`
- On a repo-symlink setup (whole repo linked into `~/.claude/skills/`):
  `~/.claude/skills/metatech-skills/skills/core/new-voice-ai-prompt/SKILL.md`

Do not add rules to this file. Two copies of the same skill with different rules is
how the team ended up running different versions of the generator for months. If a
rule needs to change, change it in `SKILL.md`.

The skill asks 13 questions, generates the 12-section prompt, pressure-tests it
against 14 scripted callers, and requires your explicit sign-off before delivering.
