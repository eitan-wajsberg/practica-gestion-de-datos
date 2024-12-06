# Simulacro - 1

Simulacro realizado el día 06 de Noviembre del 2024 para la decimo tercera clase de Gestión de Datos.

## Parte 1 - Teoria y SQL

### Ejercicio A
Explique en menos de 15 renglones qué es Dominio y las diferentes formas de implementarlo en una BD.
```
El dominio es el conjunto de valores, de los cuales los atributos obtienen sus valores. 
    - Es la menor unidad semantica de informacion.
    - Es atomico, es decir que no se puede descomponer sin perder el significado.
    - Es un conjunto de valores escalares de igual tipo.
    - No contiene valores nulos.
Las diferentes formas que existen para implementar el dominio en una base de datos son:
    - Constraints (PK, FK, DEFAULT, CHECK, NOT NULL, UNIQUE), pero los mas representativos son las claves foraneas y los checks. Restringen los valores posibles.
    - Triggers ya que restringen el Dominio mediante procedimientos que validan reglas
    de negocio.
    - Definicion de tipo de dato para cada columna, porque restringe el tipo de datoque puede tomar la columna.
    - Utilizando una convencion para colocarle nombres a las columnas ya que debe ser definido claramente para que cualquiera entienda a que se refiere ese campo.
```

### Ejercicio B
En una carilla explique Índices: Qué son, para qué sirven, tipos, ventajas, desventajas, y su relación con la funcionalidad de integridad.

```
Los indices son estructuras opcionales asociadas a una tabla. La funcion principal de los indices es permitir un acceso mas rapido a los datos de una tabla. Ademas son independientes de los datos de la tabla asociada (logica y fisicamente). 

Tipos de indices: 
    - Btree Index: Es la estructura de indice estandar y la mas utilizada. Es ideal para búsquedas rápidas y consultas con rangos.
    - Btree Cluster Index: Ordena físicamente las filas de la tabla según los valores del índice.
    - Otros indices: Bitmap Index, Hash index, Functional index y Reverse key index. 

Ventajas de utilizar indices:
    - Mejora la performance ya que al acceder mediante indices no es necesario hacer una lectura secuencial.
    - Mejora el ordenamiento de filas.
    - Mejora la performance de consultas: cuando las columnas que intervienen en un JOIN tienen indices.  

Desventajas de utilizar indices:
    - Requieren espacio adicional en disco.
    - Mantenerlos actualizados genera overhead, especialmente en tablas con alta frecuencia de inserciones, bajas o actualizaciones.

Caracteristicas diferenciadoras para los indices:
    - Unique: clave unica, una sola fila por clave.
    - Duplicada: multiples filas para una misma clave.
    - Simple: clave integrada por una sola columna.
    - Compuesta: clave integrada por mas de una columna. Facilitan multiples JOINS. 

Relacion con la funcionalidad de integridad: Los indices aseguran el cumplimiento de constraints sumamente relevantes para garantizar la integridad de los datos, los primary keys y las foreign keys.
```

### Ejercicio C
Obtener los Tipos de Productos, monto total comprado por cliente y por sus referidos. Mostrar: descripción del Tipo de Producto, Nombre y apellido del cliente, monto total comprado de ese tipo de producto, Nombre y apellido de su cliente referido y el monto total comprado de su referido. Ordenado por Descripción, Apellido y Nombre del cliente (Referente). Nota: Si el Cliente no tiene referidos o sus referidos no compraron el mismo producto, mostrar ́-- ́ como nombre y apellido del referido y 0 (cero) en la cantidad vendida.

```sql
SELECT 
    pt.description, c1.fname cliente_nombre, c1.lname cliente_apellido, 
    COALESCE(SUM(i1.quantity * i1.unit_price), 0) cliente_monto_total,
    COALESCE(s.fname, '-- ') referido_nombre, 
    COALESCE(s.lname, '-- ') referido_apellido, 
    COALESCE(s.monto_total, 0) referido_monto_total
FROM customer c1 
    INNER JOIN orders o1 ON o1.customer_num = c1.customer_num
    INNER JOIN items i1 ON i1.order_num = o1.order_num
    INNER JOIN product_types pt ON pt.stock_num = i1.stock_num
    LEFT JOIN (
        SELECT c2.customer_num, c2.fname, c2.lname, SUM(i2.quantity * i2.unit_price) monto_total, i2.stock_num
        FROM customer c2
            INNER JOIN orders o2 ON o2.customer_num = c2.customer_num
            INNER JOIN items i2 ON i2.order_num = o2.order_num 
        GROUP BY c2.customer_num, c2.fname, c2.lname, i2.stock_num
    ) s ON c1.customer_num_referedBy = s.customer_num AND i1.stock_num = s.stock_num
GROUP BY pt.description, c1.fname, c1.lname, s.fname, s.lname, s.monto_total
ORDER BY pt.description, c1.fname, c1.lname; 
```

## Parte 2 - Stored Procedures y Triggers

### Ejercicio D
Crear un procedimiento actualizaPrecios que reciba como parámetro una fecha a partir de la cual procesar los registros de una tabla Novedades que contiene los nuevos precios de Productos con la siguiente estructura/información: FechaAlta, Manu_code, Stock_num, descTipoProducto, Unit_price. Por cada fila de la tabla Novedades:
1. Si no existe el Fabricante, devolver un error de Fabricante inexistente y descartar la novedad. 
2. Si no existe el stock_num (pero existe el Manu_code) darlo de alta en la tabla Product_types.
3. Si ya existe el Producto actualizar su precio. Si no existe, Insertarlo en la tabla de productos. 
Nota: Manejar una transacción por novedad y errores no contemplados.

