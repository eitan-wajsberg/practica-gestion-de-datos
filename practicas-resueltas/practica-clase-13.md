# Práctica - Clase 13

Práctica realizada el día 06 de Noviembre del 2024 para la decimo tercera clase de Gestión de Datos.

## Parte 1 - Teoria y SQL

### Ejercicio A
Explique en menos de 15 renglones qué es Dominio y las diferentes formas de implementarlo en una BD.
```sql
```

### Ejercicio B
En una carilla explique Índices: Qué son, para qué sirven, tipos, ventajas, desventajas, y su relación con la funcionalidad de integridad.
```sql
```

### Ejercicio C
Obtener los Tipos de Productos, monto total comprado por cliente y por sus referidos. Mostrar: descripción del Tipo de Producto, Nombre y apellido del cliente, monto total comprado de ese tipo de producto, Nombre y apellido de su cliente referido y el monto total comprado de su referido. Ordenado por Descripción, Apellido y Nombre del cliente (Referente). Nota: Si el Cliente no tiene referidos o sus referidos no compraron el mismo producto, mostrar ́-- ́ como nombre y apellido del referido y 0 (cero) en la cantidad vendida.

-- FIXME!!!! Pista: Subquery
```sql
SELECT pt.description Descripcion, c1.lname Apellido, c1.fname Nombre, SUM(i1.quantity * i1.unit_price) 'Total Comprado', COALESCE(c2.lname, '--') 'Apellido Referido', COALESCE(c2.fname, '--') 'Nombre Referido', COALESCE(SUM(i2.quantity * i2.unit_price), 0) 'Total Comprado'
FROM customer c1 
    LEFT JOIN orders o ON o.customer_num = c1.customer_num
    LEFT JOIN items i ON i.order_num = o.order_num
    LEFT JOIN product_types pt ON pt.stock_num = i.stock_num
    LEFT JOIN customer c2 ON c2.customer_num = c1.customer_num_referredBy
GROUP BY pt.description, c1.lname, c1.fname, c2.lname, c2.fname
ORDER BY pt.description, c2.fname
```

## Parte 2 - Stored Procedures y Triggers

### Ejercicio D
Crear un procedimiento actualizaPrecios que reciba como parámetro una fecha a partir de la cual procesar los registros de una tabla Novedades que contiene los nuevos precios de Productos con la siguiente estructura/información: FechaAlta, Manu_code, Stock_num, descTipoProducto, Unit_price. Por cada fila de la tabla Novedades:
1. Si no existe el Fabricante, devolver un error de Fabricante inexistente y descartar la novedad. 
2. Si no existe el stock_num (pero existe el Manu_code) darlo de alta en la tabla Product_types.
3. Si ya existe el Producto actualizar su precio. Si no existe, Insertarlo en la tabla de productos. 
Nota: Manejar una transacción por novedad y errores no contemplados.

```sql
CREATE PROCEDURE actualizaPrecios @fechaComienzo DATE
AS
BEGIN
    DECLARE cursorNovedades CURSOR FOR
        SELECT o.order_date fecha_alta, i.manu_code manu_code, i.stock_num stock_num, pt.description desc_tipo_producto, i.unit_price unit_price 
        FROM items i 
        INNER JOIN product_types pt ON p.stock_num = i.stock_num
        INNER JOIN order o ON o.order_num = i.order_num 
        WHERE n.FechaAlta >= @fechaComienzo

    DECLARE @fecha_alta DATE, 
            @manu_code CHAR(3), 
            @stock_num INT, 
            @description VARCHAR(15), 
            @unit_price DECIMAL(6, 2)

    OPEN cursorNovedades
    FETCH NEXT FROM cursorNovedades INTO @fecha_alta, @manu_code, @stock_num, @description, @unit_price

    WHILE @@FETCH_STATUS = 0
    BEGIN
        BEGIN TRY
            BEGIN TRANSACTION

            IF NOT EXISTS (SELECT 1 FROM manufact WHERE manu_code = @manu_code)
            BEGIN
                THROW 50000, 'Fabricante inexistente, descartamos la novedad.', 1
            END
            
            IF NOT EXISTS (SELECT 1 FROM products WHERE stock_num = @stock_num AND manu_code = @manu_code)
            BEGIN
                IF NOT EXISTS (SELECT 1 FROM product_types WHERE stock_num = @stock_num)
                BEGIN
                    INSERT INTO product_types (stock_num, description)
                    VALUES (@stock_num, @description)
                END

                INSERT INTO products (stock_num, manu_code, unit_price)
                VALUES (@stock_num, @manu_code, @unit_price)
            END
            ELSE
            BEGIN
                UPDATE products 
                SET unit_price = @unit_price
                WHERE stock_num = @stock_num AND manu_code = @manu_code
            END

            COMMIT TRANSACTION
        END TRY

        BEGIN CATCH
            ROLLBACK TRANSACTION
            DECLARE @errorDescripcion VARCHAR(100) = 
                'Error en la novedad del Fabricante: ' + @manu_code + ', Stock_num: ' + CAST(@stock_num AS VARCHAR)
            RAISERROR(@errorDescripcion, 16, 1)
        END CATCH

        FETCH NEXT FROM cursorNovedades INTO @fecha_alta, @manu_code, @stock_num, @description, @unit_price
    END

    CLOSE cursorNovedades
    DEALLOCATE cursorNovedades
END
```

### Ejercicio E

Se desea llevar en tiempo real la cantidad de llamadas/reclamos (Cust_calls) de los Clientes (Customers) que se producen por cada mes del año y por cada tipo (Call_code). Ante este requerimiento, se solicita realizar un trigger que cada vez que se produzca un Alta o Modificación en la tabla Cust_calls, se actualice una tabla ResumenLLamadas donde se lleve en tiempo real la cantidad de llamadas por Año, Mes y Tipo de llamada. Ejemplo. Si se da de alta una llamada, se debe sumar 1 a la cantidad de ese Año, Mes y Tipo de llamada. En caso de ser una modificación y se modifica el tipo de llamada (por ejemplo por una mala clasificación del operador), se deberá restar 1 al tipo anterior y sumarle 1 al tipo nuevo. Si no se modifica el tipo de llamada no se deberá hacer nada. Tabla de llamadas:
`
CREATE TABLE ResumenLLamadas (
    Anio decimal(4) PK,
    Mes decimal(2) PK,
    Call_code char(1) PK,
    Cantidad int
);
`

Nota: No se modifica la PK de la tabla de llamadas. Tener en cuenta altas y modificaciones múltiples.

```sql
```