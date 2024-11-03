# Práctica - Clase 10

Práctica realizada el día 23 de Octubre del 2024 para la decima clase de Gestión de Datos.

## Ejercicio 1
Crear una vista que devuelva:
- Código y Nombre (manu_code, manu_name) de los fabricante, posean o no productos (en tabla Products), cantidad de productos que fabrican (cant_producto) y la fecha de la última OC que contenga un producto suyo (ult_fecha_orden). De los fabricantes que fabriquen productos sólo se podrán mostrar los que fabriquen más de 2 productos. No se permite utilizar funciones definidas por usuario, ni tablas temporales, ni UNION.
- Realizar una consulta sobre la vista que devuelva manu_code, manu_name, cant_producto y si el campo ult_fecha_orden posee un NULL informar ‘No Posee Órdenes’ si no posee NULL informar el valor de dicho campo. No se puede utilizar UNION para el SELECT.

```sql
-- Alternativa 1 (Mejor)
ALTER VIEW productosDeFabricantes AS
SELECT m.manu_code, m.manu_name, COUNT(DISTINCT p.stock_num) AS cant_producto, MAX(o.order_date) AS ult_fecha_orden
FROM manufact m
    LEFT JOIN products p ON m.manu_code = p.manu_code
    LEFT JOIN items i ON i.stock_num = p.stock_num
    LEFT JOIN orders o ON o.order_num = i.order_num
GROUP BY m.manu_code, m.manu_name
HAVING COUNT(DISTINCT p.stock_num) > 2 OR COUNT(DISTINCT p.stock_num) = 0;

SELECT manu_code, manu_name, cant_producto, CASE 
        WHEN ult_fecha_orden IS NULL THEN 'No Posee Órdenes' 
        ELSE CAST(ult_fecha_orden AS VARCHAR)
    END AS ult_fecha_orden
FROM productosDeFabricantes;

-- Alternativa 2
ALTER VIEW productosDeFabricantes AS
SELECT m.manu_code, m.manu_name, COUNT(DISTINCT p.stock_num) cant_producto, MAX(o.order_date) ult_fecha_orden
FROM products p
	INNER JOIN items i ON i.stock_num = p.stock_num
    INNER JOIN orders o ON o.order_num = i.order_num
    RIGHT JOIN manufact m ON m.manu_code = p.manu_code
WHERE m.manu_code IN (
    SELECT manu_code
    FROM products
    GROUP BY manu_code
    HAVING COUNT(*) > 2
) OR m.manu_code NOT IN (SELECT manu_code FROM products)
GROUP BY m.manu_code, m.manu_name;

SELECT manu_code, manu_name, cant_producto, ISNULL(CAST(ult_fecha_orden AS VARCHAR), 'No Posee Órdenes') ult_fecha_orden 
FROM productosDeFabricantes;
```

## Ejercicio 2
Desarrollar una consulta ABC de fabricantes que:
Liste el código y nombre del fabricante, la cantidad de órdenes de compra que contengan sus productos y monto total de los productos vendidos. Mostrar sólo los fabricantes cuyo código comience con A ó con N y posea 3 letras, y los productos cuya descripción posean el string “tennis” ó el string “ball” en cualquier parte del nombre y cuyo monto total vendido sea mayor que el total de ventas promedio de todos los fabricantes (Cantidad * precio unitario / Cantidad de fabricantes que vendieron sus productos). Mostrar los registros ordenados por monto total vendido de mayor a menor.

```sql
SELECT m.manu_code, m.manu_name, COUNT(p.stock_num) cant_productos, SUM(i.unit_price * i.quantity) monto_total
FROM manufact m 
    INNER JOIN items i ON m.manu_code = i.manu_code
    INNER JOIN products p ON p.stock_num = i.stock_num
    INNER JOIN product_types pt ON pt.stock_num = p.stock_num
WHERE 
    (m.manu_code LIKE 'A%' OR m.manu_code LIKE 'N%')
    AND LEN(m.manu_code) = 3 
    AND (pt.description LIKE '%tennis%' OR pt.description LIKE '%ball%')
GROUP BY m.manu_code, m.manu_name
HAVING SUM(i.unit_price * i.quantity) > (
    SELECT SUM(unit_price * quantity) / COUNT(DISTINCT manu_code)
    FROM items
)
ORDER BY monto_total DESC;
```


## Ejercicio 3
Crear una vista que devuelva
Para cada cliente mostrar (customer_num, lname, company), cantidad de órdenes de compra, fecha de su última OC, monto total comprado y el total general comprado por todos los clientes. De los clientes que posean órdenes sólo se podrán mostrar los clientes que tengan alguna orden que posea productos que son fabricados por más de dos fabricantes y que tengan al menos 3 órdenes de compra. Ordenar el reporte de tal forma que primero aparezcan los clientes que tengan órdenes por cantidad de órdenes descendente y luego los clientes que no tengan órdenes. No se permite utilizar funciones, ni tablas temporales.

