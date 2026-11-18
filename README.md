# claude-skills

Mis skills para Claude y Claude Code. Cada carpeta es una skill con su `SKILL.md`.

| Skill | Para qué |
|---|---|
| `auditoria-seguridad` | Auditoría y hardening de seguridad por fases, con aprobación entre cada una |

## Instalar

**En todos los proyectos (global):**

```bash
git clone git@github.com:maneljvadillo/claude-skills.git ~/claude-skills
mkdir -p ~/.claude/skills
ln -s ~/claude-skills/auditoria-seguridad ~/.claude/skills/auditoria-seguridad
```

Con el enlace simbólico, un `git pull` en `~/claude-skills` actualiza la skill.

**Solo en un proyecto (se sube con el repo):**

```bash
mkdir -p .claude/skills
cp -r ~/claude-skills/auditoria-seguridad .claude/skills/
```

## Usar

En Claude Code: *"hazle una auditoría de seguridad a este proyecto, modo estándar"*.
Para comprobar que la ha cargado: *"¿qué skills tienes?"*.
