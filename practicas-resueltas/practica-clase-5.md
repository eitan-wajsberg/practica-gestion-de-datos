# Práctica - Clase 5

Práctica realizada el día 11 de Septiembre del 2024 para la quinta clase de Gestión de Datos.

## Ejercicio 1
Obtener el número de cliente, la compañía, y número de orden de todos los clientes que tengan órdenes. Ordenar el resultado por número de cliente.

```sql
SELECT c.customer_num, c.company, o.order_num
FROM orders o 
    INNER JOIN customer c ON (c.customer_num = o.customer_num)
ORDER BY customer_num;
```

## Ejercicio 2
Listar los ítems de la orden número 1004, incluyendo una descripción de cada uno. El listado debe
contener: Número de orden (order_num), Número de Item (item_num), Descripción del producto (product_types.description), Código del fabricante (manu_code), Cantidad (quantity), Precio total (unit_price*quantity).

```sql
SELECT i.order_num, i.item_num, pt.description, i.manu_code, i.quantity, (i.unit_price*i.quantity) precio_total   
FROM items i
    INNER JOIN product_types pt ON (pt.stock_num = i.stock_num)
WHERE i.order_num = 1004;
```

## Ejercicio 3
Listar los items de la orden número 1004, incluyendo una descripción de cada uno. El listado debe contener: Número de orden (order_num), Número de Item (item_num), Descripción del Producto (product_types.description), Código del fabricante (manu_code), Cantidad (quantity), precio total (unit_price*quantity) y Nombre del fabricante (manu_name).

```sql
SELECT i.order_num, i.item_num, pt.description, i.manu_code, i.quantity, (i.unit_price*i.quantity) precio_total, manu_name
FROM items i
    INNER JOIN product_types pt ON (pt.stock_num = i.stock_num)
    INNER JOIN manufact m ON (m.manu_code = i.manu_code)
WHERE i.order_num = 1004;
```

## Ejercicio 4
Se desea listar todos los clientes que posean órdenes de compra. Los datos a listar son los siguientes: número de orden, número de cliente, nombre, apellido y compañía.

```sql
SELECT DISTINCT o.order_num, c.customer_num, c.fname, c.lname, c.company
FROM customer c
    INNER JOIN orders o ON(o.customer_num = c.customer_num); 
```

## Ejercicio 5
Se desea listar todos los clientes que posean órdenes de compra. Los datos a listar son los siguientes: número de cliente, nombre, apellido y compañía. Se requiere sólo una fila por cliente.

```sql
SELECT DISTINCT c.customer_num, c.fname, c.lname, c.company
FROM customer c
INNER JOIN orders o ON(o.customer_num = c.customer_num);
```

## Ejercicio 6
Se requiere listar para armar una nueva lista de precios los siguientes datos: nombre del fabricante (manu_name), número de stock (stock_num), descripción (product_types.description), unidad (units.unit), precio unitario (unit_price) y Precio Junio (precio unitario + 20%).

```sql
SELECT m.manu_name, p.stock_num, pt.description, u.unit, p.unit_price, (p.unit_price + 0.2*p.unit_price) precio_junio
FROM products p
    INNER JOIN units u ON (u.unit_code = p.unit_code)
    INNER JOIN manufact m ON (m.manu_code = p.manu_code)
    INNER JOIN product_types pt ON (pt.stock_num = p.stock_num);
```

## Ejercicio 7
Se requiere un listado de los items de la orden de pedido Nro. 1004 con los siguientes datos: Número de item (item_num), descripción de cada producto (product_types.description), cantidad (quantity) y precio total (unit_price*quantity).

```sql
SELECT i.item_num, pt.description, i.quantity, (i.unit_price*i.quantity) precio_total
FROM items i
    INNER JOIN product_types pt ON (pt.stock_num = i.stock_num)
WHERE i.order_num = 1004;
```

## Ejercicio 8
Informar el nombre del fabricante (manu_name) y el tiempo de envío (lead_time) de los ítems de las Órdenes del cliente 104.

