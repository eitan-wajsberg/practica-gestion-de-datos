# Simulacro - 2 (Parcial 12/07/2023)

Simulacro realizado el día 13 de Noviembre del 2024 para la decimo cuarta clase de Gestión de Datos.

## Parte 1 - Teoria y SQL

### Ejercicio A
Explicar el concepto de clave primaria y foranea, relacionando ambos conceptos con la integridad.

```
Una clave primaria es un atributo (o un conjunto de atributos) que identifican univocamente un registro / fila en una tabla.
Una clave foranea es un atributo (o un conjunto de atributos) hacen referencia a una clave primaria de otra tabla o de si misma (si se trata de una relacion recursiva).
Ambas claves estan sumamente relacionadas con el concepto de integridad:
    - La clave primaria asegura la integridad de entidad, es decir que los datos pertenecientes a una tabla tienen una unica forma de identificarse, ya que cada fila de la tabla tiene un identificador unico.
    - La clave foranea asegura la integridad referencial, es decir que garantiza la coherencia entre los datos de las dos tablas. Esta clave mantienen un estado consistente y nos evita llevar a cabo operaciones que violarian la integridad referencial, por ejemplo eliminar un producto de la tabla de productos que es referenciado en varias ventas, eliminarlo no sera posible (a menos que se indice lo contrario con un DELETE CASCADE MODE o algo asi).
```

### Ejercicio B
Explicar independencia lógica y física. Detallar las reglas de Codd que aseguran que un motor de bases de datos es relacional.

```
Independencia logica: es la capacidad de modificar el esquema logico (tablas, vistas, relaciones) sin afectar las aplicaciones que las consultan. Por ejemplo, cambiar un campo de una tabla que no afecte a las operaciones que realiza una aplicacion.
Independencia fisica: es la capacidad de modificar el esquema fisico (como se almacenan los datos en el disco) sin modificar el esquema logico. Por ejemplo, agregar un indice en una tabla.

Reglas de Codd que aseguran que un motor de bases de datos es relacional:
    - Independencia entre el motor y los programas que acceden a los datos.
    - Representar la informacion de la estructura de las tablas y sus relaciones mediante un catalogo dinamico basado en el modelo relacional.
    - Las reglas de integridad deben guardarse en el catalogto de la base, no en programas de aplicacion.
    - Soportar informacion faltante mediante valores nulos (NULL).
    - Proveer lenguajes para:
        - La definicion de datos.
        - Manipulacion de datos (insertar, eliminar, modificar, buscar, actualizar).
        - Definicion de restricciones de seguridad, integridad, autorizacion y transacciones. 
```

## Ejercicio C
Por cada estado (state) seleccionar los dos clientes que mayores montos compraron. Se deberá mostrar el código del estado, nro de cliente, nombre y apellido del cliente y monto total comprado. Mostrar la información ordenada por provincia y por monto comprado en forma descendente. Notas: No se puede usar Store procedures, ni funciones de usuarios, ni tablas temporales.

```sql
SELECT c1.state, c1.customer_num, c1.lname + ', ' + c1.fname nombre_completo, SUM(i.quantity * i.unit_price) monto_total
FROM customer c1
	INNER JOIN orders o ON o.customer_num = c1.customer_num
	INNER JOIN items i ON i.order_num = o.order_num
WHERE c1.customer_num IN (
	SELECT TOP 2 c2.customer_num
	FROM customer c2
		INNER JOIN orders o2 ON o2.customer_num = c2.customer_num
		INNER JOIN items i2 ON i2.order_num = o2.order_num
	WHERE c2.state = c1.state
	GROUP BY c2.customer_num
	ORDER BY SUM(i2.quantity * i2.unit_price) DESC
)
GROUP BY c1.state, c1.customer_num, c1.lname + ', ' + c1.fname
ORDER BY c1.state, monto_total DESC;

/*
    Atento:
      - Habias hecho la consulta menos performante ya que metiste un LEFT JOIN con una subconsulta que no era necesaria.
      - Te habias olvidado de aclarar en el ORDER BY mas importante (el que esta dentro de la subquery) que era DESC. 
*/ 
```

## Parte 2 - Stored Procedures y Triggers

### Ejercicio D
Crear un procedimiento BorrarProd que en base a una tabla ProductosDeprecados que  contiene filas con Productos a borrar realice la eliminación de los mismos de la tabla Products. El procedimiento deberá guardar en una tabla de auditoria AuditProd (stock_num, manu_code, Mensaje) el producto y un mensaje que podrá ser: ‘Borrado’, ‘Producto con ventas’ o cualquier mensaje de error que se produjera. Crear las tablas ProductosDeprecados y AuditProd. Deberá manejar una transacción por registro. Ante un error deshacer lo realizado y seguir  procesando los demás registros. Asimismo, deberá manejar excepciones ante cualquier error que ocurra.

