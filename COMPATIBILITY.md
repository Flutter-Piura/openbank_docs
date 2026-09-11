# Compatibilidad entre repositorios

Cada repositorio usa versionado semántico independiente. Antes de `1.0.0`, una versión menor puede incorporar capacidades relevantes y una versión patch debe conservar compatibilidad.

## Matriz actual

| Componente | Estado/versión | Compatible con |
| --- | --- | --- |
| `openbank_contracts` | `0.1.0` | MobileLab 1.1.0, Mobile/API `0.1.x` |
| `openbank_mobile` | `0.1.0` | Contracts `0.1.x`, MobileLab 1.1.0, API `0.1.x` |
| `openbank_api` | `0.1.0`, schema DB 2 | Contracts `0.1.x`, PostgreSQL 17 |
| `openbank_infrastructure` | `0.1.0` | API `0.1.x`, PostgreSQL 17.6 |
| `openbank_docs` | `0.1.0` | Describe el vertical slice local `0.1.x` |

## Reglas

- Ningún consumidor depende de una rama de otro repositorio para un release.
- Los contratos se consumen desde tags o artefactos identificables.
- Un cambio incompatible requiere migración, coexistencia o incremento mayor.
- oasdiff bloquea cambios incompatibles accidentales en OpenAPI.
- Los changelogs indican versiones de consumidores afectadas.
- La matriz se actualiza antes de cada release coordinada.

## Hito integrado actual

La evidencia y las instrucciones reproducibles están en el [informe de integración local 0.1.0](INTEGRATION_REPORT_0.1.0.md).

## Orden habitual de cambio

```text
contrato compatible
  → sandbox/API
  → mobile
  → infraestructura
  → documentación y release
```
