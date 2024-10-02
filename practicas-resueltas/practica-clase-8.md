# Práctica - Clase 8

Práctica realizada el día 2 de Octubre del 2024 para la octava clase de Gestión de Datos.

## Práctica de Triggers I

### Ejercicio 1
Dada la tabla Products de la base de datos stores7 se requiere crear una tabla Products_historia_precios y crear un trigger que registre los cambios de precios que se hayan producido en la tabla Products. Tabla Products_historia_precios
- Stock_historia_Id Identity (PK)
- Stock_num
- Manu_code
- fechaHora (grabar fecha y hora del evento)
- usuario (grabar usuario que realiza el cambio de precios)
- unit_price_old
- unit_price_new
- estado char default ‘A’ check (estado IN (‘A’,’I’))

```sql
CREATE TRIGGER registroCambioPrecio
ON products
AFTER UPDATE
AS
BEGIN
INSERT INTO products_historia_precios (stock_num, manu_code, fechaHora, usuario, unit_price_old, unit_price_new, estado)
	SELECT i.stock_num, i.manu_code, GETDATE(), CURRENT_USER, d.unit_price, i.unit_price, i.status 
	FROM inserted i INNER JOIN deleted d ON(i.stock_num = d.stock_num AND i.manu_code = d.manu_code)
	WHERE i.unit_price != d.unit_price;
END;

-- Prueba
UPDATE products SET unit_price=1000 WHERE stock_num=1
SELECT * FROM products_historia_precios;
SELECT * FROM products WHERE stock_num=1;
```

### Ejercicio 2
Crear un trigger sobre la tabla Products_historia_precios que ante un delete sobre la misma realice en su lugar un update del campo estado de ‘A’ a ‘I’ (inactivo).

```sql

```

### Ejercicio 3
Validar que sólo se puedan hacer inserts en la tabla Products en un horario entre las 8:00 AM y 8:00 PM. En caso contrario enviar un error por pantalla.

```sql

```

### Ejercicio 4
Crear un trigger que ante un borrado sobre la tabla ORDERS realice un borrado en cascada sobre la tabla ITEMS, validando que sólo se borre 1 orden de compra.
Si detecta que están queriendo borrar más de una orden de compra, informará un error y
abortará la operación.

```sql

```

### Ejercicio 5
Crear un trigger de insert sobre la tabla ítems que al detectar que el código de fabricante (manu_code) del producto a comprar no existe en la tabla manufact, inserte una fila en dicha tabla con el manu_code ingresado, en el campo manu_name la descripción ‘Manu Orden 999’ donde 999 corresponde al nro. de la orden de compra a la que pertenece el ítem y en el campo lead_time el valor 1.

```sql

```

### Ejercicio 6
Crear tres triggers (Insert, Update y Delete) sobre la tabla Products para replicar todas las operaciones en la tabla Products _replica, la misma deberá tener la misma estructura de la tabla Products.

```sql

```

### Ejercicio 7
Crear la vista Productos_x_fabricante que tenga los siguientes atributos: Stock_num, description, manu_code, manu_name, unit_price
Crear un trigger de Insert sobre la vista anterior que ante un insert, inserte una fila en la tabla Products, pero si el manu_code no existe en la tabla manufact, inserte además una fila en dicha tabla con el campo lead_time en 1.

```sql

```

## Práctica de Triggers II

### Ejercicio 1
Se pide: Crear un trigger que valide que ante un insert de una o más filas en la tabla ítems, realice la siguiente validación:
- Si la orden de compra a la que pertenecen los ítems ingresados corresponde a clientes del estado de California, se deberá validar que estas órdenes puedan tener como máximo 5 registros en la tabla ítem.
- Si se insertan más ítems de los definidos, el resto de los ítems se deberán insertar en la tabla items_error la cual contiene la misma estructura que la tabla ítems más
un atributo fecha que deberá contener la fecha del día en que se trató de insertar.

Ej. Si la Orden de Compra tiene 3 items y se realiza un insert masivo de 3 ítems más, el trigger deberá insertar los 2 primeros en la tabla ítems y el restante en la tabla ítems_error. Supuesto: En el caso de un insert masivo los items son de la misma orden.

```sql

```

### Ejercicio 2
Triggers Dada la siguiente vista:
```sql
CREATE VIEW ProdPorFabricante AS
SELECT m.manu_code, m.manu_name, COUNT(*)
FROM manufact m INNER JOIN products p
ON (m.manu_code = p.manu_code)
GROUP BY manu_code, manu_name;
```
Crear un trigger que permita ante un insert en la vista ProdPorFabricante insertar una fila
en la tabla manufact. Observaciones: el atributo leadtime deberá insertarse con un valor default 10. El trigger deberá contemplar inserts de varias filas, por ej. ante un
`INSERT` / `SELECT`.

```sql

```

### Ejercicio 3
Crear un trigger que ante un INSERT o UPDATE de una o más filas de la tabla Customer, realice la siguiente validación.
- La cuota de clientes correspondientes al estado de California es de 20, si se supera dicha
cuota se deberán grabar el resto de los clientes en la tabla customer_pend.
- Validar que si de los clientes a modificar se modifica el Estado, no se puede superar dicha
cuota.

Si por ejemplo el estado de CA cuenta con 18 clientes y se realiza un update o insert masivo de 5 clientes con estado de CA, el trigger deberá modificar los 2 primeros en la tabla customer y los restantes grabarlos en la tabla customer_pend. La tabla customer_pend tendrá la misma estructura que la tabla customer con un atributo adicional
fechaHora que deberá actualizarse con la fecha y hora del día.

```sql

```

### Ejercicio 4
Dada la siguiente vista:
```sql
CREATE VIEW ProdPorFabricanteDet AS
SELECT m.manu_code, m.manu_name, pt.stock_num, pt.description
FROM manufact m LEFT OUTER JOIN products p ON m.manu_code = p.manu_code
LEFT OUTER JOIN product_types pt ON p.stock_num = pt.stock_num;
```
Se pide: Crear un trigger que permita ante un `DELETE` en la vista ProdPorFabricante borrar los datos en la tabla manufact pero sólo de los fabricantes cuyo campo description sea NULO (o sea que no tienen stock).
Observaciones: El trigger deberá contemplar borrado de varias filas mediante un `DELETE` masivo. En ese caso sólo borrará de la tabla los fabricantes que no tengan productos en stock, borrando los demás.

```sql

```

### Ejercicio 5
Se pide crear un trigger que permita ante un delete de una sola fila en la vista ordenesPendientes valide:
- Si el cliente asociado a la orden tiene sólo esa orden pendiente de pago (paid_date IS NULL), no permita realizar la Baja, informando el error.
- Si la Orden tiene más de un ítem asociado, no permitir realizar la Baja, informando el error.
- Ante cualquier otra condición borrar la Orden con sus ítems asociados, respetando la integridad referencial.

Estructura de la vista: customer_num, fname, lname, Company, order_num, order_date WHERE paid_date IS NULL.

```sql

```
