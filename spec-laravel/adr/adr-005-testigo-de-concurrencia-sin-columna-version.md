# ADR-005 · Testigo de concurrencia sin columna de versión

- **Estado:** aceptada
- **Contexto:** [ADR-002](adr-002-concurrencia-optimista.md) y la decisión D-04 hablan de un «testigo de versión por libro», pero
  `data-model.md` §3 define `book` sin ninguna columna de versión y declara que ese documento manda (Art. X).
- **Incongruencia detectada:** ADR-002 exige un mecanismo que el modelo de datos no contiene. No se corrige ADR-002 ni el modelo.
- **Decisión:** el valor leído de `available_copies` actúa como testigo (compare-and-set). Dentro de la transacción, cada
  descuento se escribe como `UPDATE book SET available_copies = available_copies - 1 WHERE id = ? AND available_copies = ? AND deleted_at IS NULL`.
  Si afecta 0 filas, se deshace todo y se responde 409 `BOOK_NOT_AVAILABLE`. La última barrera sigue siendo `ck_book_available_range`.
- **Cómo se podría corregir a futuro:** añadir una columna `version` a `book`, actualizar `data-model.md` §3 (Art. X) y comparar
  esa versión en el `WHERE`. Se descarta ahora porque cambiaría el modelo de datos.
- **Alternativas descartadas:** `SELECT ... FOR UPDATE` (ADR-002 lo descartó); leer `xmin` de Postgres (depende de una columna de sistema).
- **Consecuencias:** un cambio de `available_copies` que termine en el mismo valor (devolver y volver a prestar entre la lectura y la
  escritura) no se detecta, y es inocuo: el contador sigue siendo correcto y nunca sale del rango.
- **Cómo se verifica:** prueba de concurrencia con dos peticiones simultáneas sobre un libro con un ejemplar; exactamente una recibe 201.
