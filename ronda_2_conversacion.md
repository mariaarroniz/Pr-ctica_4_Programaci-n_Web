# Práctica 4 — Transcripción de la Conversación (Ronda 2)

> **Modelo:** Gemini 3.8 Flash (High) / Antigravity
> **Fecha:** 26 de octubre de 2026
> **Contexto:** Ronda 2 con contexto (ESPECIFICACION.md y servidor.test.cjs)

## Resumen de turnos

| Turno | Intervención del usuario | Tests en verde después |
|---|---|---|
| 1 | Prompt inicial con contexto completo, restricciones y petición de plan en pasos | - (Planificación, sin código) |
| 2 | «adelante» (Paso 1: Estructura base, datos iniciales y GET /libros) | 6 / 14 |
| 3 | «adelante» (Paso 2: GET /libros/:isbn) | 7 / 14 |
| 4 | «adelante» (Paso 3: POST /prestamos) | 12 / 14 |
| 5 | «adelante» (Paso 4: GET /socios/:id/prestamos) | 13 / 14 |
| 6 | «adelante» (Paso 5: DELETE /prestamos/:id y comprobación R9) | 14 / 14 |

---

## Conversación completa

### Turno 1 — Usuario

Tengo que implementar una API REST de préstamos de biblioteca.
La especificación completa está en ESPECIFICACION.md y los tests en
servidor.test.cjs. Léelos los dos antes de nada.

Restricciones:
- Node.js 22, sin dependencias (solo módulos de Node).
- Todo en un único fichero: servidor.js.
- No modifiques servidor.test.cjs bajo ningún concepto.
- El puerto sale de la variable de entorno PORT (3012 por defecto).

Todavía no escribas código. Primero dame un plan en pasos pequeños, y para
cada paso dime cómo se comprueba (qué tests deberían pasar al acabarlo).
Cuando te diga "adelante", haz solo el paso 1, ejecuta node --test y
dime qué tests pasan antes de seguir con el siguiente.

### Turno 1 — Asistente

He leído atentamente tanto `ESPECIFICACION.md` como `servidor.test.cjs`, además de comprobar las restricciones del ejercicio (Node.js 22, sin dependencias externas, todo en `servidor.js`, `PORT` con 3012 por defecto).

Como los tests arrancan el servidor haciendo primero una petición a `GET /libros` en el hook `before`, el servidor debe existir y responder a esa ruta desde el primer paso para que la suite no aborte.

A continuación te presento la propuesta de plan dividido en **5 pasos pequeños**, detallando en cada uno qué se implementa y qué tests deberían quedar en verde:

---

### Plan de implementación por pasos

#### **Paso 1: Servidor base, datos iniciales y `GET /libros` + 404 por defecto**
* **Qué se hace:**
  * Crear `servidor.js` usando únicamente el módulo nativo `node:http`.
  * Leer el puerto de `process.env.PORT || 3012`.
  * Definir en memoria los datos iniciales (los 5 libros y los 2 socios con la función de normalización de ISBN para soportar con/sin guiones).
  * Implementar el helper para respuestas JSON (`Content-Type: application/json` y formato `{ "error": ... }`).
  * Implementar la ruta `GET /libros` devolviendo los 5 libros con el campo `disponible: true`.
  * Gestionar rutas no existentes devolviendo `404 Not Found` con mensaje de error JSON.
* **Comprobación (tests que deben pasar):**
  * `0 · sin dependencias`
  * `1 · GET /libros: los cinco, todos disponibles, en JSON`
  * `13 · una ruta que no existe: 404`
  *(Resultado esperado: 3/14 tests pasando)*.

---

#### **Paso 2: Consulta de un libro por ISBN (`GET /libros/:isbn`)**
* **Qué se hace:**
  * Enrutar `GET /libros/:isbn`.
  * Buscar el libro aceptando el ISBN con o sin guiones (según requisito R9).
  * Devolver el libro con su estado `disponible`.
  * Devolver `404` con `{ "error": ... }` si el libro no existe.
