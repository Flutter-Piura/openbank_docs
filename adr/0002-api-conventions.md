# ADR-0002: convenciones de la API

- Estado: aceptado
- Fecha: 2026-08-30
- Responsables: mantenedores de OpenBank / Flutter Piura

## Contexto

Flutter, MobileLab y la API real necesitan interpretar dinero, tiempo, identidad, errores y reintentos de forma idéntica. Las convenciones ambiguas pueden provocar pérdida de precisión, duplicación de transferencias o incompatibilidad entre repositorios.

## Decisión

### Especificación

- OpenAPI 3.0.3 autocontenido será la fuente de verdad inicial.
- La versión del contrato sigue SemVer dentro de `info.version` y tags Git.
- Redocly valida y empaqueta el contrato.
- oasdiff detecta cambios incompatibles respecto de `main`.

Se eligió 3.0.3 para maximizar compatibilidad inicial con MobileLab y generadores. Una migración a 3.1 requiere ADR y pruebas de todos los consumidores.

### Dinero

```json
{
  "minorUnits": 1050,
  "currency": "PEN"
}
```

- `minorUnits` es entero de 64 bits.
- `currency` usa ISO 4217 en mayúsculas.
- No se utiliza `double`, `float` ni decimal serializado sin escala explícita.
- Los importes de movimiento usan signo; las solicitudes de transferencia exigen importe positivo.

### Tiempo

- Timestamps con `format: date-time` se serializan en ISO 8601 y UTC.
- Los clientes convierten a zona local únicamente para presentar.
- Fechas de negocio sin hora usarán `format: date` cuando aparezcan.

### Identificadores

- IDs de recursos son strings opacos; inicialmente se expresan como UUID.
- El consumidor no deriva tipo, orden o fecha desde un ID.
- Cursores también son opacos y tienen longitud limitada.

### Paginación

- Las colecciones grandes utilizan cursor.
- La respuesta contiene `items` y `pagination.nextCursor`.
- Un cursor nulo indica fin de la colección.
- El tamaño de página está limitado por el contrato.

### Idempotencia

- Crear una transferencia exige el header `Idempotency-Key`.
- La misma clave y el mismo payload devuelven el resultado original.
- La misma clave con un payload diferente devuelve HTTP 409.
- Las claves tienen alcance por identidad y operación según la implementación.

### Errores

Los errores usan `application/problem+json` con:

```json
{
  "code": "insufficient_funds",
  "message": "La cuenta no tiene saldo disponible suficiente.",
  "correlationId": "01J6KVQZEF6SVPZJMWB2X7PZ9F",
  "details": null
}
```

- `code` es estable y apto para lógica del cliente.
- `message` es seguro y presentable; no contiene stack traces.
- `correlationId` permite investigar sin exponer información sensible.
- `details` es opcional y no puede contener secretos.

### Compatibilidad

- Añadir campos opcionales en respuestas es compatible si los clientes ignoran desconocidos.
- Quitar campos, hacerlos requeridos o restringir valores aceptados puede ser incompatible.
- Los estados de negocio se modelan inicialmente como strings con valores conocidos documentados, permitiendo fallback del cliente.
- Un cambio incompatible debe coexistir, versionarse o ser aprobado mediante un ADR de migración.

## Consecuencias

- Los clientes implementan un value object Money y mapeadores explícitos.
- Los retries de transferencia son seguros si conservan la clave.
- Las pruebas pueden compartir fixtures deterministas.
- El contrato añade disciplina y herramientas, pero reduce ambigüedad entre repositorios.

## Verificación

- Redocly lint y bundle pasan en CI.
- oasdiff bloquea cambios `ERR` en PR posteriores al contrato inicial.
- MobileLab importa el bundle sin referencias externas.
- Flutter y API prueban dinero, fechas, paginación, errores e idempotencia contra ejemplos del contrato.
