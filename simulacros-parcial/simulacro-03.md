# Simulacro - 3 (Recuperatorio 30/07/2019)

Simulacro realizado el día 23 de Noviembre del 2024 con el fin de prepararme para el parcial de Gestión de Datos.

## Parte 1 - Teoria y SQL

### Ejercicio A
¿Que es una base de datos? ¿Que es un Sistema de Administración de Base de Datos?

```
Una base de datos relacional es un conjunto de datos persistentes e interrelacionados, evitando en la mayoria de los casos las redundancias gracias a la normalizacion. 
Las bases de datos son utilizadas por sistemas de las empresas y son consultadas a medida que el sistema lo necesita.

Un sistema de adminstracion de bases de datos, tambien llamado motor de bases de datos es un programa encargado de administrar las interacciones con la base. Interpreta y ejecuta los comandos SQL.
Este motor esta compuesto por distintas capas o niveles que conforman su arquitectura:
1. Nivel externo: es la percepcion que tienen los usuarios (programadores, usuarios finales, DBA) respecto de la base de datos. Por ejemplo para un programador o un DBA, la vista seria en forma de tablas.
2. Nivel conceptual: representa la informacion contenida en la base de datos, se define mediante DDL (Data Definition Language). Contiene definicion de tipos de datos, restricciones, reglas de integridad.
3. Nivel interno: define como se almacenan los datos en el disco, es una representacion de bajo nivel de la base de datos.
Algunos ejemplos de motor de bases de datos son: Oracle, Informix, SQL Server, MySQL, PostgreSQL, entre otros.
```

### Ejercicio B
Escriba todo lo que sepa sobre VIEWs: definicion, usos, restricciones, etc.

```
Una vista es un objeto de bases de datos que contiene una consulta de una o varias tablas. Es independiente tanto fisica como logicamente de la tabla o tablas a las cuales referencia. 
Puede ser utilizada como mecanismo de seguridad para restringir el acceso a ciertas columnas o filas en especifico.
Nos aporta una perspectiva distinta de los datos, es decir que puede simplificar y de alguna manera personalizar como se ven los datos de una tabla.
Es importante aclarar que NO OCUPA ESPACIO DE ALMACENAMIENTO. 
Al actualizar la vista es posible que se generen inconsistencias si es que no utilizamos WITH CHECK OPTION, que nos permite asegurar que todas las actualizaciones que se le hagan a la vista sean datos que estan disponibles en la misma. Al no utilizar esta opcion seria posible modificar datos que no estan presentes en la vista. 
Si se hace un insert en la vista, el cual no tiene algunos campos obligatorios en la tabla, el mismo fallaria.
Restricciones:
    - No es posible utilizar ORDER BY ni UNION en una vista.
    - NO SE PUEDEN CREAR INDICES EN UNA VISTA.
    - Algunas VIEWs tienen restringido hacer inserts, deletes y updates en triggers con INSTEAD OF.
    - Para crear una view el usuario debe tener permiso de consultar las columnas que conformaran la vista.
```

### Ejercicio C
Mostrar código y descripción del Estado, código y descripción del tipo de producto y la cantidad de unidades vendidas del tipo de producto, de los tipos de productos más comprados (por cantidad) en cada Estado. Mostrar el resultado ordenado por el nombre (o Descripción) del Estado.

```sql
SELECT s.state, s.sname, I.stock_num, pt.description, SUM(i.quantity) cantidad_unidades_vendidas
FROM state s
    INNER JOIN customer c ON c.state = s.state
    INNER JOIN orders o ON o.customer_num = c.customer_num
    INNER JOIN items i ON i.order_num = o.order_num
    INNER JOIN product_types pt ON pt.stock_num = i.stock_num
WHERE pt.stock_num IN (
    SELECT TOP 1 i.stock_num
    FROM items i
		INNER JOIN orders o ON o.order_num = i.order_num
        INNER JOIN customer c ON c.customer_num = o.customer_num
    WHERE c.state = s.state
	GROUP BY i.stock_num
    ORDER BY SUM(i.quantity) DESC
)
GROUP BY s.state, s.sname, I.stock_num, pt.description
ORDER BY s.sname;
```

## Parte 2 - Stored Procedures y Triggers

### Ejercicio D
Crear un procedimiento actualizaCliente el cuál tomará de una tabla “clientesAltaOnline” previamente cargada por otro proceso, la siguiente información: Customer_num, lname, fname, Company, address1, city, state

Por cada fila de la tabla clientesAltaOnline se deberá evaluar:
- Si el cliente existe en la tabla Customer, se deberá modificar dicho cliente en la tabla Customer con los datos leídos de la tabla clientesAltaOnline.
- Si el cliente no existe en la tabla customer, se deberá insertar el cliente en la tabla Customer con los datos leídos de la tabla clientesAltaOnline.

