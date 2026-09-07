# API Security

Reglas de seguridad para APIs (FastAPI) que exponen endpoints internos y reciben webhooks externos.

---

## 1. Comparación de tokens — tiempo constante

Nunca compares un token o secreto con `==` / `!=`: la comparación corta en el primer byte distinto y filtra información por tiempo (timing attack). Usa `secrets.compare_digest` a través de un helper único:

```python
import secrets

def verify_token(provided: str, expected: str) -> bool:
    return bool(expected) and secrets.compare_digest(provided, expected)
```

- `bool(expected)` garantiza que un secreto **vacío / no configurado nunca valide** ninguna petición.
- Centraliza la verificación; no repitas la comparación inline en cada endpoint.

## 2. Endpoints internos

Cada endpoint sensible (admin, sweep, procesamiento interno) exige su token por header (`X-Admin-Token`, `X-Sweep-Token`, …), verificado con `verify_token`. El servicio Cloud Run puede ser público (`--allow-unauthenticated`) si lo llaman sistemas externos: la protección va **por token en cada endpoint**, no cerrando el servicio.

## 3. Webhooks entrantes

Un webhook público sin autenticación permite inyectar eventos falsos. Cuando el emisor lo soporte, valida una de estas:
- Un **token compartido** por header, **o**
- Una **firma HMAC** del cuerpo (preferible: prueba integridad + origen).

Si aún no está implementado, déjalo registrado como **deuda de seguridad explícita** (en el CLAUDE.md del proyecto o el backlog), no como un olvido silencioso.

## 4. Validación del payload

Valida el cuerpo entrante con un modelo (Pydantic) **en el borde**: obligatorio solo lo que el código realmente usa; el resto opcional. Un payload inválido se rechaza temprano con un error claro, en vez de reventar en lo profundo de la lógica.

## 5. Secretos

Tokens y API keys viven en un secret manager, nunca en el código y nunca con un `default` vacío que permita arrancar mal configurado. Combínalo con una validación de arranque que falle fuerte si falta un secreto crítico (ver `configuration.md §4` y `error-handling.md §3`).

---
**Version:** 1.0
**Last Updated:** 2026-06-29
