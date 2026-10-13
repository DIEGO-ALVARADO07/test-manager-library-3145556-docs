# DDD — Manager Library (PHP)

Diseño guiado por el dominio que sigue el desarrollo en PHP/Laravel. No cambia el alcance: traduce lo que ya dicen
[`spec.md`](../spec-laravel/spec.md), [`architecture.md`](../spec-laravel/architecture.md) y
[`data-model.md`](../spec-laravel/data-model.md) a un modelo táctico que se pueda implementar y probar.

## 1. Diseño estratégico

### 1.1 Subdominios

| Subdominio | Tipo | Qué resuelve | Historias |
|---|---|---|---|
| **Lending** (préstamos) | **Núcleo** | Prestar, devolver, plazo de 14 días, tope de 3, vencidos | HU-06, HU-07, HU-08, HU-09 |
| **Catalog** (catálogo) | Soporte | Libros, categorías, portadas, baja lógica | HU-03, HU-04 |
| **Membership** (socios) | Soporte | Socios y su documento | HU-05 |
| **Reporting** (reportes) | Soporte | Más prestados sobre valores congelados | HU-10 |
| **Identity & Access** | Genérico | Login, roles, alta de bibliotecarios | HU-01, HU-02 |

### 1.2 Contexto delimitado

Hay **un solo** contexto delimitado: **Library**. El spec descarta microservicios, colas y multi-tenencia
([`architecture.md`](../spec-laravel/architecture.md) §6), así que los subdominios son módulos de un mismo modelo y no contextos separados.
Los clientes (`app`, `page`, `tool`) hablan con el contexto solo por el contrato HTTP, que es su **lenguaje publicado**
([`api-contract.md`](../spec-laravel/api-contract.md)). No hay contexto externo del que defenderse.

### 1.3 Lenguaje ubicuo

El código usa el término en inglés; la interfaz y los mensajes, el término en español (Art. XI).

| Español | Código | Significado exacto |
|---|---|---|
| Libro | `Book` | Título del catálogo; sus ejemplares se cuentan, no se identifican |
| Ejemplar | `copy` | Unidad física: `total_copies`, `available_copies` |
| Socio | `Member` | Quien pide prestado; es un dato, no un usuario |
| Préstamo | `Loan` | Hecho consumado e inmutable salvo `returned_at` de sus líneas |
| Línea de préstamo | `LoanItem` | Un libro dentro de un préstamo, con valores congelados |
| Vencido | `overdue` | Línea sin devolver con `due_at` anterior a ahora |
| Congelado | `frozen` | Copia del título, ISBN y categoría en el instante del préstamo |
| Baja lógica | `retire` | Marca `deleted_at`; el libro sigue en el histórico |
| Usuario | `User` | Operador con rol `admin` o `librarian` |

## 2. Diseño táctico

### 2.1 Agregados

| Raíz | Contiene | Invariantes (las que defiende el dominio) |
|---|---|---|
| `Book` | `Isbn`, `Quantity` total, disponibles, `cover_key`, `deleted_at` | `0 <= available <= total`; `total >= 1`; título no vacío; solo se descuenta si hay ejemplares; un libro dado de baja no se presta |
| `Loan` | colección de `LoanItem` | 1 a 3 líneas; sin libro repetido; `due_at = loaned_at + 14 días`; inmutable salvo devolver una línea; estado calculado |
| `Member` | `DocumentNumber` | documento obligatorio; sin eliminación |
| `User` | rol, hash de clave | nombre en minúsculas y recortado; rol en (`admin`, `librarian`); el dominio nunca ve la clave en claro |

Referencias entre agregados **solo por identificador** (`Loan` guarda `member_id` y `book_id`, no objetos).

**Excepción justificada, tomada del spec:** `Loan::addItem` llama a `Book::withdraw` ([`architecture.md`](../spec-laravel/architecture.md) §3):
descontar el ejemplar y crear la línea son una sola operación. El servicio de aplicación carga ambos agregados y el repositorio de
préstamos los persiste en **una** transacción; ningún otro código modifica `available_copies`.

### 2.2 Objetos de valor

Inmutables (`final readonly`), con validación en el constructor y sin tabla propia: `Isbn` (13 dígitos), `Quantity` (entero ≥ 1),
`DocumentNumber`, `DateRange` (fin ≥ inicio), `Uuid` (v4 generado con `random_bytes`, solo biblioteca estándar), `Role` y `LoanStatus` (enums).

### 2.3 Política de dominio

`LendingPolicy` es una regla pura (sin I/O) que recibe las líneas sin devolver del socio y el instante actual, y decide:
primero `MemberHasOverdue`, después `MemberLimitExceeded` (orden del diagrama de secuencia de `architecture.md` §4).
No es un servicio con estado ni toca la base.

