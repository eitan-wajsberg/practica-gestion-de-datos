```sql
SELECT s.state, s.sname, sub.customer_num, sub.nombre_cliente, sub.monto_total_comprado
FROM state s INNER JOIN (
        SELECT TOP 1 c.state, c.customer_num, c.lname + ', ' + c.fname nombre_cliente, SUM(i.quantity * i.unit_price) monto_total_comprado
        FROM customer c
            INNER JOIN orders o ON o.customer_num = c.customer_num
            INNER JOIN items i ON i.order_num = o.order_num
        WHERE o.order_date > '2015-06-01'
        GROUP BY c.state, c.customer_num, c.lname + ', ' + c.fname
        ORDER BY monto_total_comprado
    ) sub ON s.state = sub.state  
GROUP BY s.state, s.sname, sub.customer_num, sub.nombre_cliente, sub.monto_total_comprado
ORDER BY s.state;
```

```sql 
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
    INSERT INTO listaPrecioHist (stock_num, manu_code, fecha_modif, precio_anterior, precio_nuevo)
    SELECT i.stock_num, i.manu_code, GETDATE(), d.unit_price, i.unit_price
    FROM inserted i INNER JOIN deleted d ON i.stock_num = d.stock_num AND i.manu_code = d.manu_code;
END
```


```sql
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
    DECLARE @stockNum SMALLINT;
    DECLARE @fechaModificacion DATETIME;
    DECLARE @unitPriceAnterior DECIMAL(6, 2);
    DECLARE @unitPriceNuevo DECIMAL(6, 2);

    DECLARE cursorProductos CURSOR FOR
        SELECT stock_num, manu_code, GETDATE(), unit_price 
        FROM products 
        WHERE manu_code = @manuCode

    BEGIN TRANSACTION
        OPEN cursorProductos;
        FETCH cursorProductos INTO @stockNum, @fechaModificacion, @unitPriceAnterior;

        BEGIN TRY
            WHILE @@FETCH_STATUS = 0 
            BEGIN
                IF NOT EXISTS (SELECT 1 FROM products WHERE stock_num = @stockNum)
                    THROW 50000, 'No existe un producto con ese stock_num.', 1;
                    
           
                SET @unitPriceNuevo = @unitPriceAnterior * @porcentajeAumento;
                UPDATE products SET unit_price = @unitPriceNuevo
                WHERE stock_num = @stockNum AND manu_code = @manuCode;

                INSERT INTO historiaPrecios (stock_num, manu_code, fecha_modificacion, porcentaje_aumento, precio_anterior, precio_nuevo)
                VALUES (@stockNum, @manuCode, @fechaModificacion, @porcentajeAumento, @unitPriceAnterior, @unitPriceNuevo);

                FETCH cursorProductos INTO @stockNum, @fechaModificacion, @unitPriceAnterior;
            END
        END TRY

        BEGIN CATCH
            ROLLBACK;
            PRINT 'Mensaje error: ' + CAST(ERROR_MESSAGE() AS VARCHAR(255));
        END CATCH
    COMMIT;
END
```