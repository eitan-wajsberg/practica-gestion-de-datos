# Práctica - Clase 11

Práctica realizada el día 23 de Octubre del 2024 para la onceava clase de Gestión de Datos.

## Ejercicio 1
Listar Número de Cliente, apellido y nombre, Total Comprado por el cliente ‘Total del Cliente’, Cantidad de Órdenes de Compra del cliente ‘OCs del Cliente’ y la Cant. de Órdenes de Compra solicitadas por todos los clientes ‘Cant. Total OC’, de todos aquellos clientes cuyo promedio de compra por Orden supere al promedio de órdenes de compra general, tengan al menos 2 órdenes y cuyo zipcode comience con 94.

```sql
SELECT c.customer_num, c.lname, c.fname, 
    SUM(i.quantity * i.unit_price) total_comprado, 
    COUNT(DISTINCT o.order_num) cantidad_oc,
    (SELECT COUNT(*) FROM orders) cantidad_total_oc
FROM customer c
    INNER JOIN orders o ON o.customer_num = c.customer_num
    INNER JOIN items i ON i.order_num = o.order_num
WHERE c.zipcode LIKE '94%'
GROUP BY c.customer_num, c.lname, c.fname
HAVING SUM(i.quantity * i.unit_price) / COUNT(DISTINCT sub_o.order_num) > (
        SELECT SUM(sub_i.quantity * sub_i.unit_price) / COUNT(DISTINCT sub_o.order_num)
        FROM items sub_i INNER JOIN orders sub_o ON sub_i.order_num = sub_o.order_num
    )
    AND COUNT(DISTINCT o.order_num) >= 2;
```

## Ejercicio 2
Se requiere crear una tabla temporal #ABC_Productos. Los datos solicitados son: Nro. de Stock, Código de fabricante, descripción del producto, Nombre de Fabricante, Total del producto pedido 'u$ por Producto', Cant. de producto pedido 'Unid. por Producto', para los productos que pertenezcan a fabricantes que fabriquen al menos 10 productos diferentes.

```sql
SELECT i.stock_num, m.manu_code, pt.description, m.manu_name, SUM(i.quantity * i.unit_price) 'u$ por Producto', COUNT(i.stock_num) 'Unid. por Producto'
INTO #ABC_Productos FROM items i 
    INNER JOIN manufact m ON m.manu_code = i.manu_code 
    INNER JOIN product_types pt ON pt.stock_num = i.stock_num
WHERE m.manu_code IN (
    SELECT DISTINCT manu_code 
    FROM products
	GROUP BY manu_code
    HAVING COUNT(stock_num) >= 10
)
GROUP BY i.stock_num, m.manu_code, pt.description, m.manu_name;

SELECT * FROM #ABC_Productos;
```


## Ejercicio 3
En función a la tabla temporal generada en el punto 2, obtener un listado que detalle para cada tipo de producto existente en #ABC_Producto, la descripción del producto, el mes en el que fue solicitado, el cliente que lo solicitó (en formato 'Apellido, Nombre'), la cantidad de órdenes de compra 'Cant OC por mes', la cantidad del producto solicitado 'Unid Producto por mes' y el total en u$ solicitado 'u$ Producto por mes'. Mostrar sólo aquellos clientes que vivan en el estado con mayor cantidad de clientes, ordenado por mes y descripción del tipo de producto en forma ascendente y por cantidad de productos por mes en forma descendente.

```sql
SELECT pt.description, MONTH(o.order_date), c.lname + ', ' + c.fname, COUNT(o.order_num) 'Cant OC por mes', SUM(i.quantity) 'Unid Producto por mes',  SUM(i.quantity * i.unit_price) 'u$ Producto por mes'
FROM orders o
    INNER JOIN customer c ON c.customer_num = o.customer_num
    INNER JOIN items i ON i.order_num = o.order_num
    INNER JOIN product_types pt ON pt.stock_num = i.stock_num
WHERE i.stock_num IN (SELECT stock_num FROM #ABC_Productos) AND c.state = (
    SELECT TOP 1 state 
    FROM customer
    GROUP BY state
    ORDER BY COUNT(*) DESC
)
GROUP BY pt.description, MONTH(o.order_date), c.lname, c.fname
ORDER BY MONTH(o.order_date) ASC, pt.description ASC, COUNT(i.stock_num) DESC;
```


## Ejercicio 4
Dado los productos con número de stock 5, 6 y 9 del fabricante 'ANZ' listar de a pares los clientes que hayan solicitado el mismo producto, siempre y cuando, el primer cliente haya solicitado más cantidad del producto que el 2do cliente. Se deberá informar nro de stock, código de fabricante, Nro de Cliente y Apellido del primer cliente, Nro de cliente y apellido del 2do cliente ordenado por stock_num y manu_code

