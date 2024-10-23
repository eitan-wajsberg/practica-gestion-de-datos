# Práctica - Clase 7

Práctica realizada el día 25 de Septiembre del 2024 para la septima clase de Gestión de Datos.

## Stored Procedures

### Ejercicio 1
- Crear la siguiente tabla CustomerStatistics con los siguientes campos customer_num (entero y pk), ordersqty (entero), maxdate (date), uniqueProducts
(entero)
- Crear un procedimiento ‘actualizaEstadisticas’ que reciba dos parámetros customer_numDES y customer_numHAS y que en base a los datos de la tabla customer cuyo customer_num estén en en rango pasado por parámetro, inserte (si no existe) o modifique el registro de la tabla CustomerStatistics con la siguiente
información:
    - Ordersqty contedrá la cantidad de órdenes para cada cliente. 
    - Maxdate contedrá la fecha máxima de la última órde puesta por cada cliente.
    - uniqueProducts contendrá la cantidad única de tipos de productos adquiridos
    por cada cliente.

```sql
CREATE PROCEDURE actualizarEstadisticas(@customer_numDES INT, @customer_numHAS INT)
AS
BEGIN
	MERGE CustomerStatistics cs
	USING (
		SELECT c.customer_num, COUNT(o.order_num) cantidad_ordenes, MAX(o.order_date) max_date, COUNT(DISTINCT i.stock_num) productos_unicos
		FROM customer c
			INNER JOIN orders o ON(o.customer_num = c.customer_num)
			INNER JOIN items i ON(i.order_num = o.order_num)
		WHERE c.customer_num BETWEEN 0 AND 200
		GROUP BY c.customer_num
	) c
	ON c.customer_num = cs.customer_num
	
	WHEN MATCHED THEN
		UPDATE SET ordersqty=cantidad_ordenes, maxdate=max_date, productsQty=productos_unicos
	
	WHEN NOT MATCHED THEN
		INSERT (customer_num, ordersqty, maxdate, productsQty) VALUES (customer_num, cantidad_ordenes, max_date, productos_unicos);
END;

EXECUTE actualizarEstadisticas 0, 200
SELECT * FROM CustomerStatistics;
```

### Ejercicio 2
Crear un procedimiento ‘migraClientes’ que reciba dos parámetros customer_numDES y customer_numHAS y que dependiendo el tipo de cliente y la cantidad de órdenes los inserte en las tablas clientesCalifornia, clientesNoCaBaja, clienteNoCAAlta.
- El procedimiento deberá migrar de la tabla customer todos los clientes de California a la tabla clientesCalifornia, los clientes que no son de California pero tienen más de 999u$ en OC en clientesNoCaAlta y los clientes que tiene menos de 1000u$ en OC en la tablas clientesNoCaBaja.
- Se deberá actualizar un campo status en la tabla customer con valor ‘P’ Procesado, para todos aquellos clientes migrados.
- El procedimiento deberá contemplar toda la migración como un lote, en el caso que ocurra un error, se deberá informar el error ocurrido y abortar y deshacer la operación.

```sql
SELECT * INTO #clientesCalifornia FROM customer WHERE 1=2; 
SELECT * INTO #clientesNoCaBaja FROM customer WHERE 1=2; 
SELECT * INTO #clientesNoCaAlta FROM customer WHERE 1=2; 

CREATE PROCEDURE migracionClientes(@customer_numDES INT, @customer_numHAS INT)
AS
BEGIN
	DECLARE cursor_clientes CURSOR FOR 
		SELECT c.customer_num, c.state, SUM(i.unit_price * i.quantity) total_ordenes
		FROM customer c
			INNER JOIN orders o ON(o.customer_num = c.customer_num)
			INNER JOIN items i ON(i.order_num = o.order_num)
		WHERE c.customer_num BETWEEN @customer_numDES AND @customer_numHAS
		GROUP BY c.customer_num, c.state;
	
	DECLARE @customer_num INT;
	DECLARE @customer_state CHAR(2);
	DECLARE @customer_total_ordenes INTEGER;
	
	BEGIN TRANSACTION
		BEGIN TRY
			OPEN cursor_clientes;
			FETCH cursor_clientes INTO @customer_num, @customer_state, @customer_total_ordenes;
			WHILE (@@FETCH_STATUS = 0)
				BEGIN
					IF @customer_state = 'CA'
						BEGIN
							INSERT INTO #clientesCalifornia SELECT * FROM customer WHERE customer_num=@customer_num;
						END;
						
					ELSE IF @customer_total_ordenes < 1000
						BEGIN
							INSERT INTO #clientesNoCaBaja SELECT * FROM customer WHERE customer_num=@customer_num
						END;
						
					ELSE IF @customer_total_ordenes > 999
						BEGIN
							INSERT INTO #clientesNoCaAlta SELECT * FROM customer WHERE customer_num=@customer_num
						END;
						
					UPDATE customer SET status='P' WHERE customer_num = @customer_num;
					FETCH cursor_clientes INTO @customer_num, @customer_state, @customer_total_ordenes;
				END;
				COMMIT;
			END TRY;

			BEGIN CATCH
				PRINT 'Numero error: ' + CAST(ERROR_NUMBER() AS VARCHAR);
				PRINT 'Mensaje' + ERROR_MESSAGE();
				ROLLBACK;
			END CATCH;
	
		CLOSE cursor_clientes;
		DEALLOCATE cursor_clientes;
END;
```

