# Práctica - Clase 6

Práctica realizada el día 17 de Septiembre del 2024 para la sexta clase de Gestión de Datos.

## Ejercicio 1
Mostrar el Código del fabricante, nombre del fabricante, tiempo de entrega y monto Total de productos vendidos, ordenado por nombre de fabricante. En caso que el fabricante no tenga ventas, mostrar el total en NULO.

```sql
SELECT m.manu_code, m.manu_name, m.lead_time, SUM(i.quantity * i.unit_price) monto_total
FROM manufact m LEFT JOIN items i ON(i.manu_code = m.manu_code)
GROUP BY m.manu_code, m.manu_name, m.lead_time
ORDER BY m.manu_name;
```

## Ejercicio 2
Mostrar en una lista de a pares, el código y descripción del producto, y los pares de fabricantes que fabriquen el mismo producto. En el caso que haya un único fabricante deberá mostrar el Código de fabricante 2 en nulo. Ordenar el resultado por código de producto. El listado debe tener el siguiente formato:
Nro. de Producto Descripcion Cód. de fabric. 1 Cód. de fabric. 2
(stock_num) (Description) (manu_code) (manu_code)

```sql
SELECT p1.stock_num, pt.description, p1.manu_code cod_fabricante_1, p2.manu_code cod_fabricante_2
FROM products p1
    LEFT JOIN products p2 ON(p1.stock_num = p2.stock_num AND p1.manu_code != p2.manu_code)
    INNER JOIN product_types pt ON(pt.stock_num = p1.stock_num)
WHERE p1.manu_code < p2.manu_code OR p2.manu_code IS NULL
ORDER BY p1.stock_num;
```

## Ejercicio 3
Listar todos los clientes que hayan tenido más de una orden.
1. En primer lugar, escribir una consulta usando una subconsulta.
2. Reescribir la consulta utilizando GROUP BY y HAVING.
La consulta deberá tener el siguiente formato:
Número_de_Cliente Nombre Apellido
(customer_num) (fname) (lname)

```sql
-- 1)
SELECT c.customer_num, c.fname, c.lname
FROM customer c
WHERE (
    SELECT COUNT(*) 
    FROM orders o 
    WHERE o.customer_num = c.customer_num
) > 1;

-- 2)
SELECT c.customer_num, c.fname, c.lname
FROM orders o INNER JOIN customer c ON(c.customer_num = o.customer_num)
GROUP BY c.customer_num, c.fname, c.lname
HAVING COUNT(o.customer_num) > 1;
```

## Ejercicio 4
Seleccionar todas las Órdenes de compra cuyo Monto total (Suma de p x q de sus items) sea menor al precio total promedio (avg p x q) de todas las líneas de las ordenes.
Formato de la salida: Nro. de Orden Total
(order_num) (suma)

```sql
SELECT i1.order_num, SUM(i1.unit_price*i1.quantity) monto_total
FROM items i1 
GROUP BY order_num
HAVING SUM(unit_price*quantity) < (
    SELECT AVG(i2.quantity* i2.unit_price) 
    FROM items i2
); 
```

## Ejercicio 5
Obtener por cada fabricante, el listado de todos los productos de stock conprecio unitario (unit_price) mayor que el precio unitario promedio de dicho fabricante. Los campos de salida serán: manu_code, manu_name, stock_num, description, unit_price.

```sql
SELECT m.manu_code, m.manu_name, p1.stock_num, pt.description, p1.unit_price
FROM products p1 
    INNER JOIN manufact m ON(m.manu_code = p1.manu_code)
    INNER JOIN product_types pt ON(pt.stock_num = p1.stock_num)
WHERE unit_price > (
    SELECT AVG(unit_price) 
    FROM products p2 
    WHERE p1.manu_code = p2.manu_code
);
```