El procedimiento deberá almacenar por cada operación realizada una fila en la una tabla Auditoría con los siguientes atributos: IdAuditoria (Identity), operación (I ó M), customer_num, lname, fname, address1, city, state.
Manejar UNA transacción por cada novedad.

```sql
SELECT * INTO clientesAltaOnline FROM customer WHERE 1=2;

CREATE TABLE auditoriaCliente (
    idAuditoria INT IDENTITY(1, 1) PRIMARY KEY,
    operacion CHAR(1) CHECK(operacion IN ('I', 'M')),
    customer_num SMALLINT FOREIGN KEY REFERENCES customer (customer_num),
    lname VARCHAR(15),
    fname VARCHAR(15),
    address1 VARCHAR(20),
    city VARCHAR(15),
    state CHAR(2) FOREIGN KEY REFERENCES state (state),
);

CREATE PROCEDURE actualizaCliente AS
BEGIN
    DECLARE @customerNum SMALLINT;
    DECLARE @lname VARCHAR(15);
    DECLARE @fname VARCHAR(15);
    DECLARE @company VARCHAR(20);
    DECLARE @address1 VARCHAR(20);
    DECLARE @city VARCHAR(15);
    DECLARE @state CHAR(2);
     
    DECLARE cursorClientes CURSOR FOR
        SELECT customer_num, lname, fname, company, address1, city, state
        FROM clientesAltaOnline

    OPEN cursorClientes
    FETCH cursorClientes INTO @customerNum, @lname, @fname, @company, @address1, @city, @state

    WHILE @@FETCH_STATUS = 0
    BEGIN
        BEGIN TRANSACTION
            BEGIN TRY
                IF EXISTS (SELECT 1 FROM customer WHERE customer_num = @customerNum)
                BEGIN
                    UPDATE customer SET lname = @lname, fname = @fname, company = @company, address1 = @address1, city = @city, state = @state WHERE customer_num = @customerNum
                    INSERT INTO auditoriaCliente VALUES ('M', @customerNum, @lname, @fname, @address1, @city, @state)
                END
                
                ELSE
                BEGIN
                    INSERT INTO customer (customer_num, lname, fname, city, company, address1, state) VALUES (@customerNum, @lname, @fname, @city, @company, @address1, @state)
                    INSERT INTO auditoriaCliente VALUES ('I', @customerNum, @lname, @fname, @address1, @city, @state)
                END

                COMMIT
            END TRY

            BEGIN CATCH
                ROLLBACK
                PRINT 'Numero error: ' + CAST(ERROR_NUMBER() AS VARCHAR);
				PRINT 'Mensaje: ' + ERROR_MESSAGE();
            END CATCH

        FETCH cursorClientes INTO @customerNum, @lname, @fname, @company, @address1, @city, @state
    END

	CLOSE cursorClientes
	DEALLOCATE cursorClientes
END
```

## Ejercicio E
Dada la vista:
```sql
CREATE VIEW OrdenItems AS 
    SELECT o.order_num, o.order_date, o.customer_num, o.paid_date, i.item_num, i.stock_num, i.manu_code, i.quantity, i.unit_price 
    FROM orders o INNER JOIN items i ON o.order_num = i.order_num;
```

Se quieren controlar las altas sobre la vista anterior. Los controles a realizar son los siguientes:
- No se permitirá que una Orden contenga ítems de fabricantes de más de dos estados en la misma orden. 
- Por otro parte los Clientes del estado de ALASKA no podrán realizar compras a fabricantes fuera de ALASKA.

Notas:
- Las altas son de una misma Orden y de un mismo Cliente pero pueden tener varias líneas de ítems.
- Ante el incumplimiento de una validación, deshacer TODA la transacción.

```sql
CREATE TRIGGER controlAltaOrdenItems ON OrdenItems
INSTEAD OF INSERT AS
BEGIN
    IF EXISTS (
        SELECT 1 FROM inserted i INNER JOIN manufact m ON i.manu_code = m.manu_code 
        GROUP BY m.state
        HAVING COUNT(DISTINCT m.state) > 2
    )
        THROW 50000, 'No se permite que una orden tenga items de fabricantes de dos estados distintos', 1 
    
    IF EXISTS (
        SELECT 1 FROM inserted i  
            INNER JOIN customer c ON c.customer_num = i.customer_num 
            INNER JOIN manufact m ON m.manu_code = i.manu_code
        WHERE c.state = 'AL' AND m.state <> 'AL'
    )
        THROW 50000, 'Los clientes de ALASKA no pueden comprarle a fabricantes fuera de ALASKA', 1 
END
```
