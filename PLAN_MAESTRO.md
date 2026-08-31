# OpenBank — Plan maestro de implementación

> Estado: aprobado para ejecución el 2026-08-30. El avance se realizará por fases, validando los criterios de salida y reportando cada hito.

## 1. Visión

OpenBank será una plataforma bancaria educativa y open source mantenida por la comunidad Flutter Piura. Permitirá practicar arquitectura de software, Flutter, APIs, contratos, automatización y seguridad usando exclusivamente usuarios, cuentas y dinero ficticios.

El resultado inicial será una aplicación móvil funcional que pueda ejecutarse completamente en un entorno local. No se necesita contratar ni configurar un servidor cloud para desarrollar el MVP. El diseño permitirá incorporar un ambiente público de demostración y posteriormente un backend desplegado sin reescribir el dominio móvil ni romper el contrato de la API.

OpenBank no será un banco real, no procesará dinero, no se conectará con instituciones financieras y no almacenará información financiera real. Todas las interfaces y documentos deberán comunicar claramente esta condición.

## 2. Objetivos

### Objetivos del MVP

- Inicio y cierre de sesión con identidades ficticias.
- Consulta de cuentas y saldos ficticios.
- Historial y detalle de movimientos.
- Transferencias internas simuladas.
- Comprobantes de transferencia.
- Manejo visible de carga, vacío, error, pérdida de conexión y sesión expirada.
- Ejecución local reproducible para nuevos contribuidores.
- Pruebas automáticas en cada repositorio.
- Contrato OpenAPI como fuente de verdad de la comunicación cliente–servidor.
- Arquitectura limpia, límites verificables y documentación suficiente para contribuir.

### Fuera del alcance inicial

- Dinero o cuentas bancarias reales.
- Conexiones con bancos, tarjetas o pasarelas de pago.
- KYC real, biometría de identidad o documentos personales.
- Criptomonedas, inversiones o créditos reales.
- Microservicios distribuidos.
- Kubernetes.
- Publicación inmediata en App Store o Play Store.
- Cumplimiento regulatorio para operación financiera real.

## 3. Repositorios y responsabilidades

La carpeta `Flutter_Piura` es únicamente un workspace local. No tendrá un repositorio Git propio. Cada subcarpeta se conectará a su repositorio independiente dentro de la organización.

| Repositorio | Responsabilidad | Artefacto versionado |
| --- | --- | --- |
| `openbank_mobile` | Aplicación Flutter para Android e iOS | Aplicación y builds de prueba |
| `openbank_api` | Backend local y futuro backend desplegable | API ejecutable e imagen de contenedor |
| `openbank_contracts` | OpenAPI, esquemas, ejemplos y política de compatibilidad | Contrato y cliente Dart generado |
| `openbank_infrastructure` | Entorno local, contenedores, configuración y despliegues | Docker Compose y definiciones de despliegue |
| `openbank_docs` | Visión, arquitectura transversal, ADR, roadmap y guías | Documentación del sistema |

No habrá imports mediante rutas relativas entre repositorios. La integración ocurrirá mediante versiones publicadas, generación desde OpenAPI, imágenes de contenedor y configuración explícita.

### Estado local observado al crear este plan

- Workspace real: `/Users/gsandovt/projects/FLUTTER/Flutter_Piura`.
- `openbank_mobile` contiene el scaffold inicial generado por Flutter, todavía con la aplicación contador.
- `openbank_api`, `openbank_contracts`, `openbank_infrastructure` y `openbank_docs` estaban vacíos antes de crear este documento.
- Ninguna de las cinco carpetas locales contiene todavía metadatos `.git`.
- El contenido e historial de los repositorios remotos no se asumirán: se compararán de forma no destructiva en la Fase 0 antes de conectar o subir archivos.

## 4. Principios de arquitectura