```sql
-- REVISAR!
SELECT p.stock_num, p.manu_code, CONCAT(c1.customer_num, ' ', c1.lname) AS 'Primer cliente', CONCAT(c2.customer_num, ' ', c2.lname) AS 'Segundo cliente'
FROM products p
    INNER JOIN items i1 ON i1.stock_num = p.stock_num
    INNER JOIN orders o1 ON o1.order_num = i1.order_num
    INNER JOIN customer c1 ON c1.customer_num = o1.customer_num
    INNER JOIN items i2 ON i2.stock_num = p.stock_num
    INNER JOIN orders o2 ON o2.order_num = i2.order_num
    INNER JOIN customer c2 ON c2.customer_num = o2.customer_num
WHERE p.stock_num IN (5, 6, 9) AND p.manu_code = 'ANZ'AND c1.customer_num < c2.customer_num
GROUP BY p.stock_num, p.manu_code, c1.customer_num, c1.lname, c2.customer_num, c2.lname
HAVING SUM(i1.quantity) > SUM(i2.quantity)
ORDER BY p.stock_num, p.manu_code;
```


## Ejercicio 5
Se requiere realizar una consulta que devuelva en una fila la siguiente información: La mayor cantidad de órdenes de compra de un cliente, mayor total en u$ solicitado por un cliente y la mayor cantidad de productos solicitados por un cliente, la menor cantidad de órdenes de compra de un cliente, el menor total en u$ solicitado por un cliente y la menor cantidad de productos solicitados por un cliente. Los valores máximos y mínimos solicitados deberán corresponderse a los datos de clientes según todas las órdenes existentes, sin importar a que cliente corresponda el dato.

```sql
SELECT 
	MAX(cantidad_ordenes) 'Mayor Cantidad OC', 
	MAX(total) 'Mayor Total en u$', 
	MAX(cantidad_productos) 'Mayor Cantidad Productos', 
	MIN(cantidad_ordenes) 'Menor Cantidad OC', 
	MIN(total) 'Menor Total en u$', 
	MIN(cantidad_productos) 'Menor Cantidad Productos'
FROM (
    SELECT COUNT(o.order_num) cantidad_ordenes, SUM(i.quantity * i.unit_price) total, SUM(i.quantity) cantidad_productos
    FROM orders o INNER JOIN items i ON i.order_num = o.order_num
	GROUP BY o.customer_num
) s;
```


## Ejercicio 6
Seleccionar los número de cliente, número de orden y monto total de la orden de aquellos clientes del estado California (CA) que posean 4 o más órdenes de compra emitidas en el 2015. Además las órdenes mostradas deberán cumplir con la salvedad que la cantidad de líneas de ítems de esas órdenes debe ser mayor a la cantidad de líneas de ítems de la orden de compra con mayor cantidad de ítems del estado AZ en el mismo año.

```sql
SELECT c.customer_num, o.order_num, SUM(i.quantity * i.unit_price) total_orden
FROM orders o
	INNER JOIN items i ON i.order_num = o.order_num
	INNER JOIN customer c ON c.customer_num = o.customer_num
WHERE c.state = 'CA' AND YEAR(o.order_date) = 2015
GROUP BY c.customer_num, o.order_num
HAVING COUNT(o.order_num) >= 4 AND COUNT(i.item_num) > (
	SELECT MAX(cantidad_items)
		FROM (
			SELECT COUNT(i2.item_num) cantidad_items FROM items i2
				INNER JOIN orders o2 ON o2.order_num = i2.order_num
				INNER JOIN customer c2 ON c2.customer_num = o2.customer_num
			WHERE c2.state = 'AZ' AND YEAR(o2.order_date) = 2015
			GROUP BY o2.order_num
		) subquery
);
```


## Ejercicio 7
Se requiere listar para el Estado de California el par de clientes que sean los que suman el mayor monto en dólares en órdenes de compra, con el formato de salida: 'Código Estado', 'Descripción Estado', 'Apellido, Nombre', 'Apellido, Nombre', 'Total Solicitado'. El total solicitado contendrá la suma de los dos clientes.

