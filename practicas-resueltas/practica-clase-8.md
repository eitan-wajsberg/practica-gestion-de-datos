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
AFTER UPDATE AS
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
Crear un trigger sobre la tabla products_historia_precios que ante un delete sobre la misma realice en su lugar un update del campo estado de ‘A’ a ‘I’ (inactivo).

```sql
CREATE TRIGGER alEliminarSettearInactividad
ON products_historia_precios
INSTEAD OF DELETE AS 
BEGIN
	DECLARE inactividadCursor CURSOR FOR (
		SELECT stock_historia_id FROM deleted
	)

	DECLARE @id INT

	OPEN inactividadCursor
	FETCH NEXT FROM inactividadCursor INTO @id

	WHILE @@FETCH_STATUS = 0
	BEGIN
		UPDATE products_historia_precios
		SET estado = 'I' 
		WHERE stock_historia_id = @id

		FETCH NEXT FROM inactividadCursor INTO @id
	END

	CLOSE inactividadCursor
	DEALLOCATE inactividadCursor
END

-- Otra solucion mas corta y sin cursor

ALTER TRIGGER alEliminarSettearInactividad
ON products_historia_precios
INSTEAD OF DELETE AS 
BEGIN
	UPDATE products_historia_precios
	SET estado = 'I'
	WHERE stock_historia_id IN (SELECT stock_historia_id FROM deleted);
END;
```

### Ejercicio 3
Validar que sólo se puedan hacer inserts en la tabla Products en un horario entre las 8:00 AM y 8:00 PM. En caso contrario enviar un error por pantalla.

```sql
ALTER TRIGGER insertsADeterminadoHorario ON products
AFTER INSERT AS
BEGIN
	IF (DATEPART(HOUR, GETDATE()) BETWEEN 8 AND 20)
	BEGIN
	    THROW 50000, 'No es posible hacer inserts en la tabla de productos en este horario', 1
	END
END
```

### Ejercicio 4
Crear un trigger que ante un borrado sobre la tabla ORDERS realice un borrado en cascada sobre la tabla ITEMS, validando que sólo se borre 1 orden de compra.
Si detecta que están queriendo borrar más de una orden de compra, informará un error y abortará la operación.

```sql
CREATE TRIGGER borradoEnCascada
ON orders
INSTEAD OF DELETE
AS
BEGIN
	IF (SELECT COUNT(*) FROM deleted) > 1
	BEGIN
	    THROW 50000, 'Estas queriendo borrar más de una orden de compra, abortamos operación', 1;
	END

	ELSE 
	BEGIN
		DELETE FROM items WHERE order_num IN (SELECT d.order_num FROM deleted d);
		DELETE FROM orders WHERE order_num IN (SELECT d.order_num FROM deleted d);
	END
END;
```

### Ejercicio 5
Crear un trigger de INSERT sobre la tabla ítems que al detectar que el código de fabricante (manu_code) del producto a comprar no existe en la tabla manufact, inserte una fila en dicha tabla con el manu_code ingresado, en el campo manu_name la descripción ‘Manu Orden 999’ donde 999 corresponde al nro. de la orden de compra a la que pertenece el ítem y en el campo lead_time el valor 1.

```sql
ALTER TRIGGER manuCodeInexistente ON items
AFTER INSERT AS
BEGIN
	DECLARE itemsCursor CURSOR FOR SELECT manu_code, order_num FROM inserted

	DECLARE @manu_code CHAR(3), @order_num INT

	OPEN itemsCursor
	FETCH NEXT FROM itemsCursor INTO @manu_code, @order_num

	WHILE @@FETCH_STATUS = 0
	BEGIN
		IF (SELECT COUNT(*) FROM manufact WHERE manu_code = @manu_code) = 0
		BEGIN
			INSERT INTO manufact (manu_code, manu_name, lead_time) VALUES (@manu_code, 'Manu Orden ' + CAST(@order_num AS VARCHAR(10)), 1)
		END

		FETCH NEXT FROM itemsCursor INTO @manu_code, @order_num
	END

	CLOSE itemsCursor
	DEALLOCATE itemsCursor
END;
```

### Ejercicio 6
Crear tres triggers (Insert, Update y Delete) sobre la tabla Products para replicar todas las operaciones en la tabla products_replica, la misma deberá tener la misma estructura de la tabla Products.

```sql
CREATE TRIGGER replicaInsertProducts ON products
AFTER INSERT AS
BEGIN
	INSERT INTO products_replica SELECT * FROM inserted;	
END

ALTER TRIGGER replicaUpdateProducts ON products
AFTER UPDATE AS
BEGIN
    MERGE products_replica pr
    USING inserted i
    ON pr.stock_num = i.stock_num AND pr.manu_code = i.manu_code 
    WHEN MATCHED THEN
        UPDATE SET pr.unit_price = i.unit_price, pr.unit_code = i.unit_code;
END

ALTER TRIGGER replicaDeleteProducts ON products
AFTER DELETE
AS
BEGIN
    DELETE pr
    FROM products_replica pr INNER JOIN deleted d ON pr.stock_num = d.stock_num AND pr.manu_code = d.manu_code;
END
```

### Ejercicio 7
Crear la vista productos_x_fabricante que tenga los siguientes atributos: Stock_num, description, manu_code, manu_name, unit_price
Crear un trigger de INSERT sobre la vista anterior que ante un insert, inserte una fila en la tabla Products, pero si el manu_code no existe en la tabla manufact, inserte además una fila en dicha tabla con el campo lead_time en 1.

```sql
CREATE VIEW productos_x_fabricante AS
SELECT p.stock_num, pt.description, m.manu_code, m.manu_name, p.unit_price
FROM products p 
	INNER JOIN manufact m ON p.manu_code = m.manu_code 
	INNER JOIN product_types pt ON p.stock_num = pt.stock_num;

CREATE TRIGGER insertEnProductosPorFabricante 
ON productos_x_fabricante
AFTER INSERT AS
BEGIN
    DECLARE @stock_num INT, @manu_code CHAR(3), @unit_price DECIMAL(6, 2);

    INSERT INTO products (stock_num, manu_code, unit_price)
    SELECT stock_num, manu_code, unit_price
    FROM inserted;

    INSERT INTO manufact (manu_code, lead_time)
    SELECT DISTINCT manu_code, 1
    FROM inserted
    WHERE manu_code NOT IN (SELECT manu_code FROM manufact);
END;
```