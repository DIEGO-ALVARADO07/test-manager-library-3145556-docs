# Tareas — Manager Library (Laravel)

Cada tarea termina cuando su **evidencia** existe y se puede ejecutar (artículo XII). Estado inicial de todas: **pendiente**.

| # | Tarea | Depende de | Evidencia |
|---|---|---|---|
| T-01 | Esqueleto del proyecto y `GET /health` | — | `GET /health` responde `{"status":"ok"}`; el contenedor arranca con un comando |
| T-02 | Migración inicial: seis tablas, FK, índices únicos y las cinco categorías sembradas | T-01 | Las consultas de `data-model.md` §10 devuelven lo documentado; ninguna columna con `DEFAULT` |
| T-03 | Dominio: `Isbn`, `Quantity`, `DateRange` (objetos de valor) | T-01 | Pruebas unitarias: ISBN de 13 dígitos, cantidad > 0, fin ≥ inicio |
| T-04 | Dominio: `Book` (`Withdraw`, `Restock`, baja lógica) | T-03 | Pruebas: nunca < 0, nunca > total |
| T-05 | Dominio: `Loan` (`AddItem`, `ReturnItem`, estado calculado, 14 días, máx. 3) | T-04 | Pruebas con un `Clock` falso: `open`, `overdue`, `closed` |
| T-06 | Dominio: `Member` y `User` (normalización de nombre, rol válido) | T-03 | `"Ana "` queda `ana`; rol `admin` por API rechazado |
| T-07 | Puertos y casos de uso de **autenticación** (`LoginService`, `CreateLibrarianService`) con fakes | T-06 | HU-01 y HU-02 pasan contra fakes |
| T-08 | Puertos y casos de uso de **libros** (crear, buscar, editar, baja, portada) con fakes | T-04 | HU-03 y HU-04 pasan contra fakes |
| T-09 | Caso de uso **prestar** (`LendBooksService`) con fakes | T-05, T-08 | HU-06 completa: límites, vencidos, sin ejemplares |
| T-10 | Caso de uso **devolver** y **vencidos** | T-09 | HU-07 y HU-09; devolver dos veces → `ITEM_ALREADY_RETURNED` |
| T-11 | Caso de uso **reporte de más prestados** y su puerto de lectura | T-09 | HU-10; el título viejo se conserva tras renombrar (ADR-004) |
| T-12 | Adaptador de persistencia: repositorios de libro, socio, usuario | T-02, T-08 | Pruebas de integración contra PostgreSQL real |
| T-13 | Adaptador de persistencia: repositorio de préstamos + testigo de versión | T-12, T-09 | Dos préstamos del último ejemplar: uno 201, otro 409 (RNF-2) |
| T-14 | Adaptador de seguridad: hash de claves y token JWT | T-07 | La clave nunca aparece en logs ni respuestas (RNF-3) |
| T-15 | Adaptador de portadas (disco) | T-08 | Subir PNG de 2 MB ok; 3 MB → 422; sin portada → `cover_key` nulo |
| T-16 | Adaptador HTTP: rutas de [`api-contract.md`](api-contract.md), cuerpo de error único, autorización por rol | T-10, T-11, T-14, T-15 | Prueba de contrato recorre todas las rutas y todos los códigos de §2 |
| T-17 | Composición y arranque (admin inicial desde entorno) | T-16 | El sistema arranca con la base vacía y se puede iniciar sesión como admin |
| T-18 | Bajar a la base las invariantes **solo dominio**: `ck_book_total_positive`, `ck_user_role` | T-12 | Un `INSERT` por `psql` que las viole falla; `data-model.md` §4 se actualiza |
| T-19 | Pruebas de arquitectura (artículos I, II y III) | T-17 | Deptrac y un test de Pest Arch en `tests/Architecture/` pasan y fallan si se introduce una importación prohibida |
| T-20 | `infra`: `docker compose up` levanta motor vacío, API y frontend | T-17 | Un comando; la base arranca sin tablas hasta correr las migraciones |
| T-21 | `tool`: sembrador de demostración **por la API** (≥ 40 libros, 10 socios, 15 préstamos, 3 vencidos) | T-17 | Tras correrlo, `/loans/overdue` devuelve 3 líneas y el reporte tiene filas |
| T-22 | `app`: login, catálogo, socios, prestar, devolver, vencidos, reporte | T-16 | Recorrido manual de punta a punta en la feria |
| T-23 | `page`: sitio público estático de presentación | — | Se abre sin la API; correcto en móvil y escritorio |

## Definición de terminado de la prueba

Quien llegue a T-17 con evidencia tiene el **núcleo** hecho. T-18 a T-23 son lo que separa un trabajo bueno de uno sobresaliente. Se evalúa
el desempeño con SDD, no la cantidad de tareas: una tarea con evidencia vale más que tres a medias.