* **Comprobación (tests que deben pasar):**
  * Se añaden:
    * `2 · GET /libros/:isbn devuelve ese libro`
    * `3 · un ISBN que no existe: 404 con { error }`
  *(Resultado esperado: 5/14 tests pasando)*.

---

#### **Paso 3: Creación de préstamos (`POST /prestamos`) y sus validaciones**
* **Qué se hace:**
  * Parser del cuerpo JSON de peticiones entrantes.
  * Enrutar `POST /prestamos` `{ isbn, socio }`.
  * Validar los errores estrictamente en el orden de la especificación:
    1. Datos faltantes o tipos incorrectos (`isbn` no string o `socio` no number) → `400`.
    2. Libro o socio inexistentes → `404`.
    3. Libro ya prestado → `409`.
    4. El socio ya tiene 3 préstamos activos → `409`.
  * Si es válido: generar ID autoincremental (iniciando en 1), calcular `inicio` (fecha UTC actual `AAAA-MM-DD`) y `devolucion` (+14 días), almacenar el préstamo y responder `201 Created` con el préstamo creado.
* **Comprobación (tests que deben pasar):**
  * Se añaden:
    * `4 · POST /prestamos: 201, con fechas de inicio y de devolución`
    * `5 · un libro prestado ya no está disponible`
    * `6 · prestar un libro ya prestado: 409`
    * `7 · faltan datos: 400`
    * `8 · socio o libro que no existen: 404`
    * `9 · como mucho tres préstamos a la vez por socio: el cuarto da 409`
  *(Resultado esperado: 11/14 tests pasando)*.

---

#### **Paso 4: Consulta de préstamos por socio (`GET /socios/:id/prestamos`)**
* **Qué se hace:**
  * Enrutar `GET /socios/:id/prestamos`.
  * Validar si el socio existe; si no existe, responder `404`.
  * Si existe, obtener y responder con la lista de préstamos activos de ese socio (`200 OK`).
* **Comprobación (tests que deben pasar):**
  * Se añade:
    * `10 · GET /socios/:id/prestamos: los activos del socio`
  *(Resultado esperado: 12/14 tests pasando)*.

---

#### **Paso 5: Devolución de préstamos (`DELETE /prestamos/:id`)**
* **Qué se hace:**
  * Enrutar `DELETE /prestamos/:id`.
  * Buscar el préstamo por ID. Si no existe, responder `404`.
  * Eliminar el préstamo (el libro vuelve a estar disponible).
  * Responder `204 No Content` (sin cuerpo).
* **Comprobación (tests que deben pasar):**
  * Se añaden:
    * `11 · DELETE /prestamos/:id devuelve el libro: 204`
    * `12 · devolver un préstamo que no existe: 404`
  *(Resultado esperado: 14/14 tests en verde)*.

---

No he modificado ni creado ningún archivo todavía. Cuando me digas **"adelante"**, procederé a implementar únicamente el **Paso 1**, ejecutaré `node --test` y te mostraré el resultado antes de continuar.

---

### Turno 2 — Usuario

adelante

### Turno 2 — Asistente

He completado el **Paso 1** (estructura base, datos iniciales en memoria, `GET /libros` y manejo de 404 por defecto).

Al ejecutar `node --test`, este ha sido el resultado:

### Tests que pasan (6 de 14):
* ✔ `0 · sin dependencias`
* ✔ `1 · GET /libros: los cinco, todos disponibles, en JSON`
* ✔ `3 · un ISBN que no existe: 404 con { error }` *(pasa por el fallback de 404)*
* ✔ `8 · socio o libro que no existen: 404` *(pasa por el fallback de 404)*
* ✔ `12 · devolver un préstamo que no existe: 404` *(pasa por el fallback de 404)*
* ✔ `13 · una ruta que no existe: 404`

