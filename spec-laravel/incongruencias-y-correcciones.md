# Incongruencias del spec y cómo corregirlas

Incongruencias, vacíos y errores encontrados al leer el spec y al implementarlo. **Ninguno se corrigió en los documentos originales** y no se cambió el alcance: cada fila dice qué se hizo en la implementación para seguir adelante y **cuál sería la corrección** si el propietario del spec decide aplicarla.

Las decisiones tomadas están en [`decisiones-y-huecos.md`](decisiones-y-huecos.md). Los dos casos más importantes tienen además un ADR propio.

| # | Incongruencia o error | Dónde | Qué se hizo (sin tocar el spec) | Cómo corregirlo |
|---|---|---|---|---|
| I-01 | ADR-002 y D-04 exigen un «testigo de versión» por libro, pero `book` no tiene columna de versión | ADR-002 vs `data-model.md` §3 | Compare-and-set sobre `available_copies` ([ADR-005](adr/adr-005-testigo-de-concurrencia-sin-columna-version.md)) | Añadir `version` a `book`, actualizar `data-model.md` §3 y §4 (Art. X) y comparar la versión en el `WHERE` |
| I-02 | T-21 pide 3 vencidos creados solo con la API, pero HU-06 fecha el préstamo con el reloj del servidor | T-21 vs HU-06, CA-06.6 | Reloj desplazable por entorno, apagado por defecto ([ADR-006](adr/adr-006-reloj-desplazable-para-el-sembrador.md)) | Permitir `loaned_at` solo para `admin` en el contrato, o que T-21 admita sembrar por SQL |
| I-03 | T-01 pone `GET /health` en la raíz y el contrato lo define bajo la base `/api/v1` | `tasks.md` T-01 vs contrato §1, §3.1 | Se exponen `/api/v1/health` y `/health` | Escribir una sola ruta en el contrato y en T-01 |
| I-04 | El texto del contrato menciona `GET /books/{id}/cover`, pero la tabla de endpoints no lo lista | contrato §3.4 | Se implementó (autenticado; binario o 404) | Añadir la fila a la tabla de §3.4 |
| I-05 | `/loans/overdue` choca con `/loans/{id}` | contrato §3.6 | `overdue` se declara antes y `{id}` se restringe a UUID | Documentar que `overdue` es un segmento reservado, o mover la ruta (por ejemplo `/overdue-items`) |
| I-06 | §2 no tiene código para un 401 por token ausente o vencido, ni para 405 ni para 500 | contrato §2 | 401 reutiliza `INVALID_CREDENTIALS`; 405 se responde como 404 `NOT_FOUND`; 5xx usa `INTERNAL_ERROR` | Añadir `UNAUTHENTICATED` e `INTERNAL_ERROR` a §2 |
| I-07 | No define la forma de `GET /categories`, ni si `/loans/overdue` se pagina | contrato §3.3, §3.6 | Ambos usan el sobre `{items, page, size, total}` | Definir las dos respuestas en el contrato |
| I-08 | `PUT /books/{id}` no incluye ISBN y no dice si es inmutable | contrato §3.4 | El ISBN es inmutable | Declararlo en §3.4 |
| I-09 | T-02 exige «seis tablas» y «ninguna columna con DEFAULT», pero Laravel crea su propia tabla `migrations` (con una columna serial) | `tasks.md` T-02, `data-model.md` §2 y §10 | La tabla queda en el esquema `public`, fuera de `library`; las consultas de §10 no la ven | Aclarar en §2 y §10 que la tabla de control del framework de migraciones queda fuera del modelo |
| I-10 | La constitución fija `src/library/domain/` (Python) y las rutas del esqueleto de Laravel son `app/` | `constitution.md` Art. I y III | Código en `src/` con namespace `Library\` (decisión H-13) | Fijar en `spec-laravel` la raíz de código y el namespace |
| I-11 | `size > 50`, un id que no es UUID y referencias inexistentes en el cuerpo no tienen código ni estado | contrato §1 y §2 | 422, 404 y 404 respectivamente; `category_id` inexistente → 422 | Añadir una tabla de validaciones de entrada al contrato §1 |
| I-12 | CA-06.4 pide «422 con el conteo actual» sin decir en qué campo va | spec CA-06.4 | `message` y `details: [{field: "member_id", issue: "active_loans=N"}]` | Especificar el formato en el contrato |
| I-13 | `data-model.md` nombra las claves primarias `PK_*` (convención de EF Core) y PostgreSQL usa `*_pkey` | `data-model.md` §4 | Se usaron los nombres por defecto del motor | Usar nombres neutrales o decir que el nombre lo elige el motor |
| I-14 | La introducción de `api-contract.md` dice «Python o .NET» en una versión pensada para Laravel | `spec-laravel/api-contract.md` | Se dejó igual para no alterar el contrato | Mencionar Laravel en la copia de `spec-laravel` |
| I-15 | `Loan.AddItem` modifica dos agregados (`Loan` y `Book`) en una sola operación | `architecture.md` §3 | Se hizo así, dentro de una transacción, como describe el spec (ver `spec-php/ddd.md` §2.1) | Separar con un servicio de dominio que coordine la retirada de ejemplares |
| I-16 | El tope de 3 libros por socio no se puede imponer con un `CHECK`; dos préstamos simultáneos del mismo socio podrían superarlo | `data-model.md` §4, RNF-2 | No se cubre: RNF-2 solo habla del último ejemplar | Bloquear la fila del socio (`SELECT ... FOR UPDATE`) durante el préstamo, o añadir una restricción en el motor |

## Condiciones de la prueba

La conexión a internet durante la prueba fue por **datos móviles propios del aprendiz**. Esto afectó las descargas de imágenes de Docker, de paquetes de Composer y de npm, pero no cambió ninguna decisión técnica.
