# Práctica - Clase 9

Práctica realizada el día 16 de Octubre del 2024 para la novena clase de Gestión de Datos.

## Práctica de Triggers II

### Ejercicio 1
Crear un trigger que valide que ante un `INSERT` de una o más filas en la tabla ítems, realice la siguiente validación:
- Si la orden de compra a la que pertenecen los ítems ingresados corresponde a clientes del estado de California, se deberá validar que estas órdenes puedan tener como máximo 5 registros en la tabla ítem.
- Si se insertan más ítems de los definidos, el resto de los ítems se deberán insertar en la tabla items_error la cual contiene la misma estructura que la tabla ítems más un atributo fecha que deberá contener la fecha del día en que se trató de insertar.

Por ejemplo, si la Orden de Compra tiene 3 items y se realiza un insert masivo de 3 ítems más, el trigger deberá insertar los 2 primeros en la tabla ítems y el restante en la tabla ítems_error. Supuesto: En el caso de un insert masivo los items son de la misma orden.

```sql
SELECT * INTO items_error FROM items WHERE 1=2;
ALTER TABLE items_error ADD fechaInsercion DATETIME DEFAULT GETDATE();

ALTER TRIGGER validacionItems ON items
INSTEAD OF INSERT AS
BEGIN
	DECLARE itemsValidacionCursor CURSOR FOR 
		SELECT o.customer_num, i.order_num, i.item_num
		FROM inserted i INNER JOIN orders o ON (o.order_num = i.order_num)

	DECLARE @customer_num SMALLINT, @order_num SMALLINT, @item_num SMALLINT 

	OPEN itemsValidacionCursor
	FETCH NEXT FROM itemsValidacionCursor INTO @customer_num, @order_num, @item_num

	WHILE @@FETCH_STATUS = 0
	BEGIN
		IF (SELECT state FROM customer WHERE customer_num = @customer_num) = 'CA' AND (SELECT COUNT(*) FROM items WHERE order_num = @order_num) > 5 
		BEGIN
			INSERT INTO items_error (item_num, order_num, stock_num, manu_code, quantity, unit_price) 
				SELECT * FROM inserted WHERE item_num = @item_num AND order_num = @order_num;
		END

		ELSE 
		BEGIN
			INSERT INTO items SELECT * FROM inserted WHERE item_num = @item_num AND order_num = @order_num;
		END

		FETCH NEXT FROM itemsValidacionCursor INTO @customer_num, @order_num, @item_num
	END

	CLOSE itemsValidacionCursor;
	DEALLOCATE itemsValidacionCursor;
END;

SELECT * FROM items WHERE order_num=1001;
SELECT * FROM items_error;
SELECT * FROM orders;
SELECT * FROM customer;

INSERT INTO items VALUES (6, 1001, 1, 'HRO', 1, 250.00);
```

### Ejercicio 2
```sql
CREATE VIEW ProdPorFabricante AS
	SELECT m.manu_code, m.manu_name, COUNT(*) cantidad_productos
	FROM manufact m INNER JOIN products p ON (m.manu_code = p.manu_code)
	GROUP BY m.manu_code, manu_name;
```
Crear un trigger que permita ante un insert en la vista ProdPorFabricante insertar una fila
en la tabla manufact. Observaciones: el atributo leadtime deberá insertarse con un valor default 10. El trigger deberá contemplar inserts de varias filas, por ejemplo ante un `INSERT` / `SELECT`.

```sql
CREATE TRIGGER insertarEnManufact ON ProdPorFabricante
INSTEAD OF INSERT AS
BEGIN
	INSERT INTO manufact (manu_code, manu_name, lead_time) SELECT manu_code, manu_name, 10 FROM inserted;
END
```

### Ejercicio 3
Crear un trigger que ante un `INSERT` o `UPDATE` de una o más filas de la tabla Customer, realice la siguiente validación.
- La cuota de clientes correspondientes al estado de California es de 20, si se supera dicha cuota se deberán grabar el resto de los clientes en la tabla customer_pend.
- Validar que si de los clientes a modificar se modifica el Estado, no se puede superar dicha cuota.

Si por ejemplo el estado de CA cuenta con 18 clientes y se realiza un update o insert masivo de 5 clientes con estado de CA, el trigger deberá modificar los 2 primeros en la tabla customer y los restantes grabarlos en la tabla customer_pend. La tabla customer_pend tendrá la misma estructura que la tabla customer con un atributo adicional fechaHora que deberá actualizarse con la fecha y hora del día.