### Ejercicio 3
Crear un procedimiento ‘actualizaPrecios’ que reciba como parámetros manu_codeDES, manu_codeHAS y porcActualizacion que dependiendo del tipo de cliente y la cantidad de órdenes genere las siguientes tablas listaPrecioMayor y listaPreciosMenor. Ambas tienen las misma estructura que la tabla Productos.
- El procedimiento deberá tomar de la tabla stock todos los productos que correspondan al rango de fabricantes asignados por parámetro. Por cada producto del fabricante se evaluará la cantidad (quantity) comprada. Si la misma es mayor o igual a 500 se grabará el producto en la tabla listaPrecioMayor y el unit_price deberá ser actualizado con (unit_price *(porcActualización *0,80)), Si la cantidad comprada del producto es menor a 500 se actualizará (o insertará) en la tabla listaPrecioMenor y el unit_price se actualizará con (unit_price * porcActualizacion)
- Asimismo, se deberá actualizar un campo status de la tabla stock con valor ‘A’ Actualizado, para todos aquellos productos con cambio de precio actualizado.
- El procedimiento deberá contemplar todas las operaciones de cada fabricante como un lote, en el caso que ocurra un error, se deberá informar el error ocurrido y deshacer la operación de ese fabricante.

```sql
ALTER PROCEDURE actualizarPrecios @manu_codeDES CHAR(3), @manu_codeHAS CHAR(3), @porcActualizacion DECIMAL (5,3)
AS
BEGIN
	DECLARE @stock_num INT, @manu_code CHAR(3), @unit_price DECIMAL(6,2), @unit_code SMALLINT, @manu_codeAux CHAR(3)

	DECLARE StockCursor CURSOR FOR
		SELECT p.stock_num, manu_code, unit_price, unit_code
		FROM products p
		WHERE manu_code BETWEEN @manu_codeDES AND @manu_codeHAS
		ORDER BY manu_code, p.stock_num

	OPEN StockCursor;
	FETCH NEXT FROM StockCursor INTO @stock_num, @manu_code, @unit_price, @unit_code
	
	SET @manu_codeAux = @manu_code
	WHILE @@FETCH_STATUS = 0
		BEGIN
			BEGIN TRY
				BEGIN TRANSACTION
				IF (SELECT SUM(quantity) FROM items WHERE manu_code = @manu_code AND stock_num=@stock_num) >= 500
					INSERT INTO listaPrecioMayor VALUES (@stock_num, @manu_code, @unit_price * (1 + @porcActualizacion * 0.80), @unit_code);
				ELSE
					INSERT INTO listaPrecioMenor VALUES (@stock_num, @manu_code, @unit_price * (1 + @porcActualizacion), @unit_code);
						
						
				UPDATE products SET status= 'A' WHERE manu_code = @manu_code AND stock_num = @stock_num;

				FETCH NEXT FROM StockCursor INTO @stock_num, @manu_code, @unit_price, @unit_code
						
				IF @manu_code != @manu_codeAux AND @manu_code IS NOT NULL
					BEGIN
						COMMIT TRANSACTION
						SET @manu_codeAux = @manu_code
					END
			END TRY
			
			BEGIN CATCH
				ROLLBACK TRANSACTION
				DECLARE @errorDescripcion VARCHAR(100)
				SELECT @errorDescripcion = 'Error en Fabricante ' + @manu_code
				RAISERROR(@errorDescripcion,14,1)
			END CATCH
		END;

	CLOSE StockCursor
	DEALLOCATE StockCursor
END;

DROP PROCEDURE actualizarPrecios;

EXECUTE actualizarPrecios 'aaa', 'ZZZ', 30.5;

SELECT * FROM listaPrecioMayor;
SELECT * FROM listaPrecioMenor;

SELECT * FROM SYS.ALL_SQL_MODULES WHERE object_id=386100416
```

## Funciones

### Ejercicio 1
Escribir una sentencia SELECT que devuelva el número de orden, fecha de orden y el nombre del día de la semana de la orden de todas las órdenes que no han sido pagadas.
Si el cliente pertenece al estado de California el día de la semana debe devolverse en inglés, caso contrario en español. Cree una función para resolver este tema.
Nota: `SET @DIA = datepart(weekday, @fecha)` devuelve en la variable @DIA el nro. de día de la semana, comenzando con 1 Domingo hasta 7 Sábado.

