# Design: Guia — Development Standards Portal

**Date:** 2026-05-31  
**Status:** Approved  
**Replaces:** `Guia Basica/` (deleted)

---

## Overview

Replace the existing `Guia Basica/` folder with a new `Guia/` folder. The new portal is a sidebar-driven single-page app where a left accordion sidebar lets the user select which standard to view; the selected HTML renders in a full-height iframe on the right. No top tab bar.

---

## Folder Structure

```
Guia/
  index.html                  ← Main shell: sidebar + iframe
  general/
    planning.html
    git-workflow.html
    documentation.html
  python/
    code-standards.html
    functions.html
    api-client.html
    automation.html
    error-handling.html
  templates/
    claude-md.html            ← Renders both .md files as plain text + copy buttons
    claude-md-en.md           ← English CLAUDE.md template (editable source)
    claude-md-es.md           ← Spanish CLAUDE.md template (editable source)
```

**10 HTML files total** (1 index + 8 standards + 1 template viewer).  
**2 .md source files** for the CLAUDE.md templates.

---

## index.html

### Layout
- Full-height, no scroll on body
- Fixed top bar (title: "Guia — Development Standards")
- Left sidebar: fixed 220px wide, full-height, scrollable if needed
- Right content area: `<iframe>` fills all remaining space (no max-width)

### Sidebar — Accordion
Three groups, each collapsible independently:

| Group | Items |
|---|---|
| ⚙️ General | 📋 Planning · 🔀 Git Workflow · 📚 Documentation |
| 🐍 Python | 🎨 Code Standards · ⚙️ Functions · 🌐 API Client · 🤖 Automation · 🛡️ Error Handling |
| 📄 Templates | 📄 Copiar CLAUDE.md |

- Clicking a group header toggles its items open/closed (arrow rotates)
- Clicking an item sets it as active (highlight + blue left border) and sets `iframe.src`
- Default on load: all groups open, first item (Planning) active

### iframe
- `width: 100%; height: 100%; border: none`
- Loads the corresponding HTML file on nav item click

---

## Each Standard HTML (`general/*.html`, `python/*.html`)

Each file is a standalone dark-theme page (self-contained CSS, no external dependencies) that renders correctly both standalone and inside the iframe.

### Sections (in order)

1. **Header** — icon + title + one-line tagline
2. **¿Qué es?** — What this standard is; key concepts defined
3. **¿Para qué sirve?** — When to use it, what problems it solves
4. **¿Por qué este enfoque?** — Justification for the decisions made; why this over alternatives
5. **Con qué** — Tools, libraries, commands involved (shown as tags/code blocks)
6. **Cómo aplicarlo** — The actual rules and conventions from the `.md`, presented as readable prose + code examples where needed
7. **Referencia rápida** — Checklist or summary table for fast lookup

### Style
- Dark theme: `#0f172a` background, `#1e293b` cards, `#60a5fa` / `#93c5fd` headings
- Cards per section, consistent with existing Guia Basica visual style
- Monospace blocks for code/commands
- No max-width — content uses full available width

---

## templates/claude-md.html

### Purpose
Renders the content of `claude-md-en.md` and `claude-md-es.md` as plain text so edits to the `.md` files are immediately reflected without touching any HTML.

### Behavior
- On load: `fetch('./claude-md-en.md')` and `fetch('./claude-md-es.md')` (same folder)
- Displays fetched content in a `<pre>` block (monospace, dark bg, full width)
- Two buttons at the top: **Copiar EN** and **Copiar ES**
  - Each copies its respective `.md` content to clipboard
  - Button shows "✓ Listo" for 2 seconds after copy
- Default display: EN version shown; ES available via copy button
- If fetch fails (e.g., opened as file:// without a server), shows an error message

### Source .md files
- `claude-md-en.md` — English CLAUDE.md template (extracted from current `02_CLAUDE_md.html` `claudeMdEN` variable)
- `claude-md-es.md` — Spanish CLAUDE.md template (extracted from current `02_CLAUDE_md.html` `claudeMdES` variable)

---

## Migration

1. Create `Guia/` with the folder structure above
2. Write all HTML and .md files
3. Delete `Guia Basica/` folder entirely

---

## Pending: Language Cleanup (future HTML files)

> **Nota pendiente:** Los HTML actuales (y todos los que se creen en el futuro) deben evitar referencias a Claude o a AI como destinatario. La Guia es una referencia personal de convenciones de desarrollo — aplica siempre que el usuario trabaje, no específicamente al usar Claude. Al crear cada nuevo HTML, revisar que:
> - Los textos digan "cómo trabajar" o "convenciones personales", no "cómo Claude trabaja"
> - Las secciones "¿Para qué sirve?" y "¿Por qué este enfoque?" estén escritas en segunda persona o impersonal, no como instrucciones a una IA
> - Los HTML existentes también se actualizarán gradualmente conforme se revisen

---

## Out of Scope

- `python/data-analysis.html` — excluded until `data-analysis.md` is complete
- Any JavaScript / Python standards — not yet written
- Search functionality — not needed yet
- Mobile responsiveness — not a priority for a local reference tool