## Ejercicio 6
Usando el operador NOT EXISTS listar la información de órdenes de compra que NO incluyan ningún producto que contenga en su descripción el string ‘ baseball gloves’. Ordenar el resultado por compañía del cliente ascendente y número de orden descendente. El formato de salida deberá ser:
Número de Cliente Compañía Número de Orden Fecha de la Orden
(customer_num) (company) (order_num) (order_date)

```sql
SELECT c.customer_num, c.company, o.order_num, o.order_date
FROM orders o INNER JOIN customer c ON(c.customer_num = o.customer_num)
WHERE NOT EXISTS (
    SELECT pt.description 
    FROM product_types pt INNER JOIN items i ON(i.order_num = o.order_num)
    WHERE pt.description LIKE '%baseball gloves%' AND pt.stock_num = i.stock_num
)
ORDER BY c.company ASC, o.order_num DESC;
```

## Ejercicio 7
Obtener el número, nombre y apellido de los clientes que NO hayan comprado productos del fabricante ‘HSK’.

```sql
SELECT c.customer_num, c.fname, c.lname
FROM customer c 
WHERE NOT EXISTS (
    SELECT i.manu_code, o.customer_num
    FROM orders o INNER JOIN items i ON(i.order_num = o.order_num) 
    WHERE manu_code = 'HSK' AND o.customer_num = c.customer_num
);
```

## Ejercicio 8
Obtener el número, nombre y apellido de los clientes que hayan comprado TODOS los productos del fabricante ‘HSK’.
REVISAR!!!
```sql
SELECT c.customer_num, c.fname, c.lname
FROM customer c 
WHERE NOT EXISTS (
    SELECT p.stock_num
    FROM products p
    WHERE p.manu_code = 'HSK' AND NOT EXISTS (
        SELECT o.order_num
        FROM orders o INNER JOIN items i ON(i.order_num = o.order_num)
        WHERE o.customer_num = c.customer_num AND i.stock_num = p.stock_num AND p.manu_code = i.manu_code
    )
);
```

## Ejercicio 9
Reescribir la siguiente consulta utilizando el operador UNION:
SELECT * FROM products WHERE manu_code = ‘HRO’ OR stock_num = 1

```sql
SELECT * FROM products WHERE manu_code = 'HRO'
UNION 
SELECT * FROM products WHERE stock_num = 1;
```

## Ejercicio 10
Desarrollar una consulta que devuelva las ciudades y compañías de todos los Clientes ordenadas alfabéticamente por Ciudad pero en la consulta deberán aparecer primero las compañías situadas en Redwood City y luego las demás. 
Formato: Clave de ordenamiento Ciudad Compañía
(sortkey) (city) (company)

```sql
SELECT city, company, 0 clave_ordenamiento
FROM customer
WHERE city = 'Redwood City'
UNION
SELECT city, company, 1 clave_ordenamiento
FROM customer
WHERE city != 'Redwood City'
ORDER BY clave_ordenamiento, city, company;
```

## Ejercicio 11
Desarrollar una consulta que devuelva los dos tipos de productos más vendidos y los dos menos vendidos en función de las unidades totales vendidas.
Formato

```sql
SELECT i1.stock_num, SUM(i1.quantity) unidades_totales
FROM items i1
WHERE i1.stock_num IN (
    SELECT TOP 2 i2.stock_num
    FROM items i2 
    GROUP BY i2.stock_num
    ORDER BY SUM(i2.quantity) DESC
)
GROUP BY i1.stock_num
UNION
SELECT i1.stock_num, SUM(i1.quantity) unidades_totales
FROM items i1
WHERE i1.stock_num IN (
    SELECT TOP 2 i2.stock_num
    FROM items i2 
    GROUP BY i2.stock_num
    ORDER BY SUM(i2.quantity) ASC
)
GROUP BY i1.stock_num
ORDER BY SUM(i1.quantity) DESC;
```


## Ejercicio 12
Crear una Vista llamada ClientesConMultiplesOrdenes basada en la consulta realizada en el punto 3.b con los nombres de atributos solicitados en dicho punto.