```sql
SELECT GETDATE() fechaAlta, p.manu_code manuCode, p.stock_num stockNum, pt.description descTipoProducto, p.unit_price unitPrice 
INTO novedades FROM products p INNER JOIN product_types pt ON pt.stock_num = p.stock_num
WHERE 1 = 2;

CREATE PROCEDURE actualizaPrecios (@fechaInicio DATE) AS
BEGIN
    DECLARE cursorNovedades CURSOR FOR 
        SELECT fechaAlta, manuCode, stockNum, descTipoProducto, unitPrice
        FROM novedades
        WHERE fechaAlta >= @fechaInicio
	
	DECLARE @fechaAlta DATE;
	DECLARE @manuCode CHAR(3);
	DECLARE @stockNum SMALLINT;
    DECLARE @descTipoProducto VARCHAR(15);
    DECLARE @unitPrice DECIMAL(6, 2);

    OPEN cursorNovedades;
    FETCH cursorNovedades INTO @fechaAlta, @manuCode, @stockNum, @descTipoProducto, @unitPrice; 

    WHILE (@@FETCH_STATUS = 0)
	BEGIN
        BEGIN TRANSACTION
		    BEGIN TRY
                IF NOT EXISTS (SELECT 1 FROM manufact WHERE manu_code = @manuCode)
                    THROW 50000, 'Fabricante inexistente, se descarta la novedad.', 1
                
                IF NOT EXISTS (SELECT 1 FROM product_types WHERE stock_num = @stockNum)
                    INSERT INTO product_types (stock_num, description) VALUES (@stockNum, @descTipoProducto)

                IF NOT EXISTS (SELECT 1 FROM products WHERE stock_num = @stockNum AND manu_code = @manuCode)
                    INSERT INTO products (stock_num, manu_code, unit_price) VALUES (@stockNum, @descTipoProducto, @unitPrice)
                ELSE 
                    UPDATE products SET unit_price = @unitPrice
	
				COMMIT
			END TRY

			BEGIN CATCH
				PRINT 'Numero error: ' + CAST(ERROR_NUMBER() AS VARCHAR);
				PRINT 'Mensaje: ' + ERROR_MESSAGE();
				ROLLBACK
			END CATCH

        FETCH cursorNovedades INTO @fechaAlta, @manuCode, @stockNum, @descTipoProducto, @unitPrice;
    END
	
    CLOSE cursorNovedades;
    DEALLOCATE cursorNovedades;
END
```

### Ejercicio E

Se desea llevar en tiempo real la cantidad de llamadas/reclamos (Cust_calls) de los Clientes (Customers) que se producen por cada mes del año y por cada tipo (Call_code). Ante este requerimiento, se solicita realizar un trigger que cada vez que se produzca un Alta o Modificación en la tabla Cust_calls, se actualice una tabla ResumenLLamadas donde se lleve en tiempo real la cantidad de llamadas por Año, Mes y Tipo de llamada. Ejemplo. Si se da de alta una llamada, se debe sumar 1 a la cantidad de ese Año, Mes y Tipo de llamada. En caso de ser una modificación y se modifica el tipo de llamada (por ejemplo por una mala clasificación del operador), se deberá restar 1 al tipo anterior y sumarle 1 al tipo nuevo. Si no se modifica el tipo de llamada no se deberá hacer nada. Tabla de llamadas:
```sql
CREATE TABLE ResumenLLamadas (
    Anio decimal(4) PK,
    Mes decimal(2) PK,
    Call_code char(1) PK,
    Cantidad int
);
```

Nota: No se modifica la PK de la tabla de llamadas. Tener en cuenta altas y modificaciones múltiples.

```sql
CREATE TRIGGER actualizarResumenLlamadas ON cust_calls 
AFTER INSERT, UPDATE AS
BEGIN
    DECLARE cursorCalls CURSOR FOR 
        SELECT YEAR(call_dtime), MONTH(call_dtime), call_code, 1
        FROM inserted i
        UNION ALL
        SELECT YEAR(call_dtime), MONTH(call_dtime), call_code, -1
        FROM deleted 
        
	DECLARE @anio DECIMAL(4);
    DECLARE @mes DECIMAL(2);
	DECLARE @tipo CHAR(1);
    DECLARE @cantidad SMALLINT;

    OPEN cursorCalls;
    FETCH cursorCalls INTO @anio, @mes, @tipo, @cantidad; 

    WHILE (@@FETCH_STATUS = 0)
	BEGIN
        IF NOT EXISTS (SELECT 1 FROM ResumenLlamadas WHERE anio = @anio AND mes = @mes AND code_call = @tipo)
            INSERT INTO ResumenLLamadas VALUES(@anio, @mes, @tipo, 0)

        UPDATE ResumenLlamadas SET cantidad += @cantidad
        WHERE anio = @anio AND mes = @mes AND call_code = @tipo 

        FETCH cursorCalls INTO @anio, @mes, @tipo, @cantidad; 
    END
	
    CLOSE cursorCalls;
    DEALLOCATE cursorCalls; 
END
```