```sql
ALTER VIEW ordenesDeCliente AS
SELECT 
    c.customer_num, c.lname, c.company, 
    COUNT(DISTINCT o.order_num) cantidad_ordenes, MAX(o.order_date) fecha_ultima_oc, 
	SUM(i.quantity * i.unit_price) total_cliente, (
        SELECT SUM(quantity * unit_price) FROM items
    ) total_general
FROM customer c 
    INNER JOIN orders o ON o.customer_num = c.customer_num
    INNER JOIN items i ON i.order_num = o.order_num
WHERE i.stock_num IN (
    SELECT stock_num 
    FROM products p
    GROUP BY stock_num
    HAVING COUNT(stock_num) > 2
)
GROUP BY c.customer_num, c.lname, c.company, i.stock_num
HAVING COUNT(DISTINCT o.order_num) >= 3
UNION ALL
SELECT 
    c.customer_num, c.lname, c.company, null cantidad_ordenes, 
    null fecha_ultima_oc, null total_cliente, (
	SELECT SUM(quantity * unit_price) FROM items
) total_general
FROM customer c 
WHERE customer_num NOT IN (SELECT DISTINCT customer_num FROM orders);

SELECT * FROM ordenesDeCliente
ORDER BY cantidad_ordenes DESC;
```


## Ejercicio 4
Crear una consulta que devuelva los 5 primeros estados y el tipo de producto (description) más comprado en ese estado (state) según la cantidad vendida del tipo de producto. Ordenarlo por la cantidad vendida en forma descendente. Nota: No se permite utilizar funciones, ni tablas temporales.

```sql
SELECT TOP 5 c.state, pt.description, SUM(i.quantity) AS cantidad_vendida
FROM products p
    INNER JOIN product_types pt ON pt.stock_num = p.stock_num
    INNER JOIN items i ON i.stock_num = p.stock_num
    INNER JOIN orders o ON o.order_num = i.order_num
    INNER JOIN customer c ON c.customer_num = o.customer_num
GROUP BY c.state, pt.description
HAVING SUM(i.quantity) = (
    SELECT TOP 1 SUM(i2.quantity)
    FROM products p2
        INNER JOIN product_types pt2 ON pt2.stock_num = p2.stock_num
        INNER JOIN items i2 ON i2.stock_num = p2.stock_num
        INNER JOIN orders o2 ON o2.order_num = i2.order_num
        INNER JOIN customer c2 ON c2.customer_num = o2.customer_num
    WHERE c2.state = c.state
    GROUP BY pt2.description
    ORDER BY SUM(i2.quantity) DESC
)
ORDER BY cantidad_vendida DESC;
```


## Ejercicio 5
Listar los customers que no posean órdenes de compra y aquellos cuyas últimas órdenes de compra superen el promedio de todas las anteriores. Mostrar customer_num, fname, lname, paid_date y el monto total de la orden que supere el promedio de las anteriores. Ordenar el resultado por monto total en forma descendiente.

```sql
WITH UltimasOrdenes AS (
    SELECT 
		c.customer_num, c.fname, c.lname, o.paid_date, 
		SUM(i.quantity * i.unit_price) monto_total,
        ROW_NUMBER() OVER (PARTITION BY c.customer_num ORDER BY o.paid_date DESC) rownumber
    FROM customer c
        LEFT JOIN orders o ON o.customer_num = c.customer_num
        LEFT JOIN items i ON i.order_num = o.order_num
    GROUP BY c.customer_num, c.fname, c.lname, o.paid_date
), PromedioOrdenes AS (
    SELECT AVG(monto_total) promedio
    FROM (
		SELECT SUM(i.quantity * i.unit_price) monto_total
        FROM orders o INNER JOIN items i ON i.order_num = o.order_num
        GROUP BY o.order_num
    ) totales
)
SELECT u.customer_num, u.fname, u.lname, u.paid_date, u.monto_total 
FROM UltimasOrdenes u INNER JOIN PromedioOrdenes p ON u.monto_total > p.promedio
WHERE u.rownumber = 1 OR u.customer_num NOT IN (SELECT customer_num FROM orders)
ORDER BY u.monto_total DESC;
```


## Ejercicio 6
Se desean saber los fabricantes que vendieron mayor cantidad de un mismo producto que la competencia según la cantidad vendida. Tener en cuenta que puede existir un producto que no sea fabricado por ningún otro fabricante y que puede haber varios fabricantes que tengan la misma cantidad máxima vendida. Mostrar el código del producto, descripción del producto, código de fabricante, cantidad vendida, monto total vendido. Ordenar el resultado código de producto, por cantidad total vendida y por monto total, ambos en forma decreciente. Nota: No se permiten utilizar funciones, ni tablas temporales.

```sql
SELECT p.stock_num, pt.description, p.manu_code, SUM(i.quantity) cantidad_vendida, SUM(i.quantity * i.unit_price) monto_total
FROM products p
    INNER JOIN product_types pt ON pt.stock_num = p.stock_num
    INNER JOIN items i ON i.stock_num = p.stock_num
GROUP BY p.stock_num, pt.description, p.manu_code
HAVING SUM(i.quantity) = (
    SELECT MAX(cantidad)
    FROM (
		SELECT SUM(quantity) cantidad 
		FROM items i
		WHERE manu_code = p.manu_code AND i.stock_num = p.stock_num
		GROUP BY manu_code
	) subquery
)
ORDER BY p.stock_num, cantidad_vendida DESC, monto_total DESC;
```