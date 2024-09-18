# Práctica - Clase 6

Práctica realizada el día 17 de Septiembre del 2024 para la sexta clase de Gestión de Datos.

## Ejercicio 1
Mostrar el Código del fabricante, nombre del fabricante, tiempo de entrega y monto Total de productos vendidos, ordenado por nombre de fabricante. En caso que el fabricante no tenga ventas, mostrar el total en NULO.

```sql
SELECT m.manu_code, m.manu_name, m.lead_time, SUM(i.quantity * i.unit_price) monto_total
FROM manufact m
LEFT JOIN items i ON(i.manu_code = m.manu_code)
GROUP BY m.manu_code, m.manu_name, m.lead_time
ORDER BY m.manu_name;
```

## Ejercicio 2
Mostrar en una lista de a pares, el código y descripción del producto, y los pares de fabricantes que fabriquen el mismo producto. En el caso que haya un único fabricante deberá mostrar el Código de fabricante 2 en nulo. Ordenar el resultado por código de producto. El listado debe tener el siguiente formato:
Nro. de Producto Descripcion Cód. de fabric. 1 Cód. de fabric. 2
(stock_num) (Description) (manu_code) (manu_code)

REVISAR!
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
a) En primer lugar, escribir una consulta usando una subconsulta.
b) Reescribir la consulta utilizando GROUP BY y HAVING.
La consulta deberá tener el siguiente formato:
Número_de_Cliente Nombre Apellido
(customer_num) (fname) (lname)

```sql
-- a)
SELECT c.customer_num, c.fname, c.lname
FROM customer c
WHERE (SELECT COUNT(*) FROM orders o WHERE o.customer_num = c.customer_num) > 1;

-- b)
SELECT c.customer_num, c.fname, c.lname
FROM orders o
INNER JOIN customer c ON(c.customer_num = o.customer_num)
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
HAVING SUM(unit_price*quantity) < (SELECT AVG(i2.quantity* i2.unit_price) FROM items i2); 
```

## Ejercicio 5
Obtener por cada fabricante, el listado de todos los productos de stock con precio
unitario (unit_price) mayor que el precio unitario promedio de dicho fabricante.
Los campos de salida serán: manu_code, manu_name, stock_num, description, unit_price.

```sql
SELECT m.manu_code, m.manu_name, p.stock_num, pt.description, p.unit_price
FROM products p 
INNER JOIN
```

## Ejercicio 6
Usando el operador NOT EXISTS listar la información de órdenes de compra que NO
incluyan ningún producto que contenga en su descripción el string ‘ baseball gloves’. Ordenar el resultado por compañía del cliente ascendente y número de orden
descendente. El formato de salida deberá ser:
Número de Cliente Compañía Número de Orden Fecha de la Orden
(customer_num) (company) (order_num) (order_date)

```sql
```

## Ejercicio 7
Obtener el número, nombre y apellido de los clientes que NO hayan comprado productos del fabricante ‘HSK’.

```sql
```

## Ejercicio 8
Obtener el número, nombre y apellido de los clientes que hayan comprado TODOS los productos del fabricante ‘HSK’.

```sql
```

## Ejercicio 9
Reescribir la siguiente consulta utilizando el operador UNION:
SELECT * FROM products WHERE manu_code = ‘HRO’ OR stock_num = 1

```sql
```

## Ejercicio 10
Desarrollar una consulta que devuelva las ciudades y compañías de todos los Clientes ordenadas alfabéticamente por Ciudad pero en la consulta deberán aparecer primero las compañías situadas en Redwood City y luego las demás.
Formato: Clave de ordenamiento Ciudad Compañía
(sortkey) (city) (company)

```sql
```

## Ejercicio 11
Desarrollar una consulta que devuelva los dos tipos de productos más vendidos y los dos menos vendidos en función de las unidades totales vendidas.
Formato

```sql
```


## Ejercicio 12
Crear una Vista llamada ClientesConMultiplesOrdenes basada en la consulta realizada en el punto 3.b con los nombres de atributos solicitados en dicho punto.

```sql
```

## Ejercicio 13
Crear una Vista llamada Productos_HRO en base a la consulta
SELECT * FROM products WHERE manu_code = “HRO” La vista deberá restringir la posibilidad de insertar datos que no cumplan con su criterio de
selección.
a. Realizar un INSERT de un Producto con manu_code=’ANZ’ y stock_num=303. Qué sucede?
b. Realizar un INSERT con manu_code=’HRO’ y stock_num=303. Qué sucede?
c. Validar los datos insertados a través de la vista.

```sql
```
