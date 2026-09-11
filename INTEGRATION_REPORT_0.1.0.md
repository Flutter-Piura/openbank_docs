# Informe de integración local 0.1.0

- Fecha de validación: 2026-09-10
- Organización: [Flutter Piura](https://github.com/Flutter-Piura)
- Estado: recorrido vertical integrado y reproducible

## Resultado

OpenBank dispone de un recorrido bancario educativo funcional sin infraestructura cloud. La misma aplicación Flutter puede usar MobileLab en el puerto 4566 o la API NestJS con PostgreSQL en el puerto 3000 cambiando únicamente `OPENBANK_API_URL`.

El recorrido validado inicia sesión con una identidad ficticia, consulta perfil, cuentas y movimientos, crea una transferencia interna, persiste un débito y un crédito balanceados, reutiliza de forma segura la respuesta al repetir la misma clave de idempotencia y cierra la sesión.

## Componentes integrados

| Repositorio | Entregable 0.1.0 | Validación principal |
| --- | --- | --- |
| `openbank_contracts` | OpenAPI 3.0.3, ejemplos, lint, detección de breaking changes y cliente Dart generable | `contract-quality`, `breaking-changes` |
| `openbank_mobile` | Flutter con Clean Architecture, login, dashboard, movimientos y transferencias | analyze, 7 tests, Android/iOS build y smoke MobileLab/API |
| `openbank_api` | NestJS, JWT, PostgreSQL, ledger y transferencias atómicas/idempotentes | 8 unit tests, E2E PostgreSQL, audit y Docker build |
| `openbank_infrastructure` | Compose para PostgreSQL, migraciones y API | health checks, migración controlada y smoke test |
| `openbank_docs` | Arquitectura, producto, dominio, compatibilidad y este informe | revisión documental en CI |

## Arquitectura ejecutada

```text
openbank_mobile
  ├── http://127.0.0.1:4566 → MobileLab (sandbox determinista)
  └── http://127.0.0.1:3000 → openbank_api → PostgreSQL 17

openbank_contracts 0.1.x → contrato común de ambos caminos
openbank_infrastructure   → construcción, migración y health checks
openbank_docs             → decisiones y compatibilidad transversal
```

Flutter y la API organizan sus capacidades con dependencias hacia adentro. Los modelos de dominio no dependen de Flutter, NestJS, HTTP ni PostgreSQL. Los adaptadores externos se conectan en la composición de cada aplicación.

## Reglas bancarias demostradas

- Los importes usan enteros en unidades menores y moneda explícita.
- Origen y destino deben ser cuentas diferentes, activas y de la misma moneda.
- La API bloquea las cuentas involucradas dentro de una transacción PostgreSQL.
- El saldo se deriva del ledger y no se modifica directamente.
- Cada transferencia completada crea un débito y un crédito con igual magnitud.
- El ledger rechaza actualizaciones y eliminaciones mediante un trigger.
- `Idempotency-Key` es un UUID y su relación con la solicitud se persiste.
- Repetir clave y payload devuelve la transferencia original; cambiar el payload produce HTTP 409.
- Los errores públicos usan código estable, mensaje seguro y correlation ID.

## Ejecución desde cero

Los repositorios deben ser hermanos dentro del mismo workspace:

```text
Flutter_Piura/
├── openbank_api/
├── openbank_contracts/
├── openbank_docs/
├── openbank_infrastructure/
└── openbank_mobile/
```

Levanta el backend persistente:

```bash
cd openbank_infrastructure
cp .env.example .env
docker compose up --build --wait
./scripts/smoke.sh
```

Ejecuta Flutter en iOS Simulator o escritorio:

```bash
cd ../openbank_mobile
flutter run --dart-define=OPENBANK_API_URL=http://127.0.0.1:3000
```

Android Emulator debe usar `http://10.0.2.2:3000`. Las credenciales son `demo@openbank.local` y `OpenBankDemo!2026`; son datos públicos y exclusivamente ficticios.

## Evidencia local y CI

La integración fue validada con:

```text
API:             format + lint + build + 8 unit tests + 1 E2E + npm audit (0)
Contenedores:    PostgreSQL healthy + migraciones 001/002 + API healthy
Infraestructura: docker compose config + build + wait + smoke
Flutter:         analyze + 7 tests + smoke contra http://127.0.0.1:3000
Contrato:        lint + bundle + generación/verificación del paquete Dart
```

El E2E del API cubre acceso sin token, login, perfil, cuentas, paginación, transferencia, replay idempotente, conflicto, consulta, rotación de refresh token y logout.

## Alcance de este hito

`0.1.0` es un vertical slice local integrado, no el release público completo descrito en las fases 8–10 del plan maestro. Permanecen en el roadmap, entre otros:

- restauración y almacenamiento seguro de sesión en Flutter;
- consumo de refresh token desde el cliente móvil;
- detalle visual de movimiento y comprobante enriquecido;
- rate limiting y política CORS por ambiente;
- validación automatizada de respuestas contra OpenAPI;
- threat model, SBOM, staging y publicación en stores.

No se ha desplegado ningún servidor cloud. OpenBank no procesa dinero ni datos bancarios reales.