1. **Dependencias hacia adentro:** presentación e infraestructura dependen de abstracciones de aplicación y dominio; el dominio no conoce Flutter, HTTP, bases de datos ni proveedores.
2. **Feature first:** el código se organiza por capacidad de negocio y después por capa, evitando carpetas globales que crecen sin límites.
3. **Contrato primero:** cualquier endpoint nace o cambia en `openbank_contracts` antes de implementarse en cliente y servidor.
4. **Backend autoritativo:** saldos, transferencias e invariantes se validan en la API. Los mocks nunca se consideran una implementación bancaria real.
5. **Monolito modular primero:** `openbank_api` se despliega como una unidad con módulos aislados. Solo se extraerá un servicio cuando exista una necesidad demostrable.
6. **Offline no significa doble gasto:** la app puede cachear consultas, pero una transferencia requiere confirmación del backend y no se marca exitosa localmente antes de recibirla.
7. **Dinero exacto:** nunca se representarán importes con `double`/punto flotante. El contrato utilizará unidades menores enteras, por ejemplo `1050` para S/ 10.50, junto con el código de moneda.
8. **Fechas inequívocas:** timestamps en UTC con ISO 8601; la interfaz los presenta en la zona horaria del usuario.
9. **Idempotencia:** las operaciones de transferencia usarán una clave de idempotencia para evitar duplicados por reintentos.
10. **Seguridad por defecto:** secretos fuera de Git, logs sanitizados, permisos mínimos y datos exclusivamente ficticios.
11. **Observabilidad desde el MVP:** errores correlacionables, logs estructurados y health checks, sin registrar tokens ni datos sensibles.
12. **Decisiones registradas:** toda decisión transversal significativa se documentará como ADR en `openbank_docs`.

## 5. Arquitectura del sistema

```text
┌──────────────────────┐
│   openbank_mobile    │
│ Flutter / Clean Arch │
└──────────┬───────────┘
           │ HTTPS/JSON según OpenAPI
           ▼
┌──────────────────────┐       ┌──────────────────────┐
│     openbank_api     │──────▶│     PostgreSQL       │
│ Monolito modular     │       │ entorno local/cloud  │
└──────────┬───────────┘       └──────────────────────┘
           │
           ├── contrato: openbank_contracts
           ├── ejecución: openbank_infrastructure
           └── decisiones: openbank_docs

Durante desarrollo y CI:

openbank_mobile ──▶ MobileLab ──▶ fixtures, SQLite y escenarios locales
```

Habrá dos adaptadores de datos intercambiables en Flutter:

- `Api*Repository`: consume `openbank_api` o el sandbox HTTP de MobileLab.
- `Fake*Repository`: reservado para pruebas unitarias y previews; no será el modo normal del MVP.

Ambos implementarán los puertos definidos por el dominio. Cambiar el origen de datos no modificará casos de uso ni pantallas.

## 6. Clean Architecture en Flutter

Estructura objetivo de `openbank_mobile`:

```text
lib/
├── app/
│   ├── app.dart
│   ├── bootstrap.dart
│   ├── router/
│   ├── theme/
│   └── di/
├── core/
│   ├── error/
│   ├── network/
│   ├── result/
│   ├── security/
│   └── utils/
├── shared/
│   ├── domain/
│   └── presentation/
└── features/
    ├── authentication/
    ├── accounts/
    ├── transactions/
    ├── transfers/
    └── profile/
        ├── domain/
        │   ├── entities/
        │   ├── value_objects/
        │   └── repositories/
        ├── application/
        │   └── use_cases/
        ├── infrastructure/
        │   ├── data_sources/
        │   ├── dtos/
        │   └── repositories/
        └── presentation/
            ├── controllers/
            ├── pages/
            └── widgets/
```

Reglas:

- `domain` usa Dart puro y no importa Flutter, HTTP o serialización.
- `application` coordina casos de uso y depende de puertos de dominio.
- `infrastructure` implementa repositorios, caché, DTO y clientes externos.
- `presentation` renderiza estado y dispara casos de uso; no contiene reglas bancarias.
- Los DTO generados desde OpenAPI no se propagan directamente a las pantallas: se mapean a entidades de dominio.
- Las dependencias se inyectan desde `app/di`.
- Cada feature tendrá pruebas de dominio, aplicación, infraestructura y presentación proporcionales a su riesgo.

El gestor de estado, navegación, cliente HTTP, almacenamiento seguro y generadores se decidirán en ADR antes de incorporarlos. La selección priorizará mantenimiento, testabilidad y adopción en la comunidad; el dominio no dependerá de esas herramientas.

## 7. Clean Architecture en la API

`openbank_api` será un monolito modular, no una colección inicial de microservicios:

```text
src/
├── app/
│   ├── bootstrap/
│   └── configuration/
├── shared/
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── interfaces/
└── modules/
    ├── identity/
    ├── customers/
    ├── accounts/
    ├── ledger/
    ├── transfers/
    └── transactions/
        ├── domain/
        ├── application/
        ├── infrastructure/
        └── interfaces/http/
```

Responsabilidades por capa:

- **Domain:** entidades, value objects, reglas e invariantes.
- **Application:** comandos, consultas, casos de uso y puertos.
- **Infrastructure:** persistencia, tokens, reloj, identificadores y adaptadores externos.
- **Interfaces:** controladores HTTP, validación de entrada y mapeo de respuestas.

La primera implementación propuesta es NestJS con TypeScript y PostgreSQL local mediante Docker. NestJS actuará en las capas externas; el dominio permanecerá desacoplado del framework. Antes de implementarlo se registrará el ADR correspondiente.

El ledger se diseñará con asientos inmutables y balanceados. Una transferencia interna se ejecutará dentro de una transacción de base de datos:

```text
Transferencia S/ 100.00
  débito  cuenta origen  -10000 PEN
  crédito cuenta destino +10000 PEN
```

El saldo consultable podrá derivarse o proyectarse desde los asientos; no se permitirá modificar un saldo arbitrariamente desde la interfaz.

## 8. Contratos y compatibilidad

`openbank_contracts` será la fuente de verdad e incluirá:

```text
openapi/
├── openapi.yaml
├── paths/
├── schemas/
└── examples/
events/
json-schema/
scripts/
tests/
CHANGELOG.md
```

Reglas del contrato:

- OpenAPI 3.x válido y empaquetable en un documento autocontenido.
- Ejemplos deterministas y sin datos reales.
- Errores con un formato común: código estable, mensaje seguro, detalles permitidos y correlation ID.
- Paginación consistente en movimientos.
- Importes como entero de unidades menores más moneda ISO 4217.
- IDs opacos; la app no infiere significado desde ellos.
- Cambios incompatibles requieren una versión mayor del contrato o un endpoint versionado.
- CI detectará cambios incompatibles antes del merge.
- El cliente Dart se generará desde una versión/tag del contrato, no copiando modelos manualmente.

Primeros endpoints previstos:

```text
POST /v1/auth/login
POST /v1/auth/refresh
POST /v1/auth/logout
GET  /v1/me
GET  /v1/accounts
GET  /v1/accounts/{accountId}
GET  /v1/accounts/{accountId}/transactions
GET  /v1/transactions/{transactionId}
POST /v1/transfers
GET  /v1/transfers/{transferId}
GET  /health
```

## 9. Papel de MobileLab

