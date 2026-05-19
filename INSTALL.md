# Install : Tailwind CSS Skill Package

## Claude Code (local)

```bash
git clone https://github.com/OpenAEC-Foundation/TailwindCSS-Claude-Skill-Package.git
cp -r TailwindCSS-Claude-Skill-Package/skills/source/ ~/.claude/skills/tailwind/
```

Restart Claude Code if running. Skills auto-discover from `~/.claude/skills/`.

## As git submodule (recommended for project-specific use)

```bash
cd your-project
git submodule add https://github.com/OpenAEC-Foundation/TailwindCSS-Claude-Skill-Package.git .claude/skills/tailwind
git commit -m "feat: add tailwind skills as submodule"
```

## Via npm-agentskills

```bash
npx skills add @openaec/tailwindcss-claude-skill-package
```

## Claude.ai (web)

1. Open a Claude.ai project
2. Upload individual `SKILL.md` files from `skills/source/**/` as project knowledge
3. Skills activate based on description triggers

## Verification

After install, ask Claude :

> {{VERIFICATION_PROMPT}}

If skill activates : install successful. If not : check `~/.claude/skills/tailwind/` exists and contains category-folders.

## Requirements

- Tailwind CSS v3.4-v4
- Claude Code (latest)
- {{ADDITIONAL_REQUIREMENTS}}
