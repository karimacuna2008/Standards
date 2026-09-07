# CLAUDE.md

---

## 📋 ESTÁNDARES

> PRIMERA SESIÓN: Verifica si la carpeta en `Local` existe.
> - Si existe: elimina este bloque y usa esa ruta directamente.
> - Si no existe: pregunta al usuario: "¿Descargo los estándares de GitHub y los guardo
>   localmente, o los leeré siempre desde GitHub?"
>   - Si elige local: clona el repo, guárdalo en el escritorio del usuario, actualiza
>     la ruta `Local` aquí y elimina este bloque.
>   - Si elige GitHub: elimina la línea `Local` y elimina este bloque. Conserva siempre
>     la línea GitHub.
>
> Una vez resuelta la ruta de Standards, DEBES indicarle al usuario que ejecute el
> siguiente comando y esperar su confirmación antes de continuar con cualquier otra tarea:
>   git update-index --skip-worktree CLAUDE.md
> No continúes hasta que el usuario confirme que fue ejecutado.

Local: C:\Users\Karim Acuna\OneDrive\Desktop\Programs\CLAUDE\Standards
GitHub: https://github.com/karimacuna2008/Code-Standards.git

### Cómo usar los Estándares:
- Si trabajas con **Python** → lee `python/INDEX.md` dentro de la carpeta Standards
- Si trabajas con **Planeación/Git/Documentación** → lee `general/INDEX.md` dentro de la carpeta Standards
- Cada `INDEX.md` contiene la lista de archivos disponibles y cuándo usar cada uno

---

## 🔒 1. PUERTA DE VALIDACIÓN OBLIGATORIA

Después de presentar el plan y las justificaciones, debes detenerte y pedir validación para cada cambio específico. **No estás autorizado a ejecutar, escribir o guardar ningún código hasta que el usuario confirme explícitamente cada paso.**

NUNCA modifiques nada sin aprobación explícita. Siempre pregunta: "¿Apruebas este cambio? Sí/No" antes de continuar.

El idioma depende del que se utilice en la sesión.

---

## 📝 2. GESTIÓN DE CAMBIOS Y JUSTIFICACIÓN

Tienes prohibido ejecutar cambios o escribir código final hasta que se complete el siguiente proceso de justificación:

### Regla "Plan Primero"
Proporciona primero un resumen estructurado de todos los cambios propuestos.

### Razonamiento por Dominio
- **Ciencia de Datos e IA/ML:** Explica la metodología. Justifica por qué se eligió este modelo/enfoque específico sobre las alternativas y por qué se adapta a este caso de uso específico.
- **Tecnología y Librerías:** Identifica todas las librerías/tecnologías a utilizar. Justifica su necesidad y cómo se integran en el stack existente.

### Desglose Técnico Profundo (Scripts y Funciones)
Para cada función o método (nativo o de librería), detalla:
- **Funcionalidad:** Lógica interna y propósito.
- **Parámetros:** Desglose de los argumentos y qué controlan.
- **Valores de Retorno:** Qué devuelve y su impacto en el flujo posterior.
- **Variables:** Desglose de las variables utilizadas y las que se crearán.

---

## 🔄 3. GESTIÓN DE CONTEXTO Y CONTINUIDAD

Cuando el contexto acumulado crezca, recomienda al usuario `/compact` o `/clear`
según corresponda:

**Cuándo recomendar `/compact`:**
- La tarea o fase actual sigue en curso (no terminó)
- El trabajo siguiente depende de detalles recientes (decisiones, código, debugging)
  que conviene preservar de forma resumida, no descartar
- Solo se necesita liberar espacio de contexto, sin cambio de alcance u objetivo

**Cuándo recomendar `/clear`** (y proporciona un comentario de inicio listo para
pegar en la siguiente sesión):
- La fase o tarea actual está completa y lo que sigue tiene un alcance u objetivo diferente
- El contexto acumulado contiene información ya no relevante para los próximos pasos
  (bugs resueltos, exploración previa, enfoques descartados)
- El usuario pregunta si conviene limpiar el contexto

**Al recomendar `/clear`, siempre incluye:**
Un comentario completo y específico que el usuario pueda pegar al inicio de la siguiente sesión:
- Qué se logró o decidió en la sesión actual
- El punto exacto donde se retomará el trabajo
- Qué hacer a continuación y en qué orden

**Formato del comentario de inicio:**
> Refiere a CLAUDE.md
> Contexto: [resumen breve de lo que se hizo / dónde nos quedamos]
> Siguiente: [qué hacer, comenzando desde [archivo / función / fase / paso]]

Debe ser conciso pero suficientemente completo para que la nueva sesión arranque sin re-explicaciones.

---

## 🔐 4. COMMITS DE GIT — REGLA ABSOLUTA

Esta regla nunca cambia, sin excepción:

- **Nunca** agregues una línea `Co-Authored-By: Claude` (ni ninguna variante) en los mensajes de commit.
- Los commits deben verse como **100% autoría humana del usuario**: usa siempre la identidad de Git ya configurada en el repo (`user.name` / `user.email`). Nunca uses `--author`, nunca tu propia identidad.
- Tú **puedes** ejecutar los comandos de Git (`git commit`, `git push`, etc.) a través de la terminal (Bash/cmd/PowerShell) — exactamente como si el usuario los hubiera escrito él mismo. No confirmes cambios por ninguna otra vía (API, integración, etc.).
- Antes de hacer un commit, pregunta siempre: **"¿Hago el commit yo o lo haces tú?"**. Si el usuario prefiere hacerlo él mismo, entrégale los comandos en vez de ejecutarlos.

---

## ✅ ACERCA DE ESTE PROYECTO

[Agrega contexto específico del proyecto aquí]

---

**Última Actualización:** 2026-06-08
**Versión de Estándares:** v1.0
