# Práctica - Clase 4

Práctica realizada el día 4 de Septiembre del 2024 para la cuarta clase de Gestión de Datos.

## Ejercicio 1
Crear una tabla temporal #clientes a partir de la siguiente consulta: SELECT * FROM customer.

```sql
SELECT * INTO #clientes
FROM customer;
```

## Ejercicio 2
Insertar el siguiente cliente en la tabla #clientes

| Columna      | Valor        |
|--------------|--------------|
| customer_num | 144          |
| fname        | Agustin      |
| lname        | Creev        |
| company      | Jaguares SA  |
| state        | CA           |
| city         | Los Angeles  |

```sql
INSERT INTO #clientes
(customer_num, fname, lname, company, state, city)
VALUES (144, 'Agustin', 'Creevy', 'Jaguares SA', 'CA', 'Los Angeles');
```

## Ejercicio 3
Crear una tabla temporal #clientesCalifornia con la misma estructura de la tabla customer. Realizar un insert masivo en la tabla #clientesCalifornia con todos los clientes de la tabla customer cuyo state sea CA.

```sql
SELECT * INTO #clientesCalifornia
FROM customer WHERE state='CA';
```

## Ejercicio 4
Insertar el siguiente cliente en la tabla #clientes un cliente que tenga los mismos datos del cliente 103,pero cambiando en customer_num por 155. Valide lo insertado.

```sql
INSERT INTO #clientes 
(customer_num, fname, lname, company, address1, address2, city, state, zipcode, phone, customer_num_referedBy, status)
SELECT (155, fname, lname, company, address1, address2, city, state, zipcode, phone, customer_num_referedBy, status)
FROM customer 
WHERE customer_num = 103;
```

## Ejercicio 5
Borrar de la tabla #clientes los clientes cuyo campo zipcode esté entre 94000 y 94050 y la ciudad comience con ‘M’. Validar los registros a borrar antes de ejecutar la acción.

```sql
DELETE FROM #clientes 
WHERE (zipcode BETWEEN '94000' AND '94050') AND city LIKE 'M%'

// zipcode es un CHAR

// Con validar se refiere a hacerle un select a la 
// tabla y ver que ocurrio lo esperado 
```

## Ejercicio 6
Modificar los registros de la tabla #clientes cambiando el campo state por ‘AK’ y el campo address2 por ‘Barrio Las Heras’ para los clientes que vivan en el state 'CO'. Validar previamente la cantidad de registros a modificar.

```sql
UPDATE #clientes
SET state='AK', address2='Barrio Las Heras'
WHERE state='CO'

// Con validar se refiere a hacerle un select a la 
// tabla y ver que ocurrio lo esperado
```

## Ejercicio 7
Modificar todos los clientes de la tabla #clientes, agregando un dígito 1 delante de cada número
telefónico, debido a un cambio de la compañía de teléfonos.
```sql
UPDATE #clientes
SET phone='1'+phone
```
```sql
UPDATE #clientes
SET concat(1, phone)
```
