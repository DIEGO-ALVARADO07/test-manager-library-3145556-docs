# Trazabilidad

De cada historia y criterio de aceptación a lo que lo implementa y a la prueba que lo comprueba. Rutas bajo `/api/v1`. Las clases viven en `api/src/` y los tests en `api/tests/`.

Todos los criterios de esta tabla tienen un test que pasa. Un criterio sin test aparece en la sección final.

| HU / CA | Endpoint | Clase | Test | Estado |
|---|---|---|---|---|
| HU-01 · CA-01.1 token de 8 h | `POST /auth/login` | `LoginService`, `JwtTokenIssuer` | `Contract/AuthTest` — token valido 8 horas | Cumple |
| CA-01.2 401 sin revelar qué falló | `POST /auth/login` | `LoginService`, `ErrorMapper` | `Contract/AuthTest` — mismo 401 usuario y clave | Cumple |
| CA-01.3 401 sin token, 403 sin rol | todos | `JwtAuth`, `RequireRole` | `Contract/AuthTest` — token y FORBIDDEN_ROLE | Cumple |
| HU-02 · CA-02.1 crear bibliotecario | `POST /users` | `CreateLibrarianService` | `Contract/AuthTest` | Cumple |
| CA-02.2 usuario normalizado y único | `POST /users` | `User::normalize`, `PgUserRepository` | `Unit/UserTest`, `Contract/AuthTest` | Cumple |
| CA-02.3 rol admin → 422, sin clave en la respuesta | `POST /users` | `CreateLibrarianService`, `Presenter::user` | `Contract/AuthTest` | Cumple |
| HU-03 · CA-03.1 crear libro | `POST /books` | `CreateBookService`, `Book::create` | `Contract/BooksTest`, `Unit/BookTest` | Cumple |
| CA-03.2 ISBN de 13 dígitos, único entre activos | `POST /books` | `Isbn`, `PgBookRepository`, índice parcial | `Unit/ValueObjectsTest`, `Contract/BooksTest` | Cumple |
| CA-03.3 total ≥ 1 y ajuste de disponibles | `PUT /books/{id}` | `Book::change`, `UpdateBookService` | `Unit/BookTest`, `Contract/BooksTest` | Cumple |
| CA-03.4 portada JPG/PNG ≤ 2 MB, clave opaca | `PUT/DELETE/GET /books/{id}/cover` | `UploadCoverService`, `DiskCoverStorage` | `Contract/BooksTest` | Cumple |
| CA-03.5 baja lógica; 409 con préstamos abiertos | `DELETE /books/{id}` | `DeleteBookService` | `Contract/BooksTest`, `Integration/SchemaTest` | Cumple |
| HU-04 · CA-04.1/04.2 búsqueda paginada con total | `GET /books` | `SearchBooksService`, `PgBookRepository::search` | `Contract/BooksTest` | Cumple |
| CA-04.3 404 si no existe o está de baja | `GET /books/{id}` | `GetBookService` | `Contract/BooksTest` | Cumple |
| HU-05 · CA-05.1 crear socio, documento único | `POST /members` | `CreateMemberService` | `Contract/LoansTest` | Cumple |
| CA-05.2 listar y buscar | `GET /members`, `GET /members/{id}` | `SearchMembersService`, `GetMemberService` | `Contract/LoansTest` | Cumple |
| CA-05.3 los socios no se eliminan | — (no hay ruta DELETE) | — | `Contract/AuthTest` — rutas inexistentes responden 404 | Cumple |
| HU-06 · CA-06.1 1 a 3 líneas | `POST /loans` | `LendBooksService`, `Loan` | `Unit/LoanTest`, `Application/LendAndReturnTest` | Cumple |
| CA-06.2 sin libro repetido | `POST /loans` | `Loan::addItem` | `Unit/LoanTest`, `Contract/LoansTest` | Cumple |
| CA-06.3 descuenta 1 ejemplar; sin ejemplares no registra nada | `POST /loans` | `PgLoanRepository::saveLending` | `Contract/LoansTest`, `Concurrency/LastCopyTest` | Cumple |
| CA-06.4 máximo 3 sin devolver, 422 con conteo | `POST /loans` | `LendingPolicy` | `Unit/LoanTest`, `Contract/LoansTest` | Cumple |
| CA-06.5 socio con vencidos → 409 | `POST /loans` | `LendingPolicy` | `Unit/LoanTest`, `Contract/LoansTest` | Cumple |
| CA-06.6 vence a 14 días, valores congelados | `POST /loans` | `Loan::addItem` | `Unit/LoanTest`, `Contract/LoansTest` | Cumple |
| CA-06.7 inmutable | — (no hay PUT ni DELETE) | — | `Contract/AuthTest` | Cumple |
| HU-07 · CA-07.1 devolver una línea | `POST /loans/{loan_id}/items/{item_id}/return` | `ReturnItemService` | `Contract/LoansTest`, `Application/LendAndReturnTest` | Cumple |
| CA-07.2 devolver dos veces → 409, nunca suma dos veces | idem | `PgLoanRepository::saveReturn` | `Contract/LoansTest` | Cumple |
| CA-07.3 disponibles nunca superan el total | idem | `Book::restock`, `ck_book_available_range` | `Unit/BookTest`, `Integration/SchemaTest` | Cumple |
| CA-07.4 préstamo cerrado | `GET /loans/{id}` | `Loan::status` | `Unit/LoanTest`, `Contract/LoansTest` | Cumple |
| HU-08 · CA-08.1 detalle con estado | `GET /loans/{id}` | `GetLoanService` | `Contract/LoansTest` | Cumple |
| CA-08.2 listado por rango, estado y socio | `GET /loans` | `ListLoansService`, `PgLoanRepository::search` | `Contract/LoansTest` | Cumple |
| CA-08.3 estado calculado | `GET /loans` | `Loan::status` | `Unit/LoanTest`, `Integration/SchemaTest` (sin columna `status`) | Cumple |
| HU-09 · CA-09.1/09.2 vencidos con días de retraso | `GET /loans/overdue` | `OverdueService` | `Contract/LoansTest` | Cumple |
| HU-10 · CA-10.1 más prestados por libro y categoría congelada | `GET /reports/most-borrowed` | `PgReportReader` | `Contract/ReportTest` | Cumple |
| CA-10.2 rango invertido → 422; vacío → lista vacía | idem | `DateRange` | `Unit/ValueObjectsTest`, `Contract/ReportTest` | Cumple |
| CA-10.3 sin desglose por socio ni bibliotecario | idem | `PgReportReader` | `Contract/ReportTest` | Cumple |
| CA-10.4 el reporte no cambia al renombrar | idem | `PgReportReader` | `Contract/ReportTest` | Cumple |
| CA-10.5 solo admin | idem | `RequireRole` | `Contract/ReportTest` | Cumple |
| RNF-1 búsqueda < 300 ms con 5 000 libros | `GET /books` | `PgBookRepository::search` | `tool/seed/seed.mjs --phase=load`: p95 de 78 ms | Cumple |
| RNF-2 último ejemplar | `POST /loans` | CAS en `saveLending` (ADR-005) | `Concurrency/LastCopyTest` | Cumple |
| RNF-3 la clave no viaja ni se guarda en claro | `POST /users`, login | `Argon2idPasswordHasher` | `Contract/AuthTest` | Parcial: respuestas verificadas, logs sin auditar |
| RNF-4 un solo cuerpo de error | todos | `ErrorMapper` | `Contract/AuthTest` | Cumple |
| RNF-5 un solo comando, base vacía | — | `infra/docker-compose.yml` | `infra/README.md` | Cumple |

## Sin test automático

- Recorrido completo de la interfaz (prestar, devolver, reporte): se verificó a mano login, catálogo y vencidos; el resto depende de los tests de la API.