```sql
SELECT * INTO customer_pend FROM customer WHERE 1=2;
ALTER TABLE customer_pend ADD fecha DATETIME DEFAULT GETDATE();

CREATE TRIGGER validacionCuotasCustomer ON customer
INSTEAD OF INSERT, UPDATE AS
BEGIN
	DECLARE customerValidacionCursor CURSOR FOR SELECT customer_num, state FROM inserted i 
	DECLARE @customer_num SMALLINT, @state CHAR(2)

	OPEN customerValidacionCursor
	FETCH NEXT FROM customerValidacionCursor INTO @customer_num, @state

	WHILE @@FETCH_STATUS = 0
	BEGIN
		DECLARE @cantidad_clientes_estado INT;
		SET @cantidad_clientes_estado = (SELECT COUNT(*) FROM customer WHERE state = @state);

		IF @cantidad_clientes_estado > 20
		BEGIN
			INSERT INTO customer_pend (customer_num, fname, lname, company, address1, address2, city, state, zipcode, phone, customer_num_referedBy, status) 
				SELECT * FROM inserted WHERE customer_num = @customer_num;
		END

		ELSE 
		BEGIN
			INSERT INTO customer SELECT * FROM inserted WHERE customer_num = @customer_num;
		END

		FETCH NEXT FROM customerValidacionCursor INTO @customer_num, @state
	END

	CLOSE customerValidacionCursor;
	DEALLOCATE customerValidacionCursor;
END
```

### Ejercicio 4
```sql
CREATE VIEW ProdPorFabricanteDet AS
	SELECT m.manu_code, m.manu_name, pt.stock_num, pt.description
	FROM manufact m 
		LEFT OUTER JOIN products p ON m.manu_code = p.manu_code
		LEFT OUTER JOIN product_types pt ON p.stock_num = pt.stock_num;
```
Crear un trigger que permita ante un `DELETE` en la vista ProdPorFabricante borrar los datos en la tabla manufact pero sólo de los fabricantes cuyo campo description sea NULO (o sea que no tienen stock).
Observaciones: El trigger deberá contemplar borrado de varias filas mediante un `DELETE` masivo. En ese caso sólo borrará de la tabla los fabricantes que no tengan productos en stock, borrando los demás.

```sql
-- Opcion 1
CREATE TRIGGER borrarAquellosSinStock ON ProdPorFabricanteDet
INSTEAD OF DELETE AS
BEGIN
	DELETE FROM manufact WHERE manu_code IN (SELECT manu_code FROM deleted d WHERE description IS NULL)
END

-- Opcion 2
ALTER TRIGGER borrarAquellosSinStock ON ProdPorFabricanteDet
INSTEAD OF DELETE AS
BEGIN
    DELETE FROM manufact 
    WHERE manu_code IN (
        SELECT d.manu_code 
        FROM deleted d
        JOIN products p ON d.manu_code = p.manu_code
        JOIN product_types pt ON p.stock_num = pt.stock_num
        GROUP BY d.manu_code 
        HAVING COUNT(pt.stock_num) = 0
    );
END;
```

### Ejercicio 5
Crear un trigger que permita ante un delete de una sola fila en la vista ordenesPendientes valide:
- Si el cliente asociado a la orden tiene sólo esa orden pendiente de pago (paid_date IS NULL), no permita realizar la Baja, informando el error.
- Si la Orden tiene más de un ítem asociado, no permitir realizar la Baja, informando el error.
- Ante cualquier otra condición borrar la Orden con sus ítems asociados, respetando la integridad referencial.

Estructura de la vista: customer_num, fname, lname, Company, order_num, order_date WHERE paid_date IS NULL.

```sql
CREATE VIEW ordenesPendientes AS
	SELECT c.customer_num, c.fname, c.lname, c.company, o.order_num, o.order_date 
	FROM orders o INNER JOIN customer c ON (c.customer_num = o.customer_num)
	WHERE o.paid_date IS NULL;

ALTER TRIGGER borradoOrdenesPendientes ON ordenesPendientes
INSTEAD OF DELETE AS
BEGIN
    DECLARE @customer_num SMALLINT, @order_num INT;

    SELECT @customer_num = customer_num, @order_num = order_num FROM deleted;

    IF (SELECT COUNT(*) FROM orders WHERE customer_num = @customer_num AND paid_date IS NULL) = 1
    BEGIN
        THROW 50000, 'No puede realizar la baja, ya que tiene solo esa orden pendiente de pago', 1;
    END

    ELSE IF (SELECT COUNT(*) FROM items WHERE order_num = @order_num) > 1
    BEGIN
        THROW 50000, 'No puede realizar la baja, ya que la orden tiene más de un ítem asociado', 1;
    END

    ELSE
    BEGIN
        DELETE FROM items WHERE order_num = @order_num;
        DELETE FROM orders WHERE order_num = @order_num;
    END
END;
```