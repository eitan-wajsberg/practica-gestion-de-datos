# Práctica - Clase 12

Práctica realizada el día 30 de Octubre del 2024 para la doceava clase de Gestión de Datos.

## Ejercicio 1
Listar el Número, nombre, apellido, estado, cantidad de Órdenes, monto total comprado por Cliente durante el año 2015 que no sean del estado de Florida. Mostrar sólo aquellos clientes cuyo monto total comprado sea mayor que el promedio del monto total comprado por Cliente que no sean del estado Florida. Ordenado por total comprado en forma descendente.

```sql
-- Alternativa 1
SELECT 
    customer_num, fname, lname, state, (
        SELECT COUNT(DISTINCT order_num) 
        FROM orders 
		WHERE customer_num = c.customer_num
    ) cantidad_ordenes, (
        SELECT SUM(i.quantity * i.unit_price) 
        FROM orders o 
			INNER JOIN items i ON i.order_num = o.order_num
			INNER JOIN customer c2 ON c2.customer_num = o.customer_num
        WHERE c2.customer_num = c.customer_num AND YEAR(o.order_date) = 2015 AND c2.state <> 'FL'
    ) total_comprado
FROM customer c
GROUP BY customer_num, fname, lname, state
HAVING (
	SELECT SUM(i.quantity * i.unit_price) 
	FROM orders o 
		INNER JOIN items i ON i.order_num = o.order_num
		INNER JOIN customer c2 ON c2.customer_num = o.customer_num
	WHERE c2.customer_num = c.customer_num AND YEAR(o.order_date) = 2015 AND c2.state <> 'FL'
) > (
    SELECT SUM(i2.quantity * i2.unit_price) / COUNT(DISTINCT c2.customer_num)
    FROM items i2 
        INNER JOIN orders o2 ON i2.order_num = o2.order_num 
        INNER JOIN customer c2 ON c2.customer_num = o2.customer_num
    WHERE c2.state <> 'FL'
)
ORDER BY total_comprado DESC;

-- Alternativa 2 (Mejor)
SELECT c.customer_num, c.fname, c.lname, s.sname, COUNT(DISTINCT o.order_num) cant_ordenes, SUM(i.quantity*i.unit_price) total_comprado
FROM customer c 
    LEFT JOIN state s ON c.state = s.state
    LEFT JOIN  orders o ON c.customer_num = o.customer_num
    LEFT JOIN items i ON o.order_num = i.order_num
WHERE s.sname NOT LIKE 'Florida' AND YEAR(o.order_date) = '2015'
GROUP BY c.customer_num, c.fname, c.lname, s.sname
HAVING SUM(i.quantity*i.unit_price) > (
    SELECT SUM(i2.quantity * i2.unit_price) / COUNT(DISTINCT c2.customer_num)
    FROM items i2 
        INNER JOIN orders o2 ON i2.order_num = o2.order_num 
        INNER JOIN customer c2 ON c2.customer_num = o2.customer_num
    WHERE c2.state <> 'FL'
)
ORDER BY total_comprado DESC;
```

## Ejercicio 2
Seleccionar todos los clientes cuyo monto total comprado sea mayor al de su refererente durante el año 2015. Mostrar número, nombre, apellido y los montos totales comprados de ambos durante ese año. Tener en cuenta que un cliente puede no tener referente y que el referente pudo no haber comprado nada durante el año 2015, mostrarlo igual.

```sql
SELECT  c1.customer_num, c1.fname, c1.lname, SUM(i1.quantity * i1.unit_price) as monto1, c2.customer_num, c2.fname, c2.lname, COALESCE(monto2, 0) as monto2
FROM customer c1 
	JOIN orders o1 ON o1.customer_num = c1.customer_num
	JOIN items i1 ON i1.order_num = o1.order_num
    JOIN (
		SELECT c2.customer_num, c2.fname, c2.lname, SUM(i2.quantity * i2.unit_price) as monto2
		FROM customer c2 
			JOIN orders o2 ON o2.customer_num = c2.customer_num
			JOIN items i2 ON i2.order_num = o2.order_num
		GROUP BY c2.customer_num, c2.fname, c2.lname
    ) C2 ON c2.customer_num = c1.customer_num_referedBy
WHERE YEAR(o1.order_date) = 2015
GROUP BY c1.customer_num, c1.fname, c1.lname,c2.customer_num, c2.fname, c2.lname, c2.monto2
HAVING SUM(i1.quantity * i1.unit_price) > COALESCE(c2.monto2, 0)
```
