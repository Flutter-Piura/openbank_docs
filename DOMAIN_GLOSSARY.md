# Glosario de dominio

Este documento define el lenguaje compartido de OpenBank. Los nombres técnicos pueden estar en inglés en el código, pero deben conservar el significado descrito aquí.

## Cliente (`Customer`)

Persona ficticia que inicia sesión y puede consultar las cuentas que le pertenecen. No representa una identidad verificada ni contiene información personal real.

## Cuenta (`Account`)

Contenedor ficticio de valor denominado en una moneda. Tiene identificador opaco, alias, número enmascarado y estado.

Estados conocidos inicialmente:

- `active`: permite operaciones compatibles.
- `blocked`: puede consultarse, pero no originar transferencias.
- `closed`: no permite nuevas operaciones.

## Dinero (`Money`)

Value object compuesto por:

- `minorUnits`: entero con signo en la unidad menor.
- `currency`: código ISO 4217 de tres letras.

Ejemplo: S/ 10.50 se representa como `1050 PEN`. Nunca se utiliza punto flotante.

Dos valores Money solo se suman o comparan directamente cuando tienen la misma moneda.

## Saldo disponible (`AvailableBalance`)

Importe que el dominio permite utilizar en una operación nueva. Es una proyección autoritativa de la API; la app móvil puede cachearlo para mostrarlo, pero no modificarlo ni usarlo como confirmación final.

## Ledger

Registro contable de asientos inmutables. Cada transferencia completada genera débitos y créditos cuya suma es exactamente cero por moneda.

## Asiento (`LedgerEntry`)

Débito o crédito inmutable asociado a una cuenta, una moneda y una operación. Un error no se corrige editando el asiento original; se registra una compensación cuando corresponda.

## Movimiento (`Transaction`)

Vista legible para el cliente de uno o más hechos del ledger. Incluye importe con signo, tipo, descripción y momento de ocurrencia. No es la fuente primaria de contabilidad.

## Transferencia (`Transfer`)

Intención de mover un importe positivo desde una cuenta origen hacia una cuenta destino interna.

Estados conocidos:

- `pending`: aceptada y aún sin resultado definitivo.
- `completed`: débito y crédito confirmados atómicamente.
- `rejected`: regla de dominio impidió procesarla.
- `failed`: error técnico impidió completar la operación.

La app no cambia el estado por sí misma.

## Clave de idempotencia (`IdempotencyKey`)

Identificador único creado por el cliente para una intención de transferencia. Repetir la misma clave y payload devuelve el resultado original. Reutilizar la clave con otro payload genera conflicto.

## Correlation ID

Identificador seguro de una solicitud o error usado para correlacionar logs. Puede mostrarse al usuario o contribuidor, pero no contiene datos sensibles.

## Cursor

Valor opaco que permite solicitar la siguiente página sin exponer offsets o detalles internos. El consumidor no lo interpreta ni modifica.

## Sandbox

Entorno local de MobileLab con fixtures, autenticación ficticia y escenarios controlados. Permite desarrollar y probar fronteras, pero no reemplaza las invariantes autoritativas del backend.

## Invariantes iniciales

1. Todo importe de transferencia es mayor que cero.
2. Origen y destino son diferentes.
3. Ambas cuentas existen, son visibles y admiten la operación.
4. La moneda del importe coincide con las reglas de las cuentas.
5. Una transferencia completada produce asientos balanceados.
6. Una clave de idempotencia no crea dos operaciones.
7. Un asiento confirmado es inmutable.
8. La app no confirma una operación sin respuesta autoritativa.
9. IDs y cursores son opacos.
10. Fechas de contrato se transmiten en UTC.
