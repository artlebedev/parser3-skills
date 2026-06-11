# parser3-skills

ИИ-навыки для работы с Парсер3

## Установка

### Способ 1. Попросить агента установить

```text
Install the parser3-skills skill for me:

1. Add as git submodule https://github.com/artlebedev/parser3-skills.git into my
   user-level skills directory as `parser3-skills/`.
   Use the skill directory globally my agent reads on this machine, for example:
   - Codex: ~/.codex/skills/
   - Claude Code: ~/.claude/skills/
2. Verify that SKILL.md, AGENTS.md, and the references/ directory are present.
3. Confirm the install path when done.
```

### Способ 2. Вручную

```bash
# Codex
cd "${CODEX_HOME:-$HOME/.codex}"
mkdir -p skills
git submodule add https://github.com/artlebedev/parser3-skills.git \
  skills/parser3-skills

# Claude Code
cd "$HOME/.claude"
mkdir -p skills
git submodule add https://github.com/artlebedev/parser3-skills.git \
  skills/parser3-skills
```
