# Decisiones y huecos del spec

Huecos y ambigüedades encontrados al leer el spec, con la decisión tomada. **No se modifican los documentos originales ni el
alcance:** cada decisión elige una forma de cumplir lo que el spec ya pide.

| # | Hueco o ambigüedad | Dónde | Decisión |
|---|---|---|---|
| H-01 | ADR-002 pide un «testigo de versión» por libro, pero `data-model.md` §3 no tiene columna de versión y ese documento manda (Art. X) | ADR-002 vs data-model §3 | Compare-and-set sobre `available_copies`; no se añade columna. [ADR-005](adr/adr-005-testigo-de-concurrencia-sin-columna-version.md) |
| H-02 | `GET /health`: T-01 lo pone en `/health`, el contrato en la base `/api/v1` | tasks vs contrato §1 | Se exponen `/api/v1/health` y `/health` |
| H-03 | `GET /books/{id}/cover` se menciona en el texto pero no está en la tabla de endpoints | contrato §3.4 | Se implementa: autenticado, devuelve el binario o 404 |
| H-04 | `/loans/overdue` choca con `/loans/{id}` | contrato §3.6 | `overdue` se declara antes y `{id}` se restringe a UUID |
| H-05 | El sembrador debe crear 3 vencidos usando solo la API, pero el servidor fecha el préstamo con su reloj | T-21 vs HU-06 | Reloj desplazable por entorno, apagado por defecto. [ADR-006](adr/adr-006-reloj-desplazable-para-el-sembrador.md) |
| H-06 | CA-06.4 pide «422 con el conteo actual» sin decir dónde va | spec CA-06.4 | En `message` y en `details: [{field: "member_id", issue: "active_loans=3"}]` |
| H-07 | Laravel responde sus propios formatos de error | RNF-4 | Manejador global que traduce toda excepción al cuerpo único `{code, message, details}` |
| H-08 | Quién crea el esquema `library` | ADR-001 | La primera migración (`CREATE SCHEMA IF NOT EXISTS library`); `infra` solo crea motor, usuario y volumen |
| H-09 | `PUT /books/{id}` no incluye ISBN | contrato §3.4 | El ISBN es inmutable tras crear |
| H-10 | Filtro `status` de `GET /loans` sobre un estado que no se almacena | HU-08 | Se calcula en SQL con `EXISTS` sobre `loan_item` |
| H-11 | `from` y `to` son fechas «ambos inclusive» pero los instantes son UTC | contrato §3.7 | `loaned_at >= from 00:00Z AND loaned_at < (to + 1 día) 00:00Z` |
| H-12 | Libros dados de baja en préstamos antiguos | ADR-003 | Las líneas usan valores congelados; no se une con el catálogo |
| H-13 | La constitución fija `src/Domain/` y el esqueleto de Laravel usa `app/` | constitution Art. I | Código del dominio en `src/` con namespace `Library\`, cargado por Composer; el punto de composición es `src/Bootstrap/Composition/PortBindings.php` |
| H-14 | El 401 por token ausente o vencido, el 405 y el 500 no tienen `code` en el contrato §2 | contrato §2 | 401 usa `INVALID_CREDENTIALS`; 405 y 500 usan `VALIDATION_ERROR` y `INTERNAL_ERROR` respectivamente, sin detalles internos |
| H-15 | Laravel crea su propia tabla `migrations`, que choca con «seis tablas» y «sin DEFAULT» | tasks T-02, data-model §10 | Queda en el esquema `public`, fuera de `library`; las consultas de §10 filtran por `library` y no la ven |
| H-16 | `size` mayor que 50, id que no es UUID y referencias inexistentes en el cuerpo no tienen código en el spec | contrato §1 | `size > 50` → 422 `VALIDATION_ERROR`; id no UUID → 404 `NOT_FOUND`; `member_id`/`book_id` inexistente en el cuerpo → 404 `NOT_FOUND` |
| H-17 | Forma de `GET /categories` y de `/loans/overdue` no está definida | contrato §3.3, §3.6 | Ambos usan el sobre `{items, page, size, total}` del contrato |