[MobileLab](https://github.com/GianSandoval5/MobileLab) se integrará como herramienta de desarrollo y pruebas, no como backend de producción. Su API local, fixtures, SQLite, autenticación ficticia, importación OpenAPI, latencia y fallos controlados permiten desarrollar la app sin una cuenta cloud y validar casos difíciles de forma repetible.

Usos previstos:

- Levantar respuestas locales a partir de `openbank_contracts`.
- Desarrollar pantallas antes de completar un endpoint real.
- Simular sesión expirada, HTTP 500, respuestas lentas y pérdida de conectividad.
- Mantener fixtures reproducibles para cuentas, movimientos y perfiles.
- Ejecutar escenarios en CI y producir reportes JUnit/HTML.
- Comprobar que Flutter usa una URL correcta en Android Emulator e iOS Simulator.

Límites asumidos:

- No sustituye la lógica autoritativa de transferencias ni el ledger.
- Su autenticación es solo de desarrollo.
- Los mocks y la base local contienen datos ficticios.
- El contrato debe funcionar tanto con MobileLab como con `openbank_api`.

Por lo tanto, no necesitas configurar un servidor cloud ahora. Se ejecutarán localmente MobileLab y, a partir de la fase correspondiente, `openbank_api` con PostgreSQL. El despliegue remoto se decidirá en una fase posterior.

## 10. Modelo de dominio inicial

Entidades y value objects principales:

- `Customer`: identidad ficticia y estado del perfil.
- `Account`: cuenta, moneda, alias y estado.
- `Money`: unidades menores enteras y moneda.
- `LedgerEntry`: débito o crédito inmutable.
- `Transaction`: movimiento visible asociado a una cuenta.
- `Transfer`: intención y resultado de mover dinero entre dos cuentas.
- `TransferStatus`: pending, completed, rejected o failed.
- `IdempotencyKey`: evita procesar dos veces una misma solicitud.

Invariantes iniciales:

- El importe de una transferencia debe ser positivo.
- Origen y destino deben ser diferentes.
- Ambas cuentas deben estar activas y ser compatibles con la operación.
- La moneda debe ser explícita.
- Débitos y créditos de una transferencia deben balancear exactamente.
- Una misma clave de idempotencia no crea dos transferencias.
- Una operación completada no se reescribe; se compensa con nuevos asientos cuando corresponda.
- El cliente nunca decide por sí solo que una transferencia quedó completada.

## 11. Estrategia de entornos

| Entorno | Fuente de datos | Propósito |
| --- | --- | --- |
| Unit test | Fakes en memoria | Probar dominio y casos de uso rápidamente |
| Sandbox | MobileLab | UI, contrato, errores y escenarios reproducibles |
| Local integration | `openbank_api` + PostgreSQL | Probar reglas reales y persistencia |
| CI | Sandbox y API efímeros | Validación automática |
| Staging | API y PostgreSQL administrados | Demo integrada antes de producción |
| Production/demo | Por definir | Demostración pública con datos ficticios |

La configuración se inyectará por ambiente. No se mantendrán URLs, claves o secretos de producción dentro del código fuente.

## 12. Fases de ejecución

### Fase 0 — Aprobación y conexión segura

Objetivo: confirmar alcance y conectar correctamente los repositorios existentes.

Entregables:

- Aprobación de este plan maestro.
- Confirmación del slug y las URLs exactas de la organización/repositorios.
- Verificación de autenticación y permisos de GitHub.
- Inicialización o clonación segura de cada repositorio sin sobrescribir contenido remoto.
- Revisión de rama por defecto, licencia y visibilidad.
- Inventario del estado local y remoto.
- Primer ADR sobre stack y arquitectura.

Criterio de salida: los cinco repositorios están conectados, limpios y con una estrategia documentada de integración.

### Fase 1 — Gobierno open source y bases de los repositorios

Objetivo: hacer que cada repositorio sea mantenible y colaborativo.

Entregables comunes:

- README específico.
- Licencia aprobada por Flutter Piura.
- `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md` y `CHANGELOG.md`.
- Plantillas de issues y pull requests.
- `CODEOWNERS` cuando estén definidos los mantenedores.
- Conventional Commits y política de versionado semántico.
- GitHub Actions iniciales.
- Dependabot o alternativa equivalente donde aplique.
- Ruleset para `main`: PR, checks, sin force-push y sin borrado.

Criterio de salida: cada repositorio acepta contribuciones mediante PR y ejecuta validaciones básicas.

### Fase 2 — Producto, dominio y contrato v0

Objetivo: acordar el lenguaje del negocio antes de construir pantallas y persistencia.

Entregables:

- Product brief y alcance detallado del MVP.
- Glosario de dominio.
- Casos de uso y criterios de aceptación.
- Modelo de cuentas, dinero, movimientos, ledger y transferencias.
- OpenAPI inicial con ejemplos y errores comunes.
- Validación/lint del contrato en CI.
- Política de compatibilidad y generación del cliente Dart.
- ADR sobre importes, IDs, timestamps, paginación e idempotencia.

Criterio de salida: el contrato puede generar mocks y cliente sin errores, y representa un recorrido vertical completo.

### Fase 3 — Sandbox local con MobileLab

Objetivo: permitir que Flutter avance sin servidor cloud ni backend completo.

Entregables:

- MobileLab inicializado para `openbank_mobile`.
- Importación del OpenAPI autocontenido.
- Fixtures ficticios de usuarios, cuentas, saldos y movimientos.
- Autenticación de desarrollo.
- Escenarios de éxito, error 500, latencia, sesión expirada e idempotencia observable.
- Scripts y documentación para macOS/Linux/Windows cuando sea viable.
- Ejecución headless en CI con reportes.

Criterio de salida: un contribuidor puede clonar, iniciar el sandbox y consultar los endpoints documentados sin servicios cloud.

### Fase 4 — Fundación Flutter

Objetivo: convertir el proyecto Flutter inicial en una base de producción mantenible.

Entregables:

- Bootstrap por ambientes.
- Clean Architecture feature-first.
- Navegación y guardas de sesión.
- Inyección de dependencias.
- Cliente HTTP y mapeo uniforme de errores.
- Tema y componentes fundamentales accesibles.
- Almacenamiento seguro de sesión y caché no sensible.
- Localización inicial en español.
- Lints reforzados, pruebas y CI para Android/iOS.

Criterio de salida: la app inicia contra MobileLab, cambia de ambiente sin recompilar reglas de dominio y supera análisis/pruebas.

### Fase 5 — Primer recorrido funcional

Objetivo: completar el MVP móvil contra el sandbox.

Recorridos:

1. Login y restauración/cierre de sesión.
2. Dashboard con resumen de cuentas.
3. Lista, filtros básicos y detalle de movimientos.
4. Formulario, confirmación, resultado y comprobante de transferencia.
5. Estados de carga, vacío, reintento, sesión expirada y error.

Pruebas:

- Unitarias de entidades y casos de uso.
- Widgets de componentes y pantallas clave.
- Golden tests selectivos para estabilidad visual.
- Integración del recorrido crítico.
- Escenarios MobileLab para fallos de frontera.

Criterio de salida: el MVP es demostrable completamente en local con datos ficticios.

### Fase 6 — Backend local funcional

Objetivo: sustituir progresivamente los mocks por reglas y persistencia reales sin desplegar en cloud.

Entregables:

- Bootstrap de `openbank_api` como monolito modular.
- PostgreSQL local mediante Docker Compose.
- Migraciones y seeds ficticios.
- Autenticación local segura para demostración.
- Módulos identity, accounts, ledger, transactions y transfers.
- Transferencias atómicas, balanceadas e idempotentes.
- Validación de solicitudes y respuestas contra OpenAPI.
- Health/readiness checks y logs sanitizados.
- Pruebas unitarias, integración, contrato y end-to-end.

Criterio de salida: el mismo cliente Flutter funciona contra MobileLab y contra la API local, y el recorrido crítico conserva el comportamiento esperado.

### Fase 7 — Infraestructura y experiencia de desarrollo

Objetivo: hacer reproducible el sistema completo.

Entregables:

- Docker Compose para API y PostgreSQL.
- Variables `.env.example` sin secretos.
- Comandos consistentes para setup, start, test, lint y reset de datos.
- Migraciones automáticas controladas.
- Datos demo deterministas.
- Matriz de compatibilidad de herramientas.
- Guía de solución de problemas.
- Pruebas del stack completo en CI.

Criterio de salida: un contribuidor nuevo puede levantar el sistema siguiendo únicamente la documentación.

### Fase 8 — Seguridad, calidad y preparación de release

Objetivo: endurecer el MVP antes de una demo pública.

Entregables:

- Threat model educativo.
- Revisión de almacenamiento de tokens y logs.
- Escaneo de secretos y dependencias.
- Rate limiting y límites de payload en API.
- CORS y encabezados seguros según ambiente.
- Auditoría de accesibilidad móvil.
- Rendimiento del arranque y recorridos críticos.
- Matriz de pruebas y cobertura mínima acordada.
- SBOM y artefactos verificables cuando la cadena de build lo permita.

Criterio de salida: no existen hallazgos críticos conocidos y todos los checks de release pasan.

### Fase 9 — Staging opcional

Objetivo: disponer de una demo accesible sin convertir el proyecto en banca real.

Decisiones previas necesarias:

- Proveedor y presupuesto.
- Propiedad de las cuentas cloud.
- Gestión de secretos y backups.
- Dominio, TLS y observabilidad.
- Política de retención y reinicio de datos ficticios.

Entregables:

- Ambiente staging.
- CI/CD con aprobación para despliegue.
- Migraciones controladas y rollback probado.
- Smoke tests posteriores al despliegue.
- URLs y datos demo documentados.

Criterio de salida: la demo funciona de extremo a extremo con HTTPS y puede restaurarse de forma segura.

### Fase 10 — Release pública del MVP

Objetivo: publicar una versión coherente y reproducible.

Entregables:

- Changelogs finalizados.
- Tags firmados/anotados cuando la configuración lo permita.
- GitHub Releases con notas, artefactos y checksums aplicables.
- Matriz de versiones compatibles entre repositorios.
- Roadmap posterior al MVP.
- Issues `good first issue` y documentación de contribución.

Criterio de salida: cualquier release publicada referencia commits verificados, checks exitosos y versiones compatibles.

## 13. Estrategia Git y GitHub

### Ramas

- `main` será estable y protegida.
- Trabajo normal: `feat/...`, `fix/...`, `docs/...`, `refactor/...`, `test/...`, `ci/...` y `chore/...`.
- Las ramas serán cortas y pertenecerán a un solo repositorio.
- No se creará una rama `develop` permanente salvo que una necesidad real lo justifique.
- Los cambios entre repositorios se coordinarán mediante issues enlazados y una matriz de compatibilidad.

### Commits

Se aplicará Conventional Commits:

```text
feat(accounts): add account summary use case
fix(transfers): prevent duplicate idempotency key
docs(architecture): record API framework decision
ci(mobile): run analyzer and tests
```

Los commits serán pequeños, verificables y no mezclarán cambios no relacionados. No se reescribirá historia compartida sin autorización explícita.

### Pull requests

Cada cambio relevante se integrará mediante PR con:

- Contexto y alcance.
- Issue relacionado.
- Evidencia de pruebas.
- Impacto en contratos y compatibilidad.
- Capturas o video cuando cambie UI.
- Checklist de seguridad cuando corresponda.

Aunque inicialmente exista un solo mantenedor, se usarán PR para conservar trazabilidad y validar CI.

### Tags y releases

- Versionado semántico independiente por repositorio.
- Antes del MVP se usarán versiones `0.x.y`.
- No se creará un tag por cada commit o fase; solo por artefactos consumibles o hitos demostrables.
- Tags previstos: `v0.1.0` para fundaciones utilizables, incrementos menores para nuevas capacidades y `v1.0.0` únicamente cuando exista estabilidad declarada.
- `openbank_contracts` tendrá especial cuidado: una versión del cliente móvil y de la API declarará la versión de contrato compatible.
- Cada GitHub Release contendrá resumen, cambios incompatibles, instrucciones de actualización, artefactos y checksums cuando aplique.

### Acciones que el asistente podrá ejecutar después del “go”

- Conectar remotos y verificar ramas sin sobrescribir contenido.
- Crear ramas de trabajo.
- Implementar y probar cambios.
- Preparar commits y pushes.
- Abrir y actualizar pull requests cuando la autenticación disponible lo permita.
- Crear tags y GitHub Releases solo después de que el estado cumpla los criterios definidos.
- Configurar CI, reglas y automatizaciones dentro de los permisos concedidos.
- Informar antes de cualquier acción destructiva, gasto, despliegue público o cambio que requiera nueva autoridad.

## 14. Versiones coordinadas entre repositorios

Cada repositorio evoluciona de forma independiente, pero el sistema publicará una matriz como esta:

| Componente | Versión ejemplo | Depende de |
| --- | --- | --- |
| Mobile | `0.3.0` | Contracts `0.2.x`, API `0.2.x` |
| API | `0.2.1` | Contracts `0.2.x`, schema DB 3 |
| Contracts | `0.2.0` | — |
| Infrastructure | `0.2.0` | API image `0.2.x` |
| Docs | `0.1.0` | Describe el release MVP correspondiente |

Los cambios seguirán, cuando sea necesario, esta secuencia:

```text
contrato compatible → API → mobile → infraestructura → documentación/release
```

Un cambio incompatible deberá coexistir temporalmente o coordinar versiones mayores; no se romperá `main` en otros repositorios.

## 15. Calidad y definición de terminado

Una funcionalidad se considera terminada cuando:

- Cumple criterios de aceptación.
- Respeta límites de Clean Architecture.
- Tiene pruebas proporcionales al riesgo.
- Pasa formato, análisis, lint, test y build aplicables.
- No introduce secretos ni datos reales.
- Actualiza contrato y ejemplos si cambia la API.
- Actualiza documentación y changelog cuando corresponde.
- Maneja errores y accesibilidad, no solo el camino exitoso.
- Fue integrada mediante PR y CI exitoso.
- Puede ejecutarse siguiendo instrucciones reproducibles.

## 16. Seguridad y advertencias

- El repositorio será público: ningún secreto, token, certificado, credencial o dato personal podrá entrar al historial.
- Las credenciales demo serán claramente ficticias y separadas de cualquier ambiente real.
- MobileLab y sus tokens se usarán únicamente para desarrollo.
- Nunca se afirmará que el proyecto tiene certificación bancaria o cumplimiento regulatorio.
- No se harán pruebas contra cuentas o APIs bancarias reales.
- Las vulnerabilidades se reportarán por el canal indicado en `SECURITY.md`, no mediante un issue público cuando puedan explotarse.
- Los releases no serán evidencia suficiente para usar el sistema con dinero real.

## 17. Riesgos y mitigaciones

| Riesgo | Mitigación |
| --- | --- |
| Demasiados repositorios para un equipo pequeño | Automatización común, límites claros y releases solo cuando aporten valor |
| Mobile bloqueado por backend | Contrato primero y MobileLab como sandbox |
| Mocks alejados del backend real | Contract tests y ejecución del mismo cliente contra ambos |
| Reglas bancarias dentro de UI | Casos de uso y dominio independientes de Flutter |
| Errores con importes | Unidades menores enteras y value object `Money` |
| Transferencias duplicadas | Idempotency key, transacción DB y pruebas concurrentes |
| Divergencia de versiones | SemVer, changelogs y matriz de compatibilidad |
| Secretos en repos públicos | `.gitignore`, escaneo de secretos y revisión CI |
| Complejidad prematura | Monolito modular, sin microservicios/Kubernetes en el MVP |
| Dependencia excesiva de una librería | ADR, puertos/adaptadores y encapsulación en infraestructura |

## 18. Decisiones pendientes antes de implementar

Estas decisiones se resolverán como parte de las primeras fases, con propuesta técnica y ADR:

1. Slug y URLs exactas de la organización Flutter Piura.
2. Licencia del proyecto: propuesta inicial Apache-2.0 o MIT.
3. Identificador de aplicación Android/iOS y nombre público definitivo.
4. Stack de API: decisión base registrada en ADR-0001: NestJS + TypeScript + PostgreSQL.
5. Gestión de estado y navegación Flutter.
6. Herramienta de generación del cliente OpenAPI.
7. Publicación o consumo del cliente Dart entre repositorios.
8. Política mínima de cobertura y plataformas soportadas en CI.
9. Proveedor de staging, solo cuando el MVP local esté validado.
10. Mantenedores, CODEOWNERS y política de aprobación.

## 19. Primer bloque después de la aprobación

Tras recibir el “go”, se ejecutará únicamente la Fase 0 y se reportará su resultado antes de avanzar:

1. Verificar URLs remotas y contenido de los cinco repositorios.
2. Comprobar autenticación de GitHub y permisos en la organización.
3. Comparar el proyecto Flutter local con `openbank_mobile` remoto.
4. Conectar o clonar sin sobrescribir archivos.
5. Crear una rama `docs/project-foundation` en `openbank_docs`.
6. Incorporar este plan y el primer ADR mediante commit convencional.
7. Ejecutar validaciones disponibles.
8. Hacer push y preparar PR.
9. Esperar revisión del PR antes de aplicar cambios fundacionales a los demás repositorios.

No se crearán tags ni releases en este primer bloque: el plan por sí solo todavía no constituye un artefacto funcional. El primer tag se propondrá cuando exista una base reproducible que cumpla los criterios de su fase.

## 20. Criterio de éxito del proyecto

El MVP tendrá éxito cuando una persona nueva pueda:

1. Encontrar los cinco repositorios desde Flutter Piura.
2. Comprender su propósito y contribuir mediante la documentación.
3. Levantar MobileLab y ejecutar la app con datos ficticios.
4. Levantar la API y PostgreSQL localmente.
5. Ejecutar login, consulta de cuentas, movimientos y transferencia simulada.
6. Ver que las transferencias respetan idempotencia y ledger balanceado.
7. Ejecutar todas las pruebas y validaciones con comandos documentados.
8. Identificar exactamente qué versiones de cada repositorio son compatibles.

---

## Registro de aprobación

- Documento creado: 2026-08-30.
- Estado: aprobado para ejecución.
- Aprobado por: `GianSandoval5`, responsable del proyecto.
- Fecha de aprobación: 2026-08-30.
- Observaciones: avanzar por fases y reportar cada hito antes de cambios de alcance, despliegues públicos o acciones destructivas.
