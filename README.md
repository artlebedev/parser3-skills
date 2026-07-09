# parser3-skills

ИИ-навыки для работы с [Parser3](https://www.parser.ru/)

## Установка

### Способ 1. Попросить агента установить

```text
Install the parser3-skills skill for me:

1. Add as git submodule https://github.com/artlebedev/parser3-skills.git into my
   user-level skills directory as `parser3-skills/`.
   Use the skill directory globally my agent reads on this machine, for example:
   - Codex: ~/.codex/skills/parser3-skills/
   - Claude Code: ~/.claude/skills/parser3-skills/
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

## Обновление

### Способ 1. Попросить агента обновить

```text
Update the parser3-skills skill to the latest version:

1. Go to the parser3-skills submodule directory:
   - Codex: ~/.codex/skills/parser3-skills/
   - Claude Code: ~/.claude/skills/parser3-skills/
2. Pull the latest changes: git pull origin main
3. Confirm the current commit hash when done.
```

### Способ 2. Вручную

```bash
# Codex
git -C "${CODEX_HOME:-$HOME/.codex}/skills/parser3-skills" pull origin main

# Claude Code
git -C "$HOME/.claude/skills/parser3-skills" pull origin main
```

Или из директории, где установлен submodule (например `~/.claude`), с обновлением всех submodule сразу:

```bash
git submodule update --remote skills/parser3-skills
```

## Использование

После установки скилл активируется автоматически — достаточно упомянуть в запросе Parser3 или открыть `.p` файл. 
```text
Проверь этот .p файл на ошибки Parser3
```
Можно также вызвать явно `/parser3-skills` в Claude Code.
```text
/parser3-skills <запрос>
```