```sql
SELECT order_num, order_date, c.state, dbo.diaSegunEstado(c.state, order_date) dia
FROM orders o INNER JOIN customer c ON o.customer_num = c.customer_num
WHERE paid_date IS NULL;


CREATE FUNCTION diaSegunEstado(@estado CHAR(2), @fecha DATE)
RETURNS VARCHAR(12)
AS 
BEGIN
    DECLARE @resultado VARCHAR(12);
    
    IF @estado = 'CA'
        SET @resultado = DATENAME(weekday, @fecha);
  
    ELSE
        SET @resultado = dbo.diaSegunNumero(DATEPART(weekday, @fecha));
    
    RETURN @resultado;
END;


CREATE FUNCTION diaSegunNumero(@numero INT)
RETURNS VARCHAR(12)
AS 
BEGIN
	RETURN 
		CASE @numero
			WHEN 1 THEN 'Domingo'
			WHEN 2 THEN 'Lunes'
			WHEN 3 THEN 'Martes'
			WHEN 4 THEN 'Miércoles'
			WHEN 5 THEN 'Jueves'
			WHEN 6 THEN 'Viernes'
			WHEN 7 THEN 'Sábado'
		END;
END;
```

### Ejercicio 2
Escribir una sentencia SELECT para los clientes que han tenido órdenes en al menos 2 meses diferentes, los dos meses con las órdenes con el mayor ship_charge.
Se debe devolver una fila por cada cliente que cumpla esa condición, el formato es:
Cliente: NNNN; Año y mes mayor carga: YYYY - Total: NNNN.NN; Segundo año mayor carga: YYYY - Total: NNNN.NN
  
La primera columna es el id de cliente y las siguientes 2 se refieren a los campos ship_date y ship_charge. Se requiere crear una función que devuelva la información de 1er o 2do año mes con la orden con mayor Carga (ship_charge).

```sql
SELECT DISTINCT customer_num, dbo.mesConCargasMayores(1, customer_num), dbo.mesConCargasMayores(2, customer_num)
FROM orders o1
WHERE EXISTS (
	SELECT 1 
	FROM orders o2 
	WHERE o2.customer_num = o1.customer_num AND MONTH(o1.order_date) > MONTH(o2.order_date)
)

ALTER FUNCTION mesConCargasMayores(@ORDEN SMALLINT, @CLIENTE INT)
RETURNS VARCHAR(100)
AS
BEGIN
	DECLARE @MES VARCHAR(4)
	DECLARE @CARGA VARCHAR(50)
	DECLARE @RETORNO VARCHAR(100)
	
	IF @ORDEN = 1
	BEGIN
		SELECT TOP 1 @MES = DATENAME(month, order_date), @CARGA = MAX(ship_charge)
		FROM orders
		WHERE customer_num = @CLIENTE
		GROUP BY order_date, ship_charge
		ORDER BY ship_charge DESC
		
		SET @RETORNO = @MES + ' - Total: ' + @CARGA
	END

	ELSE
	BEGIN
		SELECT TOP 1 @MES = order_date, @CARGA = COALESCE(ship_charge, 0)
		FROM (
			SELECT TOP 2 DATENAME(month, order_date) order_date, MAX(ship_charge) ship_charge
			FROM orders
			WHERE customer_num = @CLIENTE
			GROUP BY order_date, ship_charge
			ORDER BY 2 DESC
		) as SQL1
		ORDER BY ship_charge ASC

		SET @RETORNO = @MES + ' - Total: ' + @CARGA
	END
	
	RETURN @RETORNO
END
```

### Ejercicio 3
Escribir un SELECT que devuelva para cada producto de la tabla products que exista en la tabla catalog todos sus fabricantes separados entre sí por el caracter pipe (|). Utilizar una función para resolver parte de la consulta.
Ejemplo de la salida: 
Stock_num: 5; Fabricantes: NRG | SMT | ANZ

```sql
SELECT DISTINCT stock_num, dbo.fabricantesDeProducto(stock_num) fabricantes
FROM catalog

ALTER FUNCTION fabricantesDeProducto(@stock_num SMALLINT)
RETURNS VARCHAR(100) 
AS 
BEGIN
	DECLARE @resultado VARCHAR(100), @manu_code CHAR(3)

	DECLARE fabricantesCursor CURSOR FOR 
		SELECT DISTINCT manu_code
		FROM catalog
		WHERE stock_num = @stock_num

	OPEN fabricantesCursor;
	FETCH NEXT FROM fabricantesCursor INTO @manu_code

	SET @resultado = ''
	WHILE @@FETCH_STATUS = 0
	BEGIN
		SET @resultado += @manu_code
		FETCH NEXT FROM fabricantesCursor INTO @manu_code
		IF @@FETCH_STATUS = 0
			SET @resultado += ' | '
	END
	CLOSE fabricantesCursor
	DEALLOCATE fabricantesCursor

	RETURN @resultado
END


-- Otra alternativa mas corta para resolver la funcion

ALTER FUNCTION fabricantesDeProducto(@stock_num SMALLINT)
RETURNS VARCHAR(100)
AS 
BEGIN
    DECLARE @resultado VARCHAR(100);

    SELECT @resultado = STRING_AGG(manu_code, ' | ')
    FROM catalog
    WHERE stock_num = @stock_num;

    RETURN @resultado;
END
```