```sql
SELECT * INTO productosDeprecados FROM products WHERE 1=2;

CREATE TABLE AuditProd (
    stock_num SMALLINT NOT NULL,
    manu_code CHAR(3) NOT NULL,
    mensaje VARCHAR(100) NULL,
    PRIMARY KEY (stock_num, manu_code),
    FOREIGN KEY (stock_num, manu_code) REFERENCES products(stock_num, manu_code)
);

CREATE PROCEDURE borrarProd AS
BEGIN
    DECLARE cursorDeprecados CURSOR FOR 
    SELECT stock_num, manu_code FROM productosDeprecados;

	DECLARE @stockNum SMALLINT, @manuCode CHAR(3);

    OPEN cursorDeprecados
    FETCH cursorDeprecados INTO @stockNum, @manuCode; 
    
    WHILE (@@FETCH_STATUS = 0)
	BEGIN
        BEGIN TRANSACTION
		    BEGIN TRY
                IF NOT EXISTS (SELECT 1 FROM products WHERE stock_num = @stockNum AND manu_code = @manuCode)
                    RAISERROR('No existe el producto.', 16, 1)

                IF EXISTS (SELECT 1 FROM items WHERE stock_num = @stockNum AND manu_code = @manuCode)
                    RAISERROR('Producto con ventas', 16, 1)
                
                DELETE FROM products WHERE stock_num = @stockNum AND manu_code = @manuCode;
                INSERT INTO AuditProd VALUES (@stockNum, @manuCode, 'Borrado')
        
				COMMIT
			END TRY

			BEGIN CATCH
                ROLLBACK
				PRINT 'Numero error: ' + CAST(ERROR_NUMBER() AS VARCHAR);
				PRINT 'Mensaje: ' + ERROR_MESSAGE();
                INSERT INTO AuditProd VALUES (@stockNum, @manuCode, ERROR_MESSAGE())
			END CATCH

        FETCH cursorDeprecados INTO @stockNum, @manuCode;
    END
	
    CLOSE cursorDeprecados;
    DEALLOCATE cursorDeprecados;
END

/*
    Atento:
      - Te olvidaste de tratar como errores al "No existe el producto" y al "Producto con ventas" y por ende no se le iba a mostrar el mismo.
      - Acordate que el RAISERROR y el THROW funcionan igual en el TRY, pero en el CATCH no. En el CATCH:
        - El RAISERROR permite personalizar mensajes, pero no detiene la ejecución automáticamente.
        - El THROW lanza el error y detiene la ejecución de inmediato.
*/ 
```

### Ejercicio E
Crear un trigger que ante un cambio de precios en un producto inserte un nuevo registro con el precio anterior (no el nuevo) en la tabla PRECIOS_HIST. La estructura de la tabla PRECIOS_HIST es (stock_num, manu_code, fechaDesde, fechaHasta, precio_unit). La fecha desde del nuevo registro será la fecha hasta del último cambio de precios de ese producto y su fecha hasta será la fecha del dia. SI no tuviese un registro de precio anterior ingrese como fecha desde ‘2000-01-01’. Nota: Las actualizaciones de precios pueden ser masivas.

```sql
CREATE TRIGGER actualizarHistorialPrecios ON products
AFTER UPDATE AS
BEGIN
    DECLARE cursorProductos CURSOR FOR 
    SELECT stock_num, manu_code, unit_price FROM deleted;

	DECLARE @stockNum SMALLINT, @manuCode CHAR(3), @unitPrice DECIMAL(6, 2);
    DECLARE @fechaDesde DATE;
    DECLARE @fechaHasta DATE = GETDATE();

    OPEN cursorProductos
    FETCH cursorProductos INTO @stockNum, @manuCode, @unitPrice; 
    
    WHILE (@@FETCH_STATUS = 0)
	BEGIN
        INSERT INTO precios_hist(stock_num, manu_code, fecha_desde, fecha_hasta, precio_unit)
        VALUES (@stockNum, @manuCode, COALESCE((
                SELECT MAX(fecha_hasta) 
                FROM precios_hist 
                WHERE stock_num = @stockNum AND manu_code = @manuCode
            ), '2000-01-01'), 
            GETDATE(), @unitPrice
        )

        FETCH cursorProductos INTO @stockNum, @manuCode, @unitPrice; 
    END
	
    CLOSE cursorProductos;
    DEALLOCATE cursorProductos;
END

-- Otra opcion valida
CREATE TRIGGER actualizarHistorialPrecios ON products
AFTER UPDATE AS
BEGIN
    INSERT INTO precios_hist(stock_num, manu_code, fecha_desde, fecha_hasta, precio_unit)
    SELECT d.stock_num, d.manu_code, COALESCE((
            SELECT MAX(fecha_hasta) 
            FROM precios_hist h 
            WHERE h.stock_num = d.stock_num AND h.manu_code = d.manu_code
        ), '2000-01-01'), 
        GETDATE(), unit_price
    FROM deleted d
END

/*
    Atento!!!!!
      - Habias puesto AFTER INSERT en lugar de AFTER UPDATE. Atento a estas cosas que son basicas y suman muchos puntos. Un 70% del ejercicio es definir bien eso.
      - Habias metido una transaccion dentro del TRIGGER, eso esta muy mal, los triggers ya manejan transacciones intrinsecamente.
      - Habias sacado los datos de INSERTED, pero los datos eran pertenecientes a DELETED, ya que es el anterior el que se guarda en el historial.

    Lo que habias puesto antes dentro del while tambien era posible:
    
        IF EXISTS (SELECT 1 FROM precios_hist WHERE stock_num = @stockNum AND manu_code = @manuCode)
                SET @fechaDesde = '2000-01-01'
            ELSE 
                SET @fechaDesde = (
                        SELECT MAX(fecha_hasta) 
                        FROM precios_hist 
                        WHERE stock_num = @stockNum AND manu_code = @manuCode
                )
        
        INSERT INTO precios_hist VALUES (@stockNum, @manuCode, @fechaDesde, @fechaHasta, @unitPrice)`
*/ 
```