```sql
-- Alternativa 1
SELECT s.state 'Código Estado', s.sname 'Descripción Estado', CONCAT(s1.lname, ', ', s1.fname) 'Primer Cliente', CONCAT(s2.lname, ', ', s2.fname) 'Segundo Cliente', total_cliente1 + total_cliente2 'Total Solicitado'
FROM state s
    INNER JOIN (
        SELECT TOP 1 c.customer_num, lname, fname, state, SUM(i.quantity * i.unit_price) total_cliente1
        FROM orders o 
            INNER JOIN items i ON i.order_num = o.order_num
            INNER JOIN customer c ON c.customer_num = o.customer_num
        WHERE c.state = 'CA'
        GROUP BY c.customer_num, lname, fname, state
        ORDER BY total_cliente1 DESC
    ) s1 ON s1.state = s.state
    INNER JOIN (
        SELECT TOP 2 c.customer_num, lname, fname, state, SUM(i.quantity * i.unit_price) total_cliente2
        FROM orders o 
            INNER JOIN items i ON i.order_num = o.order_num
            INNER JOIN customer c ON c.customer_num = o.customer_num
        WHERE c.state = 'CA'
        GROUP BY c.customer_num, lname, fname, state
        ORDER BY total_cliente2 DESC
    ) s2 ON s2.state = s.state
WHERE s1.customer_num <> s2.customer_num;

-- Alternativa 2
WITH TotalPorCliente AS (
    SELECT c.customer_num,c.lname, c.fname, SUM(i.quantity * i.unit_price) AS total
    FROM orders o 
		INNER JOIN items i ON i.order_num = o.order_num
		INNER JOIN customer c ON c.customer_num = o.customer_num
    WHERE c.state = 'CA'
    GROUP BY c.customer_num, c.lname, c.fname
),
TopClientes AS (
    SELECT customer_num, lname, fname, total, ROW_NUMBER() OVER (ORDER BY total DESC) AS rn
    FROM TotalPorCliente
)
SELECT s.state AS 'Código Estado', s.sname AS 'Descripción Estado', CONCAT(c1.lname, ', ', c1.fname) AS 'Primer Cliente', CONCAT(c2.lname, ', ', c2.fname) AS 'Segundo Cliente', (c1.total + c2.total) AS 'Total Solicitado'
FROM state s
	INNER JOIN TopClientes c1 ON c1.rn = 1
	INNER JOIN TopClientes c2 ON c2.rn = 2
WHERE s.state = 'CA';
```


## Ejercicio 8
Se observa que no se cuenta con stock suficiente para las últimas 5 órdenes de compra emitidas que contengan productos del fabricante 'ANZ'. Por lo que se decide asignarle productos en stock a la orden del cliente que más cantidad de productos del fabricante 'ANZ' nos haya comprado. Se solicita listar el número de orden de compra, número de cliente, fecha de la orden y una fecha de orden “modificada” a la cual se le suma el lead_time del fabricante más 1 día por preparación del pedido a aquellos clientes que no son prioritarios. Para aquellos clientes a los que les entregamos los productos en stock, la “fecha modificada” deberá estar en NULL. Listar toda la información ordenada por “fecha modificada”.

```sql
SELECT o.order_num, o.customer_num, o.order_date, NULL fecha_modificada
FROM orders o
    INNER JOIN items i ON i.order_num = o.order_num
    INNER JOIN manufact m ON m.manu_code = i.manu_code
WHERE o.customer_num = (
    SELECT TOP 1 customer_num 
    FROM orders o INNER JOIN items i ON i.order_num = o.order_num
    WHERE i.manu_code = 'ANZ'
    GROUP BY customer_num
    ORDER BY COUNT(i.quantity) DESC
) AND o.order_num IN (
    SELECT TOP 5 order_num 
    FROM orders
    ORDER BY order_date DESC
)
UNION
SELECT DISTINCT o.order_num, o.customer_num, o.order_date, DATEADD(DAY, (m.lead_time + 1), o.order_date) AS fecha_modificada
FROM orders o
    INNER JOIN items i ON i.order_num = o.order_num
    INNER JOIN manufact m ON m.manu_code = i.manu_code
WHERE o.customer_num <> (
    SELECT TOP 1 customer_num 
    FROM orders o INNER JOIN items i ON i.order_num = o.order_num
    WHERE i.manu_code = 'ANZ'
    GROUP BY customer_num
    ORDER BY COUNT(i.quantity) DESC
) AND o.order_num IN (
    SELECT TOP 5 order_num 
    FROM orders
    ORDER BY order_date DESC
)
ORDER BY fecha_modificada;
```


## Ejercicio 9
Listar el número, nombre, apellido, estado, cantidad de órdenes y monto total comprado de los clientes que no sean del estado de Wisconsin y cuyo monto total comprado sea mayor que el monto total promedio de órdenes de compra.

```sql
SELECT c.customer_num, c.fname, c.lname, c.state, COUNT(DISTINCT o1.order_num) cantidad, SUM(i1.quantity * i1.unit_price) total
FROM customer c
    INNER JOIN orders o1 ON o1.customer_num = c.customer_num
    INNER JOIN items i1 ON i1.order_num = o1.order_num
WHERE c.state <> 'WI'
GROUP BY c.customer_num, c.fname, c.lname, c.state
HAVING SUM(i1.quantity * i1.unit_price) > (
    SELECT SUM(quantity * unit_price) / COUNT(DISTINCT o2.order_num)
    FROM items i2 INNER JOIN orders o2 ON i2.order_num = o2.order_num 
);
```