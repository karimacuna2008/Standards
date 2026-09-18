# Quick Interfaces Standard (Streamlit & CustomTkinter)

> ⚠️ **STATUS: DRAFT PROPOSED BY GEMINI - PENDING VALIDATION WITH CLAUDE**
> <!-- VALIDAR CON CLAUDE: Review rapid UI separation patterns and external API contract before final consolidation -->

Operational instructions for developing and refactoring rapid desktop/web interfaces (Streamlit, CustomTkinter, CLI GUIs).

---

## 1. Golden Rule: Pure Frontend & API Decoupling
- **Strict Decoupling:** Rapid UI interfaces MUST act purely as a presentation and interaction layer (frontend).
- **No Direct Database or Heavy Business Logic:** Interfaces must NOT contain SQL queries, direct ORM database sessions, or long-running CPU algorithms.
- **External API Consumer:** Wherever an API exists (or can be decoupled), the interface MUST consume the backend via an HTTP API client (referencing `python/api-client.md`) or decoupled service modules.
- **Local Utilities Exception:** Internal scripts without network access may call decoupled `services/` modules, but never mix data processing directly into UI callbacks or widgets.

---

## 2. Streamlit Applications

### 2.1 File & Layer Layout
```
project/
├── app.py                      # Main entrypoint, page routing, and layout
├── views/                      # Optional multi-page views (if large)
├── config.py                   # Centralized st.secrets and environment access
├── services/                   # Business logic and API client calls (testable without Streamlit)
├── .streamlit/
│   ├── config.toml             # Theme and server settings
│   ├── secrets.toml.example    # Required committed template with dummy keys
│   └── secrets.toml            # Quarantined in .gitignore (NEVER committed)
└── requirements.txt
```

### 2.2 Execution Control & Modern Navigation (2026)
- **`st.set_page_config()`:** MUST be the first Streamlit command executed in `app.py`.
- **Navigation:** Prefer `st.navigation` and `st.Page` over the legacy `pages/` directory for dynamic menu control, access control, and state awareness.
- **Partial Reruns (`@st.fragment`):** Wrap interactive widgets, charts, or isolated control panels in `@st.fragment` to avoid re-executing the entire page script on every click.
- **Forms (`with st.form("form_key"):`):** Mandatory for multi-input workflows (filters, parameter configuration, creation forms) to batch updates upon submit.

### 2.3 Caching Policy
- **`@st.cache_data`:** Use for tabular data loading, HTTP GET requests, and immutable transformations. Set `ttl` appropriately (e.g. `ttl=300`).
- **`@st.cache_resource`:** Use for global singleton objects (HTTP clients, thread pools). Never mutate cached resource objects.

### 2.4 State Management
- Initialize state variables at boot using guard clauses:
  ```python
  if "authenticated" not in st.session_state:
      st.session_state.authenticated = False
  ```

### 2.5 Inyección Segura de CSS/HTML (`st.markdown`)
- **Regla Anti-Corrupción del Resaltado de Sintaxis (VS Code / Cursor):**
  - **NUNCA** utilices f-strings multilínea con triples comillas (`f"""...<style>..."""`) que mezclen etiquetas `<style>`, dobles llaves (`{{`, `}}`) y colores hexadecimales (`#hex`).
  - **Razón técnica:** Los analizadores de gramática (TextMate) en VS Code y Cursor conmutan a modo CSS al detectar `<style>`. Las llaves dobles de f-string y las almohadillas `#` desincronizan el analizador, provocando que no reconozca el cierre de las triples comillas `"""` y tiña **todo el resto del archivo Python** como si fuera un comentario o cadena sin cerrar.
  - **Patrón Obligatorio:** Construye las reglas CSS como cadenas simples concatenadas (o tuplas de strings) y aplica `<style>` en una sola línea cerrada:
    ```python
    # ❌ INCORRECTO: Rompe el syntax highlighting del editor para el resto del archivo
    st.markdown(
        f"""
        <style>
        .st-key-{key} button {{
            background-color: {bg_color};
            border: 2px solid {border_color};
            color: #ffffff;
        }}
        </style>
        """,
        unsafe_allow_html=True,
    )

    # ✅ CORRECTO: Inmune a desincronización del parser en el editor
    css = (
        f".st-key-{key} button {{"
        f" background-color: {bg_color};"
        f" border: 2px solid {border_color};"
        f" color: #ffffff;"
        f" width: 100%;"
        f" }}"
    )
    st.markdown(f"<style>{css}</style>", unsafe_allow_html=True)
    ```

---


## 3. CustomTkinter / Desktop GUIs

### 3.1 Architecture & Separation
- The GUI class handles window geometry, widget bindings, and visual status updates only.
- Complex operations and API requests MUST run in background threads (`threading.Thread` or `concurrent.futures`) to prevent freezing the desktop UI event loop.
- Use thread-safe queues or root `after()` callbacks to update UI elements from background threads.

### 3.2 Testing Policy (Manual Checklists)
- As defined in `python/testing.md` §6, functions opening Tkinter windows/dialogs are excluded from automated pytest coverage.
- Deliver a clear manual testing checklist (Steps + Expected Outcomes) whenever updating desktop GUI flows.

---

## 4. Secrets & Configuration
- Never hardcode URLs, API keys, or credentials in UI files.
- Streamlit secrets come from `st.secrets` (via `config.py`).
- Desktop GUI configs come from `.env` or user config files (via `python/configuration.md`).

---
**Version:** 1.0 (Draft)  
**Last Updated:** 2026-09-17
