# ADR-006 · Reloj desplazable solo para el sembrador de demostración

- **Estado:** aceptada
- **Contexto:** T-21 exige que el sembrador deje 3 préstamos vencidos y que lo haga **por la API**. HU-06 fecha el préstamo con el
  reloj del servidor y la API no permite indicar `loaned_at`. Con solo la API es imposible crear un préstamo ya vencido.
- **Incongruencia detectada:** T-21 y HU-06/CA-06.6 no pueden cumplirse a la vez sin ayuda del reloj. No se modifican.
- **Decisión:** el adaptador del puerto `Clock` lee la variable de entorno `CLOCK_OFFSET_DAYS` (entero, **0 por defecto**). El
  sembrador levanta una fase con la API desplazada hacia el pasado, crea los 3 préstamos y vuelve a la fase normal. El contrato HTTP
  no cambia y no se acepta ninguna fecha desde el cliente.
- **Cómo se podría corregir a futuro:** que el spec permita un parámetro de fecha solo para `admin`, o que T-21 acepte SQL; ambos
  cambian el alcance y no se aplican.
- **Alternativas descartadas:** cabecera HTTP que mueva el reloj (abre un agujero de seguridad); SQL directo (prohibido por el spec).
- **Consecuencias:** `CLOCK_OFFSET_DAYS` solo se activa en la demo y nunca va en `.env.example` con un valor distinto de 0.
- **Cómo se verifica:** tras sembrar, `GET /loans/overdue` devuelve 3 líneas y con el reloj normal sigue devolviéndolas.
