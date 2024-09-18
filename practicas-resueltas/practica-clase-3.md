# Práctica - Clase 3

Práctica realizada el día 28 de Agosto del 2024 para la tercera clase de Gestión de Datos.

## Ejercicio 1
Obtener un listado de todos los clientes y sus direcciones.

```sql
SELECT customer_num, fname, lname, address1, address2
FROM customer;
```

## Ejercicio 2
Obtener el listado anterior pero sólo los clientes que viven en el estado de California “CA”.

```sql
SELECT customer_num, fname, lname, address1, address2, city, state,
FROM customer
WHERE state='CA';
```

## Ejercicio 3
Listar todas las ciudades (city) de la tabla clientes que pertenecen al estado de “CA”, mostrar sólo una vez cada ciudad.

```sql
SELECT DISTINCT city
FROM customer
WHERE state='CA';
```

## Ejercicio 4
Ordenar la lista anterior alfabéticamente.

```sql
SELECT DISTINCT city
FROM customer
WHERE state='CA'
ORDER BY city;
```

## Ejercicio 5
Mostrar la dirección sólo del cliente 103. (customer_num)

```sql
SELECT DISTINCT address1, address2
FROM customer
WHERE customer_num=103;
```

## Ejercicio 6
Mostrar la lista de productos que fabrica el fabricante “ANZ” ordenada por el campo Código de Unidad de Medida. (unit_code)

```sql
SELECT unit_price, unit_code, stock_num
FROM products
WHERE manu_code='ANZ'
ORDER BY unit_code;
```

## Ejercicio 7
Listar los códigos de fabricantes que tengan alguna orden de pedido ingresada, ordenados alfabéticamente y no repetidos.

```sql
SELECT DISTINCT manu_code
FROM items
ORDER BY manu_code;
```

## Ejercicio 8
Escribir una sentencia SELECT que devuelva el número de orden, fecha de orden, número de cliente y fecha de embarque de todas las órdenes que no han sido pagadas (paid_date es nulo), pero fueron embarcadas (ship_date) durantelos primeros seis meses de 2015.

```sql
SELECT DISTINCT order_num, order_date, customer_num, ship_date
FROM orders
WHERE paid_date IS NULL AND ship_date BETWEEN '2015-01-01' AND '2015-07-01'
ORDER BY order_num;
```

## Ejercicio 9
Obtener de la tabla cliente (customer) los número de clientes y nombres de las compañías, cuyos nombres de compañías contengan la palabra “town”.

```sql
SELECT custumer_num, company
FROM custumer
WHERE company LIKE '%town%';
```

## Ejercicio 10
Obtener el precio máximo, mínimo y precio promedio pagado (ship_charge) portodos los embarques. Se pide obtener la información de la tabla ordenes (orders).

```sql
SELECT MAX(ship_charge), MIN(ship_charge), AVG(ship_charge)
FROM orders;
```

## Ejercicio 11
Realizar una consulta que muestre el número de orden, fecha de orden y fecha de embarque de todas que fueron embarcadas (ship_date) en el mismo mes que fue dada de alta la orden(order_date).

```sql
SELECT order_num, order_date, ship_date
FROM orders
WHERE YEAR(ship_date)=YEAR(order_date) AND MONTH(ship_date)=MONTH(order_date);
```

## Ejercicio 12
Obtener la Cantidad de embarques y Costo total (ship_charge) del embarque por número de cliente y por fecha de embarque. Ordenar los resultados por el total de costo en orden inverso.

```sql
SELECT COUNT(*), SUM(ship_charge) total, customer_num, ship_date
FROM orders
GROUP BY customer_num, ship_date
ORDER BY total DESC;
```

## Ejercicio 13
Mostrar fecha de embarque (ship_date) y cantidad total de libras (ship_weight) por día, de aquellos días cuyo peso de los embarques superen las 30 libras. Ordenar el resultado por el total de libras en orden descendente.

```sql
SELECT ship_date, SUM(ship_weight) total_libras
FROM orders
GROUP BY ship_date
HAVING SUM(ship_weight) > 30
ORDER BY total_libras DESC;
```

## Ejercicio 14
Crear una consulta que liste todos los clientes que vivan en California ordenados por compañía.

```sql
SELECT *
FROM customer
WHERE state='CA'
ORDER BY company;
```

