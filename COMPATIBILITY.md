# Compatibilidad entre repositorios

Cada repositorio usa versionado semántico independiente. Antes de `1.0.0`, una versión menor puede incorporar capacidades relevantes y una versión patch debe conservar compatibilidad.

## Matriz actual

| Componente | Estado/versión | Compatible con |
| --- | --- | --- |
| `openbank_contracts` | contrato `0.1.0` en desarrollo | MobileLab y futura API `0.1.x` |
| `openbank_mobile` | fundación sin release | contrato `0.1.x` previsto |
| `openbank_api` | fundación sin release | contrato `0.1.x` previsto |
| `openbank_infrastructure` | fundación sin release | futura imagen API `0.1.x` |
| `openbank_docs` | documentación viva | describe la línea `0.1.x` |

## Reglas

- Ningún consumidor depende de una rama de otro repositorio para un release.
- Los contratos se consumen desde tags o artefactos identificables.
- Un cambio incompatible requiere migración, coexistencia o incremento mayor.
- oasdiff bloquea cambios incompatibles accidentales en OpenAPI.
- Los changelogs indican versiones de consumidores afectadas.
- La matriz se actualiza antes de cada release coordinada.

## Orden habitual de cambio

```text
contrato compatible
  → sandbox/API
  → mobile
  → infraestructura
  → documentación y release
```
