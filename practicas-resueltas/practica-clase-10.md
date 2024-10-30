# Práctica - Clase 10

Práctica realizada el día 23 de Octubre del 2024 para la decima clase de Gestión de Datos.

## Ejercicio 1
Crear una vista que devuelva:
- Código y Nombre (manu_code,manu_name) de los fabricante, posean o no productos (en tabla Products), cantidad de productos que fabrican (cant_producto) y la fecha de la última OC que contenga un producto suyo (ult_fecha_orden). De los fabricantes que fabriquen productos sólo se podrán mostrar los que fabriquen más de 2 productos.  No se permite utilizar funciones definidas por usuario, ni tablas temporales, ni UNION.
- Realizar una consulta sobre la vista que devuelva manu_code, manu_name, cant_producto y si el campo ult_fecha_orden posee un NULL informar ‘No Posee Órdenes’ si no posee NULL informar el valor de dicho campo. No se puede utilizar UNION para el SELECT.

```sql
```

## Ejercicio 2
Desarrollar una consulta ABC de fabricantes que:
Liste el código y nombre del fabricante, la cantidad de órdenes de compra que contengan sus productos y la monto total de los productos vendidos. Mostrar sólo los fabricantes cuyo código comience con A ó con N y posea 3 letras, y los productos cuya descripción posean el string “tennis” ó el string “ball” en cualquier parte del nombre y cuyo monto total vendido sea mayor que el total de ventas promedio de todos los fabricantes (Cantidad * precio unitario / Cantidad de fabricantes que vendieron sus productos). Mostrar los registros ordenados por monto total vendido de mayor a menor.

```sql
```


## Ejercicio 3
Crear una vista que devuelva
Para cada cliente mostrar (customer_num, lname, company), cantidad de órdenes de compra, fecha de su última OC, monto total comprado y el total general comprado por todos los clientes. De los clientes que posean órdenes sólo se podrán mostrar los clientes que tengan alguna orden que posea productos que son fabricados por más de dos fabricantes y que tengan al menos 3 órdenes de compra. Ordenar el reporte de tal forma que primero aparezcan los clientes que tengan órdenes por cantidad de órdenes descendente y luego los clientes que no tengan órdenes. No se permite utilizar funciones, ni tablas temporales.

```sql
```


## Ejercicio 4
Crear una consulta que devuelva los 5 primeros estados y el tipo de producto (description) más comprado en ese estado (state) según la cantidad vendida del tipo de producto. Ordenarlo por la cantidad vendida en forma descendente. Nota: No se permite utilizar funciones, ni tablas temporales.

```sql
```


## Ejercicio 5
Crear una vista que devuelva:
- Código y Nombre (manu_code,manu_name) de los fabricante, posean o no productos (en tabla Products), cantidad de productos que fabrican (cant_producto) y la fecha de la última OC que contenga un producto suyo (ult_fecha_orden). De los fabricantes que fabriquen productos sólo se podrán mostrar los que fabriquen más de 2 productos. No se permite utilizar funciones definidas por usuario, ni tablas temporales, ni UNION.
- Realizar una consulta sobre la vista que devuelva manu_code, manu_name, cant_producto y si el campo ult_fecha_orden posee un NULL informar ‘No Posee Órdenes’ si no posee NULL informar el valor de dicho campo.  No se puede utilizar UNION para el SELECT.

```sql
```


## Ejercicio 6
Crear una vista que devuelva:
- Código y Nombre (manu_code,manu_name) de los fabricante, posean o no productos (en tabla Products), cantidad de productos que fabrican (cant_producto) y la fecha de la última OC que contenga un producto suyo (ult_fecha_orden). De los fabricantes que fabriquen productos sólo se podrán mostrar los que fabriquen más de 2 productos.  No se permite utilizar funciones definidas por usuario, ni tablas temporales, ni UNION.
- Realizar una consulta sobre la vista que devuelva manu_code, manu_name, cant_producto y si el campo ult_fecha_orden posee un NULL informar ‘No Posee Órdenes’ si no posee NULL informar el valor de dicho campo.  No se puede utilizar UNION para el SELECT.

```sql
```