```sql
SELECT m.manu_name, m.lead_time
FROM items i
    INNER JOIN manufact m ON(m.manu_code = i.manu_code)
    INNER JOIN orders o ON(o.order_num = i.order_num)
WHERE o.customer_num = 104;
```

## Ejercicio 9
Se requiere un listado de las todas las órdenes de pedido con los siguientes datos: Número de orden (order_num), fecha de la orden (order_date), número de ítem (item_num), descripción de cada producto (description), cantidad (quantity) y precio total (unit_price*quantity).

```sql
SELECT o.order_num, o.order_date, i.item_num, pt.description, i.quantity, (i.unit_price*i.quantity) precio_total
FROM orders o
    INNER JOIN items i ON(i.order_num = o.order_num)
    INNER JOIN product_types pt ON(pt.stock_num = i.stock_num);
```

## Ejercicio 10
Obtener un listado con la siguiente información: Apellido (lname) y Nombre (fname) del Cliente separado por coma, Número de teléfono (phone) en formato (999) 999-9999. Ordenado por apellido y nombre.

```sql
SELECT lname + ',' + fname apellidoYNombre, '(' + SUBSTRING(phone,1,3)+')'+ ' ' + SUBSTRING(phone,5,8) telefono_formateado
FROM customer
ORDER BY apellidoYNombre;
```

## Ejercicio 11
Obtener la fecha de embarque (ship_date), Apellido (lname) y Nombre (fname) del Cliente separado por coma y la cantidad de órdenes del cliente. Para aquellos clientes que viven en el estado con descripción (sname) “California” y el código postal está entre 94000 y 94100 inclusive. Ordenado por fecha de embarque y, Apellido y nombre.

```sql
SELECT c.lname + ',' + c.fname apellidoYNombre, COUNT(o.order_num) cantidad_ordenes
FROM customer c
    INNER JOIN orders o ON(o.customer_num = c.customer_num)
    INNER JOIN state s ON(s.state = c.state)
WHERE s.sname='California' AND c.zipcode BETWEEN 94000 AND 94100
GROUP BY o.ship_date, c.lname, c.fname
ORDER BY apellidoYNombre;

-- No tengo en cuenta la ship_date ya que al ponerla surge un problema todavia no resoluble con los contenidos dados hasta el momento. Al incorporar la ship_date deberias agrupar por dos columnas.
```

## Ejercicio 12
Obtener por cada fabricante (manu_name) y producto (description), la cantidad vendida y el Monto Total vendido (unit_price * quantity). Sólo se deberán mostrar los ítems de los fabricantes ANZ, HRO, HSK y SMT, para las órdenes correspondientes a los meses de mayo y junio del 2015. Ordenar el resultado por el monto total vendido de mayor a menor.

```sql
SELECT m.manu_name, pt.description, COUNT(i.quantity) cantidad_vendida, SUM(i.unit_price * i.quantity) monto_total_vendido
FROM items i
    INNER JOIN manufact m ON(m.manu_code = i.manu_code)
    INNER JOIN product_types pt ON(pt.stock_num = i.stock_num)
    INNER JOIN orders o ON(o.order_num = i.order_num)
WHERE m.manu_code IN('ANZ', 'HRO', 'HSK', 'SMT') AND order_date BETWEEN '2015-05-01' AND '2015-06-30'
GROUP BY m.manu_name, pt.description
ORDER BY monto_total_vendido DESC
```
Acordate los siguiente:
- Usa `WHERE` cuando los filtros se aplican a las filas individuales antes de la agrupación.
- Usa `HAVING` cuando necesitas filtrar resultados después de la agrupación, especialmente si filtras basándote en funciones agregadas.

## Ejercicio 13
Emitir un reporte con la cantidad de unidades vendidas y el importe total por mes de productos, ordenado por importe total en forma descendente. Formato: Año/Mes, Cantidad, Monto_Total

```sql
SELECT FORMAT(o.order_date, 'yyyy/MM'), SUM(i.quantity) cantidad_unidades_vendidas, SUM(quantity*unit_price) importe_total
FROM items i
    INNER JOIN orders o ON(o.order_num = i.order_num)
GROUP BY FORMAT(o.order_date, 'yyyy/MM')
ORDER BY importe_total DESC;
```