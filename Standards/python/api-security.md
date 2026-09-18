# API Security Standard

> ⚠️ **STATUS: DRAFT PROPOSED BY GEMINI - PENDING VALIDATION WITH CLAUDE**
> <!-- VALIDAR CON CLAUDE: Review Rate Limiting with SlowAPI/Redis vs Cloud Armor, Debug routes quarantine (CWE-489), and Client IP extraction behind reverse proxies -->

Reglas de seguridad para APIs (FastAPI) que exponen endpoints internos, reciben webhooks externos o interactúan con servicios cloud (Cloud Run).

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

---

## 2. Endpoints internos y autenticación de servicios

Cada endpoint sensible (admin, sweep, procesamiento interno) exige su token por header (`X-Admin-Token`, `X-Sweep-Token`, …), verificado con `verify_token`. El servicio Cloud Run puede ser público (`--allow-unauthenticated`) si lo llaman sistemas externos: la protección va **por token en cada endpoint**, no cerrando el servicio.

---

## 3. Webhooks entrantes e integridad criptográfica

Un webhook público sin autenticación permite inyectar eventos falsos. Cuando el emisor lo soporte, valida una de estas:
- Un **token compartido** por header, **o**
- Una **firma HMAC** del cuerpo (preferible: prueba integridad + origen).

El cuerpo del request debe leerse en bytes crudos (`await request.body()`) **antes** de parsear JSON para asegurar que el cálculo HMAC coincida con el payload exacto enviado por el proveedor (ej: Meta `X-Hub-Signature-256`).

---

## 4. Validación temprana y límite de tamaño de payload

1. Valida el cuerpo entrante con un modelo (Pydantic) **en el borde**: obligatorio solo lo que el código realmente usa; el resto opcional.
2. **Límite de tamaño:** Para evitar ataques de Denegación de Servicio (DoS) por agotamiento de memoria RAM (buffers gigantes), implementa un middleware o verificación de `Content-Length` (ej: max 10MB para media, max 256KB para JSON regular).

---

## 5. Cuarentena de endpoints de depuración (CWE-489)

Los endpoints diseñados para pruebas locales, dry-runs o inyección de payloads simulados (ej: `/debug/*`) **nunca deben exponerse incondicionalmente en entornos productivos**.

- **Riesgo:** Permiten evasión de autenticación, suplantación de eventos externos y abuso de cuotas o servicios de mensajería (SMS/WhatsApp).
- **Regla:** Montar los enrutadores de depuración únicamente bajo bandera de entorno explícita:
  ```python
  if os.getenv("ENABLE_DEBUG_ROUTES", "false").strip().lower() in ("true", "1", "yes"):
      app.include_router(debug_router)
  ```
- En producción (Cloud Run), `ENABLE_DEBUG_ROUTES` debe permanecer ausente o fijarse en `false`.

---

## 6. Rate Limiting y Protección contra Abuso / DoS

Para prevenir ataques de fuerza bruta, scraping, saturación de CPU o agotamiento de presupuesto en APIs externas (Meta Graph API, OpenAI):

### A. Capa de Aplicación (FastAPI + SlowAPI)
- En servicios con una sola instancia o desarrollo local, el backend en memoria de `SlowAPI` es suficiente.
- **Arquitectura Horizontal / Cloud Run Serverless:** Dado que Cloud Run escala a múltiples instancias efímeras e independientes, el conteo en memoria local es ineficaz (el tráfico se dispersa entre instancias). En producción multi-instancia se debe utilizar `SlowAPI` respaldado por un almacenamiento global compartido (**Redis** / Memorystore) o delegar el límite a la infraestructura.
- **Identificador de Límite:** Aplicar límites por IP de cliente (`get_remote_address`) o por identificador de usuario/API Key.

### B. Capa de Infraestructura (Cloud Armor / API Gateway)
- Cuando el servicio está detrás de un Load Balancer (HTTPS External Load Balancer), delegar la protección contra DoS volumétrico y rate limiting a **Google Cloud Armor**. Esto frena el tráfico malicioso antes de que consuma recursos del contenedor Cloud Run.

---

## 7. Mitigación de IP Spoofing en Proxies y Cloud Run

Cuando un servicio corre detrás de proxies inversos o balanceadores (Cloud Run, Cloudflare):
- El header `request.client.host` suele corresponder a la IP del balanceador interno de Google, no a la IP real del atacante.
- Si se lee `X-Forwarded-For`, se debe extraer la IP del cliente real de forma segura (normalmente el primer valor de la lista o validando que el proxy sea confiable) para evitar que un atacante inyecte una IP falsa en el header y evada el rate limiting.

---

## 8. Gestión de secretos y fallos en arranque

Tokens y API keys viven en un secret manager (ej: GCP Secret Manager), nunca en el código y nunca con un `default` hardcodeado que permita arrancar mal configurado o apuntar por error a recursos de otro cliente. Combínalo con una validación de arranque (`lifespan`) que falle fuerte si falta un secreto crítico (ver `configuration.md §4` y `error-handling.md §3`).

---

**Version:** 1.1 (Draft)  
**Last Updated:** 2026-09-17
