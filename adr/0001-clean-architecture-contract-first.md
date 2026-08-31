# ADR-0001: Clean Architecture, contrato primero y monolito modular

- Estado: aceptado
- Fecha: 2026-08-30
- Responsables: mantenedores de OpenBank / Flutter Piura

## Contexto

OpenBank comienza como un proyecto multirepo con una aplicación Flutter, una API, contratos compartidos, infraestructura y documentación. El equipo necesita desarrollar un MVP funcional localmente, permitir contribuciones independientes y evitar que el cliente móvil quede bloqueado mientras se implementa el backend.

El dominio incluye dinero, cuentas, movimientos y transferencias. Estas capacidades requieren reglas explícitas, importes exactos, idempotencia y una autoridad central para confirmar operaciones. Un conjunto de pantallas conectadas directamente a mocks o persistencia no ofrece límites suficientes.

Al mismo tiempo, dividir prematuramente la API en microservicios elevaría el costo de desarrollo, despliegue, observabilidad y coordinación entre repositorios.

## Decisión

### Arquitectura transversal

- Adoptar Clean Architecture con dependencias dirigidas hacia el dominio.
- Organizar el código por feature o módulo de negocio y luego por capa.
- Usar `openbank_contracts` como fuente de verdad para HTTP mediante OpenAPI.
- Mantener DTO, framework, persistencia y transporte fuera del dominio.
- Registrar decisiones transversales posteriores mediante ADR.

### Aplicación Flutter

Cada feature tendrá, según su necesidad, las capas:

```text
domain → application ← infrastructure
                    ← presentation
```

- `domain`: entidades, value objects y puertos en Dart puro.
- `application`: casos de uso y coordinación.
- `infrastructure`: HTTP, caché, DTO, mapeadores e implementaciones de puertos.
- `presentation`: estado, navegación, páginas y widgets.

Los modelos generados desde OpenAPI se mapearán a modelos de dominio y no llegarán directamente a la interfaz.

### API

Implementar `openbank_api` inicialmente como un monolito modular con NestJS y TypeScript. Cada módulo tendrá dominio, aplicación, infraestructura e interfaces. PostgreSQL será la base de datos de integración local y del futuro despliegue.

Los módulos iniciales serán:

- identity
- customers
- accounts
- ledger
- transactions
- transfers

La API será autoritativa sobre saldos y transferencias. El ledger utilizará asientos inmutables y balanceados, y las transferencias serán atómicas e idempotentes.

### Desarrollo sin cloud

Usar MobileLab como sandbox local para mocks, fixtures, autenticación ficticia, importación OpenAPI y escenarios de fallos. MobileLab no será el backend productivo ni implementará las invariantes definitivas del ledger.

Usar Docker Compose para ejecutar localmente `openbank_api` y PostgreSQL cuando comience la implementación real del backend. No se requiere un servidor cloud para el MVP local.

## Consecuencias positivas

- Flutter puede avanzar contra un contrato estable y un sandbox reproducible.
- Las reglas de negocio se prueban sin depender de Flutter, NestJS o PostgreSQL.
- La API conserva simplicidad operativa mientras mantiene módulos extraíbles.
- Los adaptadores pueden cambiar sin reescribir casos de uso.
- El sistema puede probarse localmente y en CI antes de pagar infraestructura.

## Costos y riesgos aceptados

- Habrá mapeo explícito entre DTO y dominio.
- Los cinco repositorios requieren coordinación de versiones.
- Clean Architecture agrega estructura que debe justificarse con límites y pruebas reales.
- MobileLab y la API real deben someterse a pruebas de contrato para evitar divergencias.
- NestJS no debe filtrarse dentro de las entidades o reglas del dominio.

## Reglas verificables

- Los paquetes de dominio no importan Flutter, NestJS, clientes HTTP ni drivers de base de datos.
- Los importes no utilizan punto flotante.
- Las pantallas no calculan ni confirman saldos o transferencias.
- Los cambios HTTP comienzan en `openbank_contracts`.
- No se extraerá un microservicio sin un ADR que demuestre una necesidad operativa o de propiedad clara.
- El mismo recorrido móvil deberá funcionar contra MobileLab y contra la API local.

## Cuándo revisar esta decisión

Revisar este ADR si aparece alguna de estas condiciones:

- Un módulo necesita despliegue o escalado independiente demostrado.
- El stack elegido impide cumplir requisitos funcionales o de contribución.
- La generación OpenAPI introduce acoplamiento mayor que el beneficio obtenido.
- Se decide operar con datos o integraciones financieras reales, lo que exigiría un proyecto de seguridad y cumplimiento diferente.