```sql
CREATE VIEW ClientesConMultiplesOrdenes 
(nro_cliente, nombre, apellido) AS 
SELECT c.customer_num, c.fname, c.lname
FROM orders o INNER JOIN customer c ON(c.customer_num = o.customer_num)
GROUP BY c.customer_num, c.fname, c.lname
HAVING COUNT(o.customer_num) > 1;
```

## Ejercicio 13
Crear una Vista llamada Productos_HRO en base a la consulta SELECT * FROM products WHERE manu_code = “HRO”. La vista deberá restringir la posibilidad de insertar datos que no cumplan con su criterio de selección.
a. Realizar un INSERT de un Producto con manu_code=’ANZ’ y stock_num=303. Qué sucede?
b. Realizar un INSERT con manu_code=’HRO’ y stock_num=303. Qué sucede?
c. Validar los datos insertados a través de la vista.

```sql
CREATE VIEW Productos_HRO AS
SELECT * FROM products WHERE manu_code = 'HRO'
WITH CHECK OPTION;

-- a) Realizar el INSERT no es posible, ya que se utilizo la clausula WITH CHECK OPTION. Error: The attempted insert or update failed because the target view either specifies WITH CHECK OPTION or spans a view that specifies WITH CHECK OPTION and one or more rows resulting from the operation did not qualify under the CHECK OPTION constraint.

-- b) Realizar el INSERT es posible y se inserta en la tabla de productos y puede ser verificado tambien en la vista.
```

## Ejercicio 14
Escriba una transacción que incluya las siguientes acciones:
1. BEGIN TRANSACTION.
    - Insertar un nuevo cliente llamado “Fred Flintstone” en la tabla de clientes (customer).
    - Seleccionar todos los clientes llamados Fred de la tabla de clientes (customer).
2. ROLLBACK TRANSACTION.

Luego volver a ejecutar la consulta:
1. Seleccionar todos los clientes llamados Fred de la tabla de clientes (customer).
2. Completado el ejercicio descripto arriba. Observar que los resultados del segundo SELECT difieren con respecto al primero.

```sql
BEGIN TRANSACTION
INSERT INTO customer (customer_num, fname, lname) VALUES (129, 'Fred', 'Flintstone');
SELECT * 
FROM customer 
WHERE fname = 'Fred';
ROLLBACK TRANSACTION;

SELECT * 
FROM customer 
WHERE fname = 'Fred';
-- Los dos selects difieren. En el primero figura el registro creado de Fred Flintstone, pero en el segundo no, ya que se hizo rollback.
```

## Ejercicio 15
Se ha decidido crear un nuevo fabricante AZZ, quién proveerá parte de los mismos productos que provee el fabricante ANZ, los productos serán los que contengan el string ‘tennis’ en su descripción.
1. Agregar las nuevas filas en la tabla manufact y la tabla products.
2. El código del nuevo fabricante será “AZZ”, el nombre de la compañía “AZZIO SA” y el tiempo de envío será de 5 días (lead_time).
3. La información del nuevo fabricante “AZZ” de la tabla Products será la misma que la del fabricante “ANZ” pero sólo para los productos que contengan 'tennis' en su descripción.
4. Tener en cuenta las restricciones de integridad referencial existentes, manejar todo dentro de una misma transacción.

```sql
BEGIN TRANSACTION
INSERT INTO manufact (manu_code, manu_name, lead_time) VALUES ('AZZ', 'AZZIO SA', 5); 
INSERT INTO products (p.stock_num, p.manu_code, p.unit_price, p.unit_code, p.status)
    SELECT p.stock_num, 'AZZ', p.unit_price, p.unit_code, p.status
    FROM products p INNER JOIN product_types pt ON(pt.stock_num = p.stock_num)
    WHERE pt.description LIKE '%tennis%' AND p.manu_code = 'ANZ';
COMMIT;

SELECT * FROM products WHERE manu_code = 'ANZ';
SELECT * FROM products WHERE manu_code = 'AZZ';
```