# ADR-007 · JWT propio en vez de Sanctum

- **Estado:** aceptada
- **Contexto:** el contrato pide un JWT firmado (HS256, 8 horas, `sub` y `role`, sin datos personales). Sanctum emite tokens opacos y
  crea la tabla `personal_access_tokens`, que rompe el modelo de «seis tablas» (`data-model.md` §2).
- **Decisión:** el puerto `TokenIssuer` se implementa con `firebase/php-jwt`, algoritmo fijo HS256, clave en `JWT_SECRET`. Un
  middleware propio valida el token y otro comprueba el rol. No se instala Sanctum ni Passport.
- **Alternativas descartadas:** Sanctum (token opaco y tabla extra); Passport (OAuth2 completo, fuera de alcance).
- **Consecuencias:** sin estado en servidor; un token no se puede revocar antes de vencer, igual que en las otras versiones del spec.
- **Cómo se verifica:** el token decodificado solo trae `sub`, `role`, `iat` y `exp`; sin token o con token alterado responde 401.