### Tests que fallan (8 de 14):
* ✖ `2 · GET /libros/:isbn devuelve ese libro` *(esperaba 200, obtuvo 404)*
* ✖ `4 · POST /prestamos: 201, con fechas de inicio y de devolución` *(esperaba 201, obtuvo 404)*
* ✖ `5 · un libro prestado ya no está disponible`
* ✖ `6 · prestar un libro ya prestado: 409`
* ✖ `7 · faltan datos: 400`
* ✖ `9 · como mucho tres préstamos a la vez por socio: el cuarto da 409`
* ✖ `10 · GET /socios/:id/prestamos: los activos del socio`
* ✖ `11 · DELETE /prestamos/:id devuelve el libro: 204`

---

Dime cuando quieras que continúe con el **Paso 2** (`GET /libros/:isbn` soportando ISBN con o sin guiones y su 404 correspondiente).

---

### Turno 3 — Usuario

adelante

### Turno 3 — Asistente

He completado el **Paso 2** (`GET /libros/:isbn` soportando ISBN con o sin guiones y su 404 correspondiente).

Al ejecutar `node --test`, este ha sido el resultado:

### Tests que pasan (7 de 14):
* ✔ `0 · sin dependencias`
* ✔ `1 · GET /libros: los cinco, todos disponibles, en JSON`
* ✔ `2 · GET /libros/:isbn devuelve ese libro` *(nuevo en verde)*
* ✔ `3 · un ISBN que no existe: 404 con { error }` *(ahora pasando con la lógica propia de la ruta)*
* ✔ `8 · socio o libro que no existen: 404`
* ✔ `12 · devolver un préstamo que no existe: 404`
* ✔ `13 · una ruta que no existe: 404`

### Tests que fallan (7 de 14):
* ✖ `4 · POST /prestamos: 201, con fechas de inicio y de devolución`
* ✖ `5 · un libro prestado ya no está disponible`
* ✖ `6 · prestar un libro ya prestado: 409`
* ✖ `7 · faltan datos: 400`
* ✖ `9 · como mucho tres préstamos a la vez por socio: el cuarto da 409`
* ✖ `10 · GET /socios/:id/prestamos: los activos del socio`
* ✖ `11 · DELETE /prestamos/:id devuelve el libro: 204`

---

Dime cuando quieras que proceda con el **Paso 3** (`POST /prestamos` con sus validaciones en orden, cálculo de fechas UTC y disponibilidad).

---

### Turno 4 — Usuario

adelante

### Turno 4 — Asistente

He completado el **Paso 3** (`POST /prestamos` con lectura de body JSON, cálculo de fechas en UTC y validaciones ordenadas).

Al ejecutar `node --test`, este ha sido el resultado:

### Tests que pasan (12 de 14):
* ✔ `0 · sin dependencias`
* ✔ `1 · GET /libros: los cinco, todos disponibles, en JSON`
* ✔ `2 · GET /libros/:isbn devuelve ese libro`
* ✔ `3 · un ISBN que no existe: 404 con { error }`
* ✔ `4 · POST /prestamos: 201, con fechas de inicio y de devolución` *(nuevo en verde)*
* ✔ `5 · un libro prestado ya no está disponible` *(nuevo en verde)*
* ✔ `6 · prestar un libro ya prestado: 409` *(nuevo en verde)*
* ✔ `7 · faltan datos: 400` *(nuevo en verde)*
* ✔ `8 · socio o libro que no existen: 404`
* ✔ `9 · como mucho tres préstamos a la vez por socio: el cuarto da 409` *(nuevo en verde)*
* ✔ `12 · devolver un préstamo que no existe: 404`
* ✔ `13 · una ruta que no existe: 404`

### Tests que fallan (2 de 14):
* ✖ `10 · GET /socios/:id/prestamos: los activos del socio`
* ✖ `11 · DELETE /prestamos/:id devuelve el libro: 204`

---

Dime cuando quieras que continúe con el **Paso 4** (`GET /socios/:id/prestamos`).

---

### Turno 5 — Usuario

adelante

### Turno 5 — Asistente