### 2.4 Errores de dominio

Cada violación es una excepción propia en `Domain/Exceptions`; solo el adaptador HTTP las traduce al cuerpo único.

| Excepción | `code` | Estado |
|---|---|---|
| `BookNotAvailable` | `BOOK_NOT_AVAILABLE` | 409 |
| `MemberHasOverdue` | `MEMBER_HAS_OVERDUE` | 409 |
| `MemberLimitExceeded` | `MEMBER_LIMIT_EXCEEDED` | 422 |
| `DuplicateBookInLoan` | `DUPLICATE_BOOK_IN_LOAN` | 422 |
| `ItemAlreadyReturned` | `ITEM_ALREADY_RETURNED` | 409 |
| `BookHasOpenLoans` | `BOOK_HAS_OPEN_LOANS` | 409 |
| `IsbnTaken`, `UsernameTaken`, `DocumentTaken` | `ISBN_TAKEN`, `USERNAME_TAKEN`, `DOCUMENT_TAKEN` | 409 |
| `InvalidDateRange` | `INVALID_DATE_RANGE` | 422 |
| `InvalidValue` (objetos de valor y reglas de entrada) | `VALIDATION_ERROR` | 422 |
| `NotFound` | `NOT_FOUND` | 404 |
| `InvalidCredentials` | `INVALID_CREDENTIALS` | 401 |
| `Forbidden` (lo lanza el middleware de rol) | `FORBIDDEN_ROLE` | 403 |

### 2.5 Repositorios y puertos

Los define la capa de aplicación (Art. II). Ninguna firma menciona tipos del framework ni `Illuminate`. Los nombres son los del código en `src/Application/Ports/`.

| Puerto | Responsabilidad |
|---|---|
| `BookRepository` | `find` (Q2), `findMany` (Q3), `search` (Q1), `activeIsbnExists`, `hasOpenLoans`, `insert`, `update` (compare-and-set), `setCoverKey`, `retire` |
| `LoanRepository` | `find` (Q5), `search` (Q6), `unreturnedItemsOfMember` (Q7), `overdueItems` (Q8), `saveLending` (préstamo + libros, una transacción), `saveReturn` |
| `MemberRepository` | `find`, `search`, `documentExists`, `insert`, `activeLoans` |
| `UserRepository` | `findByUsername` (Q10), `insert` |
| `CategoryReader` | `all` (Q4), `find`: lectura de las cinco categorías sembradas |
| `ReportReader` | `mostBorrowed`: consulta agregada del reporte (Q9) |
| `Clock`, `PasswordHasher`, `TokenIssuer`, `CoverStorage` | servicios externos |

### 2.6 Eventos de dominio

**No hay.** El spec los excluye (`architecture.md` §6: «sin eventos, sin colas»). El estado se calcula y los efectos ocurren
dentro de la misma transacción.

## 3. Cómo se materializa en Laravel

| Concepto DDD | Dónde vive | Regla |
|---|---|---|
| Agregados, objetos de valor, política, errores | `src/Domain/` | PHP puro: sin `Illuminate`, Eloquent ni Carbon (Art. I) |
| Casos de uso (uno por servicio) y puertos | `src/Application/` | Solo conoce `Domain` (Art. II) |
| Controladores, requests, middleware, mapeo de errores | `src/Adapters/Inbound/Http/` | Delgados: validan formato, llaman al servicio, devuelven JSON |
| Repositorios | `src/Adapters/Outbound/Persistence/` | Consultas con el Query Builder de Laravel (sin modelos Eloquent); cada fila se traduce a un agregado |
| Comando de arranque del admin | `src/Adapters/Inbound/Console/` | `php artisan library:bootstrap-admin` (D-10) |
| Hash, JWT, portadas, reloj | `src/Adapters/Outbound/{Security,Storage,Time}/` | Implementan puertos |
| Enlace puerto → adaptador | `src/Bootstrap/Composition/PortBindings.php` | Único lugar que instancia infraestructura (Art. III) |

Namespace raíz `Library\` cargado por Composer desde `src/` (decisión H-13 en [`decisiones-y-huecos.md`](../spec-laravel/decisiones-y-huecos.md)).

## 4. Transacciones y concurrencia

- Un caso de uso = una transacción, abierta por el repositorio que persiste (la aplicación no conoce `DB`).
- Prestar: todas las líneas o ninguna. Cada descuento usa compare-and-set sobre `available_copies` ([ADR-005](../spec-laravel/adr/adr-005-testigo-de-concurrencia-sin-columna-version.md)).
- Devolver: `UPDATE ... WHERE returned_at IS NULL`; 0 filas afectadas implica `ItemAlreadyReturned`.
- La última barrera es `ck_book_available_range` en la base.
