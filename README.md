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
3. Levantar el microservicio correspondiente (por ahora, `customer-service` en el puerto 8081
   contra Mongo local).
4. Correr las requests del folder correspondiente **en orden** — varias dependen del resultado de
   la anterior (por ejemplo, "Obtener cliente por id" usa el id que devolvio "Crear cliente
   personal", guardado automaticamente en la variable de environment `personalCustomerId` via un
   test script).

## folder `customer-service`

Reproduce exactamente el smoke test manual que se corrio a mano contra la app real antes de armar
esta coleccion: crear personal, crear empresarial, duplicado (409), reglas de negocio invalidas
(400 x2), listar, obtener por id, 404, actualizar, eliminar, 404 post-delete. Cada request incluye
un test script (`pm.test`) que verifica el status code esperado.

## Nota de esta sesion

Esta coleccion se armo y se valido como JSON bien formado, pero **no se corrio de punta a punta
con Newman** (CLI de Postman) porque esta maquina no tiene Node.js instalado. Las mismas 10
llamadas ya se habian probado a mano con `curl` contra `customer-service` real, con los mismos
resultados esperados que codifica cada test script — pero vale la pena que la corras una vez en
Postman antes de darla por buena del todo.