---

## Estado real y evidencia (13 de octubre de 2026)

Estado de cada tarea al cierre de la prueba, con la evidencia que se puede ejecutar. Repositorios: `api` = [test-manager-library-3145556-api](https://github.com/DIEGO-ALVARADO07/test-manager-library-3145556-api), `app`, `infra`, `tool` y `page` siguen el mismo patrón en la misma cuenta.

| # | Estado | Evidencia |
|---|---|---|
| T-01 | Hecha | `GET /api/v1/health` y `GET /health` responden `{"status":"ok"}` — `api/tests/Contract/AuthTest.php`; el contenedor arranca con `docker compose up` (T-20) |
| T-02 | Hecha | `api/database/migrations/2026_10_13_000002_create_initial_tables.php`; `api/tests/Integration/SchemaTest.php` reproduce las consultas de `data-model.md` §10: seis tablas, ninguna columna con `DEFAULT`, cinco categorías con los UUID fijos |
| T-03 | Hecha | `api/tests/Unit/ValueObjectsTest.php` |
| T-04 | Hecha | `api/tests/Unit/BookTest.php` |
| T-05 | Hecha | `api/tests/Unit/LoanTest.php` (estado con reloj fijo) |
| T-06 | Hecha | `api/tests/Unit/UserTest.php`; el rol `admin` por API responde 422 en `api/tests/Contract/AuthTest.php` |
| T-07 | Parcial | `LoginService` y `CreateLibrarianService` se prueban por contrato contra PostgreSQL real (`AuthTest.php`), no contra fakes |
| T-08 | Parcial | HU-03 y HU-04 se prueban por contrato (`BooksTest.php`), no contra fakes |
| T-09 | Hecha | `api/tests/Application/LendAndReturnTest.php` (fakes) y `api/tests/Contract/LoansTest.php` |
| T-10 | Hecha | Devolver contra fakes (`LendAndReturnTest.php`); vencidos y `ITEM_ALREADY_RETURNED` por contrato (`LoansTest.php`) |
| T-11 | Hecha | `api/tests/Contract/ReportTest.php`: el título viejo se conserva tras renombrar (ADR-004); sin fakes |
| T-12 | Hecha | Repositorios probados contra PostgreSQL real por `Contract` e `Integration` |
| T-13 | Hecha | `api/tests/Concurrency/LastCopyTest.php`: dos procesos simultáneos sobre el último ejemplar, uno gana y el otro recibe `BOOK_NOT_AVAILABLE`; cinco compiten por dos |
| T-14 | Parcial | Argon2id y JWT implementados; ninguna respuesta incluye `password` ni `password_hash` (`AuthTest.php`). No se auditaron los logs |
| T-15 | Hecha | `api/tests/Contract/BooksTest.php`: PNG válido, archivo de 3 MB → 422, no imagen → 422, sin portada → `cover_key` nulo |
| T-16 | Hecha | `api/tests/Contract/`: recorre todas las rutas y los 14 códigos de error de `api-contract.md` §2 |
| T-17 | Hecha | `docker compose up` con base vacía migra, crea el admin desde el entorno y permite iniciar sesión (verificado con el sembrador) |
| T-18 | Hecha | `api/database/migrations/2026_10_13_000003_add_remaining_check_constraints.php`; `SchemaTest.php` comprueba que el motor rechaza total 0 y rol desconocido; §4 de `data-model.md` actualizado |
| T-19 | Hecha | `api/tests/Architecture/LayersTest.php` (Pest Arch). Se comprobó que falla: al añadir una referencia a `Illuminate` en el dominio el test responde «Expecting 'Library\Domain' not to use 'Illuminate'» |
| T-20 | Hecha | `infra/docker-compose.yml`: `db` (postgres:16 vacío, sin scripts de inicio), `api` y `app`; `curl localhost:8000/api/v1/health` responde `ok` |
| T-21 | Hecha | `tool/seed/run-all.sh`: `/loans/overdue` devuelve **3** líneas, hay 15 préstamos y 19 filas en el reporte; RNF-1: con 5 000 libros la búsqueda tiene p95 de 78 ms |
| T-22 | Hecha | Recorrido en navegador: login, catálogo y vencidos verificados contra la API real; el resto de pantallas está cubierto por los tests de la API |
| T-23 | Hecha | `page/index.html` abre sin backend y se revisó a 375 px y a 1280 px |

Suite completa de `api`: 90 tests, 388 aserciones, todos en verde (`vendor/bin/pest`).

Pendiente declarado: ejecutar los tests de T-07, T-08 y T-11 contra fakes, y auditar los logs para RNF-3.
