# spec-laravel

Traducción del spec de Manager Library al stack del reto: **React** (frontend) y **PHP/Laravel** (backend) sobre PostgreSQL 16.

El negocio no cambia: historias, criterios de aceptación, reglas, modelo de datos y contrato HTTP son los de `spec-python/`.
Solo cambia lo que el spec define sobre tecnología.

## Cómo se leyó el spec

1. `constitution.md` — los 12 artículos y su verificación, con herramientas de PHP.
2. `spec.md` — qué debe hacer el sistema.
3. `plan.md`, `architecture.md`, `arquitectura-panoramica.md` — cómo se construye en Laravel.
4. `data-model.md` y `api-contract.md` — idénticos a los de Python y .NET.
5. `tasks.md` — T-01…T-23 con el estado real y enlace a la evidencia.
6. `adr/` — ADR-001…004 originales, sin tocar; ADR-005…007 son decisiones nuevas.

## Qué cambia frente a Python y .NET

| Tema | Python | .NET | Laravel |
|---|---|---|---|
| Framework HTTP | FastAPI | Minimal API | Rutas de `routes/api.php` + controladores |
| Persistencia y migraciones | SQLAlchemy + Alembic | EF Core | Migraciones de Laravel (SQL crudo donde hace falta) |
| Hash de claves | Argon2id | BCrypt | Argon2id |
| Pruebas | pytest | xUnit | Pest |
| Pruebas de arquitectura | import-linter | NetArchTest | Deptrac y Pest Arch |

## Documentos añadidos en esta carpeta

| Archivo | Para qué |
|---|---|
| [`decisiones-y-huecos.md`](decisiones-y-huecos.md) | Huecos del spec y la decisión tomada en cada uno |
| [`trazabilidad.md`](trazabilidad.md) | HU/CA → endpoint → clase → test → estado |
| [`adr/adr-005-testigo-de-concurrencia-sin-columna-version.md`](adr/adr-005-testigo-de-concurrencia-sin-columna-version.md) | Concurrencia sin columna de versión |
| [`adr/adr-006-reloj-desplazable-para-el-sembrador.md`](adr/adr-006-reloj-desplazable-para-el-sembrador.md) | Reloj desplazable solo para la demo |
| [`adr/adr-007-jwt-propio-en-vez-de-sanctum.md`](adr/adr-007-jwt-propio-en-vez-de-sanctum.md) | JWT propio |

El diseño de dominio está en [`../spec-php/ddd.md`](../spec-php/ddd.md).
