# Database Standards

Reglas para diseñar y nombrar esquemas SQL (MySQL/PostgreSQL) en cualquier proyecto.

---

## 1. Primary Keys — nunca `id` a secas

Toda PK se llama `id_<entidad>`, nunca solo `id`. Esto hace que los joins sean
autoexplicativos sin necesidad de alias: al ver `id_merchant` en cualquier
tabla, queda claro de dónde viene sin tener que revisar el schema.

```sql
-- ❌ BAD
CREATE TABLE merchants (
    id INT AUTO_INCREMENT PRIMARY KEY,
    ...
);

-- ✅ GOOD
CREATE TABLE merchants (
    id_merchant INT PRIMARY KEY,
    ...
);
```

## 2. Foreign Keys — mismo nombre que la PK que referencian

Por defecto, una FK se llama exactamente igual que la PK a la que apunta.
Nunca `merchant` o `mid` cuando la PK es `id_merchant`.

```sql
CREATE TABLE orders (
    id_buho_order BIGINT PRIMARY KEY,
    id_merchant INT,  -- mismo nombre que merchants.id_merchant
    FOREIGN KEY (id_merchant) REFERENCES merchants(id_merchant)
);
```

**Excepción:** cuando reusar el nombre exacto de la PK sería ambiguo, o
cuando un nombre distinto comunica mejor el rol de esa relación, usa ese
nombre — siempre que siga siendo obvio a qué PK apunta. No es una regla
limitada a un caso particular (auto-referencias, roles, etc.); aplica en
general a cualquier FK donde el nombre "natural" no sea el más claro.

```sql
-- Caso 1: FK auto-referenciada (misma tabla) — "id_empleado" otra vez
-- sería ambiguo (¿el empleado o su jefe?).
CREATE TABLE empleados (
    id_empleado INT PRIMARY KEY,
    id_jefe INT,
    FOREIGN KEY (id_jefe) REFERENCES empleados(id_empleado)
);

-- Caso 2: FK a una tabla distinta, pero el rol de la relación necesita
-- quedar explícito (y la tabla podría ganar otras FKs al mismo destino
-- con otro rol más adelante, ej. id_secretario_departamento).
CREATE TABLE departamentos (
    id_departamento INT PRIMARY KEY,
    id_jefe_departamento INT,
    FOREIGN KEY (id_jefe_departamento) REFERENCES empleados(id_empleado)
);
```

## 3. Preferir clave natural sobre surrogate `id`

Si la entidad ya tiene un identificador único y estable en el sistema de
origen (ej. el `merchant_id` de Shipstream), úsalo como PK directamente —
no agregues un `id INT AUTO_INCREMENT` adicional. Un surrogate solo se
justifica cuando no existe ninguna clave natural confiable.

## 4. Nombres completos — sin abreviaciones, en ningún lado

Misma regla que en `python/code-standards.md` §4, aplicada a columnas: nada
de `mid`, `addr`, `tmp`, `qty`. Si el dominio tiene una palabra completa para
algo, esa es la que va en la columna.

```sql
-- ❌ BAD
merchant_ph VARCHAR(50)

-- ✅ GOOD
merchant_phone VARCHAR(50)
```

## 5. NOT NULL refleja quién garantiza el dato

Si una columna SIEMPRE la pone el sistema automáticamente (ej. un proceso de
ingestión que resuelve el dato de una API antes de insertar la fila), va
`NOT NULL` — así un valor faltante falla fuerte y visible en vez de guardarse
silenciosamente incompleto. Si el dato se captura después, manualmente, por
un humano (ej. vía un panel de administración), va nullable.

```sql
CREATE TABLE merchants (
    id_merchant INT PRIMARY KEY,
    merchant_name VARCHAR(255) NOT NULL,  -- siempre lo resuelve el sistema
    merchant_phone VARCHAR(50),           -- lo captura un humano después
    chatbot_url VARCHAR(500)              -- lo captura un humano después
);
```

---

**Version:** 1.1
**Last Updated:** 2026-08-08
