```sql
-- PUNTO 3 - QUERY

SELECT s.state, s.sname, c1.customer_num, c1.lname + ', ' + c1.fname nombre_cliente, SUM(i1.quantity * i1.unit_price) monto_total_comprado
FROM customer c1 
    INNER JOIN state s ON s.state = c1.state
    INNER JOIN orders o1 ON o1.customer_num = c1.customer_num
    INNER JOIN items i1 ON i1.order_num = o1.order_num 
WHERE c1.customer_num IN (
    SELECT TOP 1 c2.customer_num
    FROM customer c2
        INNER JOIN orders o2 ON o2.customer_num = c2.customer_num
        INNER JOIN items i2 ON i2.order_num = o2.order_num 
    WHERE o2.order_date > '2015-06-01' AND s.state = c2.state 
    GROUP BY c2.customer_num
    ORDER BY SUM(i2.quantity * i2.unit_price) DESC
) AND o1.order_date > '2015-06-01'
GROUP BY s.state, s.sname, c1.customer_num, c1.lname + ', ' + c1.fname
ORDER BY s.state ASC;
```

```sql 
-- PUNTO 4 - TRIGGER

CREATE TABLE listaPreciosHist (
    id SMALLINT PRIMARY KEY IDENTITY(1, 1),
    stock_num SMALLINT NOT NULL,
    manu_code CHAR(3) NOT NULL,
    fecha_modif DATETIME NOT NULL,
    precio_anterior DECIMAL(6, 2),
    precio_nuevo DECIMAL(6, 2) NOT NULL,
    FOREIGN KEY (stock_num, manu_code) REFERENCES products (stock_num, manu_code)
);

CREATE TRIGGER registrarCambiosPrecio ON products
AFTER INSERT, UPDATE AS
BEGIN
    INSERT INTO listaPreciosHist (stock_num, manu_code, fecha_modif, precio_anterior, precio_nuevo)
    SELECT i.stock_num, i.manu_code, GETDATE(), d.unit_price, i.unit_price
    FROM inserted i INNER JOIN deleted d ON i.stock_num = d.stock_num AND i.manu_code = d.manu_code;
END

/* 
    Como los triggers ya manejan transacciones intrinsectamente no utilizo transacciones. Tampoco considere usar
    un cursor, ya que fue posible cargar masivamente el cambio de precio mediante un INSERT-SELECT.
*/
```


```sql
-- PUNTO 5 - PROCEDURE

CREATE TABLE historiaPrecios (
    id SMALLINT PRIMARY KEY IDENTITY(1, 1),
    stock_num SMALLINT NOT NULL,
    manu_code CHAR(3) NOT NULL,
    fecha_modificacion DATETIME NOT NULL,
    porcentaje_aumento DECIMAL(6, 2) NOT NULL,
    precio_anterior DECIMAL(6, 2),
    precio_nuevo DECIMAL(6, 2) NOT NULL,
    FOREIGN KEY (stock_num, manu_code) REFERENCES products (stock_num, manu_code)
);

CREATE PROCEDURE aumentaPreciosPR (@manuCode CHAR(3), @porcentajeAumento DECIMAL(3, 2)) AS
BEGIN
    IF @porcentajeAumento <= 0
        THROW 50000, 'No se puede utilizar un porcentaje de aumento negativo o igual a cero.', 1

    DECLARE @stockNum SMALLINT;
    DECLARE @fechaModificacion DATETIME = GETDATE();
    DECLARE @unitPriceAnterior DECIMAL(6, 2);
    DECLARE @unitPriceNuevo DECIMAL(6, 2);

    DECLARE cursorProductos CURSOR FOR
        SELECT stock_num, unit_price 
        FROM products 
        WHERE manu_code = @manuCode

    BEGIN TRANSACTION
        OPEN cursorProductos;
        FETCH cursorProductos INTO @stockNum, @unitPriceAnterior;

        BEGIN TRY
            WHILE @@FETCH_STATUS = 0 
            BEGIN
                SET @unitPriceNuevo = @unitPriceAnterior * @porcentajeAumento;
                UPDATE products SET unit_price = @unitPriceNuevo
                WHERE stock_num = @stockNum AND manu_code = @manuCode;

                INSERT INTO historiaPrecios (stock_num, manu_code, fecha_modificacion, porcentaje_aumento, precio_anterior, precio_nuevo)
                VALUES (@stockNum, @manuCode, @fechaModificacion, @porcentajeAumento, @unitPriceAnterior, @unitPriceNuevo);

                FETCH cursorProductos INTO @stockNum, @unitPriceAnterior;
            END
            COMMIT;
        END TRY
        BEGIN CATCH
            ROLLBACK;
            PRINT 'Numero error: ' + CAST(ERROR_NUMBER() AS VARCHAR);
			PRINT 'Mensaje: ' + ERROR_MESSAGE();
        END CATCH

	CLOSE cursorProductos;
	DEALLOCATE cursorProductos;
END
```