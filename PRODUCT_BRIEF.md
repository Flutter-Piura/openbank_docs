# OpenBank — Product brief del MVP

## Problema

Las personas que aprenden Flutter y backend necesitan un proyecto realista para practicar arquitectura, contratos, pruebas, seguridad y colaboración multirepo. Una aplicación bancaria educativa ofrece reglas suficientemente interesantes, pero no debe confundirse con un sistema financiero real ni depender inicialmente de servicios cloud.

## Propuesta

OpenBank será una aplicación móvil de banca simulada que permite iniciar una sesión ficticia, consultar cuentas y movimientos, y transferir saldo ficticio entre cuentas internas. El proyecto será reproducible localmente mediante MobileLab y mediante una API real local con PostgreSQL.

## Personas

### Cliente demo

Quiere recorrer una experiencia bancaria clara y accesible sin registrar información personal real.

### Contribuidor Flutter

Quiere implementar features con Clean Architecture, estados verificables y una API estable.

### Contribuidor backend

Quiere practicar dominio, persistencia, ledger, idempotencia y contratos sin operar dinero real.

### Mantenedor

Quiere revisar cambios pequeños, reproducibles y compatibles entre los cinco repositorios.

## Recorrido crítico

```text
Abrir app
  → iniciar sesión demo
  → ver resumen de cuentas
  → abrir una cuenta
  → revisar movimientos
  → iniciar transferencia
  → confirmar datos
  → recibir resultado y comprobante
```

## Capacidades del MVP

### Autenticación ficticia

- Login con credenciales demo documentadas.
- Restauración de sesión.
- Renovación y cierre de sesión.
- Estado de sesión expirada reproducible.

### Cuentas

- Listado de cuentas visibles.
- Alias, número enmascarado, moneda y saldo disponible.
- Estados de carga, vacío y error.

### Movimientos

- Lista paginada del más reciente al más antiguo.
- Débitos y créditos con signo, moneda, descripción y fecha.
- Detalle de un movimiento.

### Transferencias

- Selección de cuenta origen y destino interno.
- Importe positivo y referencia opcional.
- Revisión antes de confirmar.
- Envío con clave de idempotencia.
- Resultado completado, rechazado o fallido.
- Comprobante ficticio.

### Resiliencia visible

- Reintento seguro.
- Sesión expirada.
- HTTP 500.
- Latencia controlada.
- Prevención de doble envío.

## Criterios de aceptación del recorrido crítico

1. La persona puede usar credenciales demo sin crear una cuenta real.
2. El saldo se presenta con formato de moneda, pero internamente conserva unidades menores enteras.
3. La app nunca muestra una transferencia como exitosa antes de recibir confirmación.
4. Repetir la misma solicitud con la misma clave no crea otra transferencia.
5. Reutilizar la clave con un payload distinto produce conflicto.
6. Una transferencia completada genera un débito y un crédito balanceados.
7. Los errores muestran un mensaje seguro y permiten correlacionar el incidente.
8. El recorrido funciona contra MobileLab y contra la API local.
9. Las pruebas y CI pasan en los repositorios afectados.
10. Ningún flujo solicita ni almacena datos financieros reales.

## Métricas del MVP

- Tiempo de setup local documentado y reproducible.
- Porcentaje de PR con CI exitoso antes de merge.
- Recorrido crítico cubierto por pruebas de integración.
- Cero secretos o datos reales en el historial.
- Compatibilidad declarada entre releases.

## No objetivos

- Operar dinero real.
- Integrarse con bancos, tarjetas o pasarelas.
- KYC real.
- Ofrecer asesoría financiera.
- Cumplir regulación bancaria de producción.
- Construir microservicios antes de demostrar la necesidad.