He completado el **Paso 4** (`GET /socios/:id/prestamos`).

Al ejecutar `node --test`, este ha sido el resultado:

### Tests que pasan (13 de 14):
* ✔ `0 · sin dependencias`
* ✔ `1 · GET /libros: los cinco, todos disponibles, en JSON`
* ✔ `2 · GET /libros/:isbn devuelve ese libro`
* ✔ `3 · un ISBN que no existe: 404 con { error }`
* ✔ `4 · POST /prestamos: 201, con fechas de inicio y de devolución`
* ✔ `5 · un libro prestado ya no está disponible`
* ✔ `6 · prestar un libro ya prestado: 409`
* ✔ `7 · faltan datos: 400`
* ✔ `8 · socio o libro que no existen: 404`
* ✔ `9 · como mucho tres préstamos a la vez por socio: el cuarto da 409`
* ✔ `10 · GET /socios/:id/prestamos: los activos del socio` *(nuevo en verde)*
* ✔ `12 · devolver un préstamo que no existe: 404`
* ✔ `13 · una ruta que no existe: 404`

### Tests que fallan (1 de 14):
* ✖ `11 · DELETE /prestamos/:id devuelve el libro: 204`

---

Dime cuando quieras que proceda con el **Paso 5** (`DELETE /prestamos/:id` y verificación de los 14 tests).

---

### Turno 6 — Usuario

adelante

### Turno 6 — Asistente

He completado el **Paso 5** (`DELETE /prestamos/:id` y respuesta 204 sin cuerpo).

Al ejecutar `node --test`, **los 14 tests están en verde**:

```text
✔ 0 · sin dependencias (12.849492ms)
✔ 1 · GET /libros: los cinco, todos disponibles, en JSON (4.461198ms)
✔ 2 · GET /libros/:isbn devuelve ese libro (3.663386ms)
✔ 3 · un ISBN que no existe: 404 con { error } (3.431413ms)
✔ 4 · POST /prestamos: 201, con fechas de inicio y de devolución (5.304527ms)
✔ 5 · un libro prestado ya no está disponible (1.845598ms)
✔ 6 · prestar un libro ya prestado: 409 (2.839619ms)
✔ 7 · faltan datos: 400 (6.635039ms)
✔ 8 · socio o libro que no existen: 404 (3.640627ms)
✔ 9 · como mucho tres préstamos a la vez por socio: el cuarto da 409 (5.145854ms)
✔ 10 · GET /socios/:id/prestamos: los activos del socio (3.512985ms)
✔ 11 · DELETE /prestamos/:id devuelve el libro: 204 (4.372089ms)
✔ 12 · devolver un préstamo que no existe: 404 (1.390985ms)
✔ 13 · una ruta que no existe: 404 (1.00731ms)

ℹ tests 14
ℹ suites 0
ℹ pass 14
ℹ fail 0
```

---

### Verificación adicional del requisito sin test (R9)
Como se indicaba en el enunciado de la práctica, el requisito **R9** (*aceptar ISBN con o sin guiones en rutas y cuerpo, pero devolver siempre la versión canónica con guiones*) no estaba cubierto en la batería de tests automatizada.

He realizado una comprobación específica con:
1. `GET /libros/9788400000028` (sin guiones) $\rightarrow$ Devuelve `200 OK`, título `"La Regenta"` y campo `isbn: "978-84-0000-002-8"` (con guiones).
2. `POST /prestamos` enviando `{ "isbn": "9788400000011", "socio": 1 }` $\rightarrow$ Crea el préstamo con éxito (`201 Created`) devolviendo `isbn: "978-84-0000-001-1"` (con guiones).

La API cumple todas las restricciones y la especificación completa en [servidor.js](file:///home/mariaarroniz/Documentos/uni/4ºaño/prograWeb/practicas/Practica_4_Del_Prompt_Escueto_Al_Verde/Practica4%20Codigo/codigo_2/servidor.js).

---

### Turno 7 — Usuario

Transcribe toda la conversacion a un .md