## Ejercicio 15
Obtener un listado de la cantidad de productos únicos comprados a cada fabricante, en donde el total comprado a cada fabricante sea mayor a 1500. El listado deberá estar ordenado por cantidad de productos comprados de mayor a menor.

```sql
SELECT manu_code, COUNT(DISTINCT stock_num)
FROM items
GROUP BY manu_code
HAVING SUM(quantity * unit_price) > 1500
ORDER BY COUNT(DISTINCT stock_num) DESC;
```

## Ejercicio 16
Obtener un listado con el código de fabricante, nro de producto, la cantidad vendida (quantity), y el total vendido (quantity x unit_price), para los fabricantes cuyo código tiene una “R” como segunda letra. Ordenar el listado por código de fabricante y nro de producto.

```sql
SELECT manu_code, stock_num, COUNT(quantity) cantidad_vendida, SUM(quantity * unit_price) total_vendido
FROM items
WHERE manu_code LIKE '_R%'
GROUP BY manu_code, stock_num
ORDER BY manu_code, stock_num;
```

## Ejercicio 17
Crear una tabla temporal OrdenesTemp que contenga las siguientes columnas: cantidad de órdenes por cada cliente, primera y última fecha de orden de compra (order_date) del cliente. Realizar una consulta de la tabla temp OrdenesTemp en donde la primer fecha de compra sea anterior a '2015-05-23 00:00:00.000', ordenada por fechaUltimaCompra en forma descendente.

```sql
SELECT customer_num, COUNT(order_num) cantidad_compras, MIN(order_date) primera_orden, MAX(order_date) ultima_orden
INTO #ordenesTemp
FROM orders
GROUP BY customer_num;

SELECT *
FROM #ordenesTemp
WHERE primera_orden < '2015-05-23 00:00:00.000'
ORDER BY ultima_orden DESC;
```

## Ejercicio 18
Consultar la tabla temporal del punto anterior y obtener la cantidad de clientes con igual cantidad de compras. Ordenar el listado por cantidad de compras en orden descendente.

```sql
SELECT COUNT(customer_num) cantidad_clientes, cantidad_compras,
FROM #ordenesTemp
GROUP BY cantidad_compras
ORDER BY cantidad_compras DESC;
```

## Ejercicio 19
Desconectarse de la sesión. Volver a conectarse y ejecutar SELECT * from #ordenesTemp. Que sucede?
No funciona, ya que es una tabla temporal.


## Ejercicio 20
Se desea obtener la cantidad de clientes por cada state y city, donde los clientes contengan el string ‘ts’ en el nombre de compañía, el código postal este entre 93000 y 94100 y la ciudad no sea 'Mountain View'. Se desea el listado ordenado por ciudad

```sql
SELECT state, city, COUNT(customer_num) cantidad_clientes
FROM customer
WHERE company LIKE '%ts%' 
	  AND zipcode BETWEEN 93000 AND 94100 
	  AND city != 'Mountain View'
GROUP BY state, city
ORDER BY city;
```

## Ejercicio 21
Para cada estado, obtener la cantidad de clientes referidos. Mostrar sólo los clientes que hayan sido referidos cuya compañía empiece con una letra que este en el rango de ‘A’ a ‘L’.

```sql
SELECT state, COUNT(customer_num) cantidad_clientes
FROM customer
WHERE company LIKE '[A-L]%'
	  AND customer_num_referedBy IS NOT NULL
GROUP BY state;
```

## Ejercicio 22
Se desea obtener el promedio de lead_time por cada estado, donde los Fabricantes tengan una ‘e’ en manu_name y el lead_time sea entre 5 y 20.

```sql
SELECT state, AVG(lead_time)
FROM manufact
WHERE manu_name LIKE '%e%'
	  AND lead_time BETWEEN 5 AND 20
GROUP BY state;
```

## Ejercicio 23
Se tiene la tabla units, de la cual se quiere saber la cantidad de unidades que hay por cada tipo (unit) que no tengan en nulo el descr_unit, y además se deben mostrar solamente los que cumplan que la cantidad mostrada se superior a 5. Al resultado final se le debe sumar 1.

```sql
SELECT unit, COUNT(unit) + 1 cantidad_unidades
FROM units
WHERE unit_descr IS NOT NULL
GROUP BY unit
HAVING COUNT(unit) > 5;
```
