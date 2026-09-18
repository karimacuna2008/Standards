# SKILL.md — Submodule Template

Template for the `SKILL.md` file that lives at the root of each monorepo submodule (see `AGENTS.md` §7-8). Copy it as `SKILL.md` inside that subfolder and fill in its sections — don't duplicate rules already covered by the root `AGENTS.md` or by `Standards/` in there.

**When to use this template:** when the "Monorepo Map" (`AGENTS.md` §8) identifies a piece that doesn't have its `SKILL.md` yet, before the Executor works on it.

**Note on language:** the fenced `SKILL.md` template below is written in Spanish on purpose — it's the literal payload copied into a real project, matching the language Karim actually writes project files in. See `Standards/README.md` decisions log ("Excepción de idioma") for why this file is exempt from the English-only rule inside the fence.

## Template

```markdown
# SKILL.md — [Nombre del Módulo]

> **Directiva obligatoria:** si esta sesión inició directamente dentro de esta carpeta (sin haber leído primero el `AGENTS.md` de la raíz del proyecto), debes localizar y leer el `AGENTS.md` más cercano en un directorio ancestro antes de continuar. Este archivo **no reemplaza** esas reglas globales — solo las complementa con lo específico de este módulo.

## 1. Qué es este módulo
[Una o dos líneas: propósito de esta carpeta dentro del monorepo.]

## 2. Reglas locales de dominio
[Convenciones específicas de este módulo que no aplican al resto del monorepo: stack, patrones, restricciones. Si una regla ya está en un estándar de `Standards/`, referencia el archivo en vez de repetirla aquí.]
- ...

## 3. Comandos de test
Comando(s) que el Ejecutor debe correr **antes de marcar cualquier tarea de `tasks.md` como completada** en este módulo:
```bash
[comando de test]
```
Si el comando falla, el Ejecutor se detiene y genera `error_dump.txt` (ver `AGENTS.md` §5.B) — nunca marca una tarea `[x]` con tests en rojo.

## 4. Dependencias y contratos con otros módulos
[Qué expone este módulo a otras piezas del monorepo, y qué no se debe romper sin avisar al Arquitecto.]

---

**Última Actualización:** [fecha]
```

---

**Version:** 1.0
**Last Updated:** 2026-09-11
