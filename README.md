# bootcamp-postman

Colecciones Postman de verificacion funcional de los microservicios del sistema bancario
(`proyecto-bootcamp`). El enunciado pide explicitamente un repo aparte para esto ("crear y
mantener un repositorio en donde tengan los proyectos postman para las pruebas de sus APIs",
Parte III) y que toda funcionalidad se verifique con Postman, ya que el sistema no tiene interfaz
grafica (Parte I).

## Contenido

- `bootcamp-bancario.postman_collection.json` — una coleccion unica, con un folder por
  microservicio (se va agregando un folder nuevo a medida que se suma cada servicio).
- `environments/local.postman_environment.json` — variables para correr todo contra los
  microservicios levantados en local.

## Como usar

1. Importar ambos archivos en Postman (File &rarr; Import).
2. Seleccionar el environment "bootcamp-local" arriba a la derecha.
3. Levantar los microservicios correspondientes: `customer-service` (8081), `account-service`
   (8082), `credit-service` (8083), `card-service` (8084) — los ultimos tres llaman a
   `customer-service` por REST, asi que para sus folders hace falta `customer-service` levantado
   tambien, mas Mongo local.
4. Correr las requests del folder correspondiente **en orden** — varias dependen del resultado de
   la anterior (por ejemplo, "Obtener cliente por id" usa el id que devolvio "Crear cliente
   personal", guardado automaticamente en la variable de environment `personalCustomerId` via un
   test script).

## folder `customer-service`

Reproduce exactamente el smoke test manual que se corrio a mano contra la app real antes de armar
esta coleccion: crear personal, crear empresarial, duplicado (409), reglas de negocio invalidas
(400 x2), listar, obtener por id, 404, actualizar, eliminar, 404 post-delete. Cada request incluye
un test script (`pm.test`) que verifica el status code esperado.

## folder `account-service`

23 requests: 2 de setup (crear cliente personal y empresarial en `customer-service`, sus ids
quedan en `accPersonalCustomerId`/`accBusinessCustomerId` — variables separadas de las del folder
`customer-service` para no pisarlas), y el resto reproduce el smoke test manual completo: alta de
cuenta de ahorro personal, limite de 1 cuenta por tipo (D8), cliente/holder/signer inexistente,
signer vacio rechazado por Bean Validation, cuenta empresarial de ahorro bloqueada (D8), deposito
sin `Idempotency-Key` (400) y con clave repetida (no duplica el movimiento), retiro valido y con
fondos insuficientes (422), listar movimientos, `update` re-validando D8 (agregar holder invalido
falla, agregar signer valido no), y `delete` bloqueado con saldo distinto de cero.

## folder `credit-service`

23 requests: 2 de setup (clientes personal/empresarial), y el resto reproduce el smoke test manual
completo: alta de credito con cuotas generadas automaticamente, limite de 1 credito **activo** por
persona (D8 — permite uno nuevo una vez pagado el anterior), cliente inexistente, pago sin
`Idempotency-Key` (400), pago con monto que no coincide con la cuota (400), pago valido con clave
repetida (no duplica), pagar las 3 cuotas y confirmar que el credito pasa a `PAID` automaticamente,
listar pagos, `update`/`delete` bloqueados si ya hay cuotas pagadas pero permitidos si no,
empresarial con multiples creditos sin limite.

## folder `card-service`

20 requests: alta de tarjeta, **segunda tarjeta para el mismo cliente sin bloqueo** (a diferencia
de cuentas/creditos, tarjetas no tienen limite por cliente), cliente inexistente, consumo sin
`Idempotency-Key` (400), consumo valido con clave repetida (no duplica), consumo que supera el
disponible (422), pago valido, pago que supera lo adeudado (400), listar movimientos,
`update`/`delete` bloqueados si ya hay movimientos o saldo pendiente respectivamente, permitidos
si no.

## Nota de esta sesion

Esta coleccion se armo y se valido como JSON bien formado, pero **no se corrio de punta a punta
con Newman** (CLI de Postman) porque esta maquina no tiene Node.js instalado. Las mismas llamadas
ya se habian probado a mano con `curl` contra los servicios reales, con los mismos resultados
esperados que codifica cada test script — pero vale la pena que corras ambos folders una vez en
Postman antes de darlos por buenos del todo.
