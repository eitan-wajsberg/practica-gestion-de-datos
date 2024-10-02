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
CREATE PROCEDURE actualizarPrecios(@manu_codeDES CHAR(3), @manu_codeHAS CHAR(3), @porcActualizacion DECIMAL(5,3))
AS
BEGIN
	DECLARE cursor_precios CURSOR FOR 
		SELECT p.manu_code, SUM(i.quantity) cantidad_comprada
		FROM products p
			INNER JOIN items i ON(i.stock_num = p.stock_num AND i.manu_code = i.manu_code)
		WHERE p.manu_code BETWEEN @manu_codeDES AND @manu_codeHAS
		GROUP BY p.manu_code
	
	DECLARE @manu_code CHAR(3);
	DECLARE @cantidad_comprada INT;
	
	BEGIN TRANSACTION
		BEGIN TRY
			OPEN cursor_precios;
			FETCH cursor_precios INTO @manu_code, @cantidad_comprada;
			WHILE (@@FETCH_STATUS = 0)
				BEGIN
					IF @cantidad_comprada >= 500
						BEGIN
							INSERT INTO listaPrecioMayor SELECT stock_num, @manu_code, unit_price * (1 + @porcActualizacion * 0.80), unit_code FROM products WHERE manu_code=@manu_code;							
						END;
						
					ELSE IF @cantidad_comprada < 500
						BEGIN
							INSERT INTO listaPrecioMenor SELECT stock_num, @manu_code, unit_price * (1 + @porcActualizacion), unit_code FROM products WHERE manu_code=@manu_code;
						END;
					
					UPDATE products SET status='A' WHERE manu_code = @manu_code;
					FETCH cursor_precios INTO @manu_code, @cantidad_comprada;
				END;
			COMMIT;
		END TRY

		BEGIN CATCH
			PRINT 'Numero error: ' + CAST(ERROR_NUMBER() AS VARCHAR);
			PRINT 'Mensaje: ' + ERROR_MESSAGE();
			ROLLBACK;
		END CATCH;
	
		CLOSE cursor_precios;
		DEALLOCATE cursor_precios;
END;

DROP PROCEDURE actualizarPrecios;

EXECUTE actualizaPrecios 'aaa', 'ZZZ', 30;

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
```

### Ejercicio 2
Escribir una sentencia SELECT para los clientes que han tenido órdenes en al menos 2 meses diferentes, los dos meses con las órdenes con el mayor ship_charge.
Se debe devolver una fila por cada cliente que cumpla esa condición, el formato es:
Cliente: NNNN; Año y mes mayor carga: YYYY - Total: NNNN.NN; Segundo año mayor carga: YYYY - Total: NNNN.NN
  
La primera columna es el id de cliente y las siguientes 2 se refieren a los campos ship_date y ship_charge. Se requiere crear una función que devuelva la información de 1er o 2do año mes con la orden con mayor Carga (ship_charge).

```sql
```

### Ejercicio 3
Escribir un Select que devuelva para cada producto de la tabla Products que exista en la tabla Catalog todos sus fabricantes separados entre sí por el caracter pipe (|). Utilizar una función para resolver parte de la consulta.
Ejemplo de la salida: 
Stock_num: 5; Fabricantes: NRG | SMT | ANZ

```sql
```