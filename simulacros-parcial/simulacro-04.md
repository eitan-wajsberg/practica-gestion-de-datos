# Simulacro - 4 (Parcial 26/11/2020)

Simulacro realizado el día 24 de Noviembre del 2024 con el fin prepararme para el parcial de Gestión de Datos.

## Parte 1 - Teoria y SQL

### Ejercicio A
Describa 3 diferentes formas de implementar el concepto de Dominio definido por Edgar Codd, en un motor de Base de datos Relacional.

```
El dominio segun Edgar Codd es el conjunto de valores que puede tomar un atributo en una columna. Es la minima unidad semantica de informacion y es atomica, esto quiere decir que
no se pueden descomponer/separar sin perder el significado. Es un conjunto de datos escalares de un mismo tipo. Las formas de implementar el concepto de dominio en un
motor de bases de datos son:

    - Constraints: restrigen los valores posibles del dominio, los ejemplos mas caracteristicos son las Foreign Keys y CHECK, pero tambien las constraint de PK, NOT NULL, DEFAULT, entre
    otras son fundamentales.
    - Tipos de datos: restrigen el tipo de valor que puede tomar esa columna.
    - Nombres de las columnas: nos permiten comprender que datos esperamos de esa columna.
```

### Ejercicio B
Defina que es una transacción y explique su relación con la funcionalidad de consistencia en un motor de BD Relacional. 

```
Una transaccion es una operacion en una base de datos que ejecuta un conjunto de sentencias SQL de forma atomica, consistente, aislada y durable (ACID). Son atomicas porque
se ejecutan por completo una vez hecho el commit, o no se ejecutan (gracias a un rollback). Son consistentes ya que nos llevan de un estado valido hacia otro estado valido. 
Son aisladas ya que dependiendo del nivel de aislamiento indicado, nos permite ejecutar sentencias sin que otras transacciones se interpongan en el camino. Son durables ya que una
vez hecho el commit, los datos quedan almacenados / persistidos en la base y en caso de perdidas nos permiten restaurar / recuperar los datos perdidos.
Las transacciones garantizan consistencia, como nombramos anteriormente nos transladan de un estado A consistente hacia otro estado B consistente. Que un estado sea consistente implica que cumple con todas las restricciones definidas en la base de datos y en modelo especifico utilizado. 
Muchas veces se confunde el termino de integridad con el de consistencia, muchas fuentes indican que son sinonimos, pero en el contexto de las bases de datos esto no es asi.
El concepto de integridad en las bases de datos esta plasmado mediante las reglas de integridad de entidades, referencial y semantica.
```

### Ejercicio C
Crear una consulta que devuelva: Para aquellos 3 estados de mayor facturación el Código de estado, número cliente, nombre, apellido, promedio de orden de compra por cliente, total comprado por cliente y total comprado por estado. Para el cálculo de lo facturado solamente tener en cuenta aquellas líneas de ítems que hayan facturado mas de $85. Ordenar la consulta por el monto facturado total por estado en forma descendente y por monto facturado por cliente también en forma descendente.
Nota: No se pueden utilizar tablas temporales, ni funciones de usuario.

```sql
SELECT 
    c1.state, c1.customer_num, c1.fname, c1.lname, 
    SUM(i1.quantity * i1.unit_price) / COUNT(DISTINCT o1.order_num) promedio_oc_cliente, 
    SUM(i1.quantity * i1.unit_price) monto_facturado_cliente, (
        SELECT SUM(i.quantity * i.unit_price)
        FROM orders o 
            INNER JOIN customer c ON c.customer_num = o.customer_num
            INNER JOIN items i ON i.order_num = o.order_num
        WHERE c.state = c1.state
    ) monto_facturado_estado
FROM customer c1 
	INNER JOIN orders o1 ON o1.customer_num = c1.customer_num
    INNER JOIN items i1 ON i1.order_num = o1.order_num
WHERE c1.state IN (
    SELECT TOP 3 c2.state
    FROM customer c2 
        INNER JOIN orders o2 ON o2.customer_num = c2.customer_num
        INNER JOIN items i2 ON i2.order_num = o2.order_num
    GROUP BY c2.state
    ORDER BY SUM(i2.quantity * i2.unit_price) DESC
) AND i1.quantity * i1.unit_price > 85
GROUP BY c1.state, c1.customer_num, c1.fname, c1.lname
ORDER BY monto_facturado_estado DESC, monto_facturado_cliente DESC;

/*
    Atento:
        - Te pedia el promedio orden de compra del cliente en especifico no de todos.
        - Mi solucion y la del resuelto discrepan, ya que yo considero que: "Para el cálculo de lo facturado solamente tener en cuenta aquellas líneas de ítems que hayan facturado mas de $85" se refiere a un renglon especifico de la orden, no a la sumatoria de la orden por completo. En el resuelto el filtro lo ponen en el HAVING ya que consideran que se refiere a la sumatoria de cantidades por producto en una orden. 
*/
```


## Parte 2 - Stored Procedures y Triggers

### Ejercicio D
Desarrollar un script para crear la tabla CuentaCorriente con la siguiente estructura:

Id BIGINT IDENTITY, fechaMovimiento DATETIME, customer_num SMALLINT (FK), order_num INT (FK), importe DECIMAL(12,2)

Desarrollar un stored procedure que cargue la tabla cuentaCorriente de acuerdo a la información de las tablas orders e ítems. 

Por cada OC deberá cargar un movimiento con fechaMovimiento igual al order_date y el importe = SUM(quantity*total_price) de cada orden.

Por cada OC pagada cargar además un movimiento con fechaMovimiento igual al paid_date y el importe = SUM (quantity* total_price * (-1)) de cada orden

```sql
CREATE TABLE CuentaCorriente (
    Id BIGINT IDENTITY PRIMARY KEY,
    fechaMovimiento DATETIME,
    customer_num SMALLINT FOREIGN KEY REFERENCES customer (customer_num), 
    order_num SMALLINT FOREIGN KEY REFERENCES orders (order_num),
    importe DECIMAL(12,2)
)

CREATE PROCEDURE cargarCuentaCorriente AS
BEGIN
    DECLARE cursorOrderItems CURSOR FOR
        SELECT o.customer_num, o.order_num, o.paid_date, o.order_date, SUM(i.quantity * i.unit_price) importe
        FROM orders o INNER JOIN items i ON i.order_num = o.order_num
		GROUP BY o.customer_num, o.order_num, o.paid_date, o.order_date;

    DECLARE @customerNum SMALLINT;
    DECLARE @orderNum INT;
    DECLARE @paidDate DATETIME;
    DECLARE @orderDate DATETIME;
    DECLARE @importe DECIMAL(12,2);

    OPEN cursorOrderItems
    FETCH cursorOrdenItems INTO @customerNum, @orderNum, @paidDate, @orderDate, @importe

    WHILE @@FETCH_STATUS = 0
    BEGIN
        BEGIN TRANSACTION 
            BEGIN TRY
                IF @paidDate IS NULL
                    INSERT INTO CuentaCorriente VALUES (@orderDate, @customerNum, @orderNum, @importe)
                ELSE 
                    INSERT INTO CuentaCorriente VALUES (@paidDate, @customerNum, @orderNum, @importe * (-1))

                COMMIT
            END TRY

            BEGIN CATCH
                ROLLBACK
                PRINT 'Numero error: ' + CAST(ERROR_NUMBER() AS VARCHAR);
				PRINT 'Mensaje error: ' + ERROR_MESSAGE();        
            END CATCH

        FETCH cursorOrdenItems INTO @customerNum, @orderNum, @paidDate, @orderDate, @importe
    END
END

-- Otra alternativa sin cursores:
CREATE PROCEDURE cargarCuentaCorriente AS 
BEGIN
    INSERT INTO cuentaCorriente(fechaMovimiento, customer_num, order_num, importe)
    SELECT o.order_date, o.customer_num, o.order_num, SUM(quantity * unit_price)
    FROM orders o INNER JOIN items i ON o.order_num = i.order_num
    WHERE o.paid_date IS NULL
    GROUP BY o.order_num, o.order_date, o.customer_num
    
    UNION

    SELECT o.paid_date, o.customer_num, o.order_num, SUM(quantity * unit_price * (-1))
    FROM orders o INNER JOIN items i ON o.order_num = i.order_num
    WHERE o.paid_date IS NOT NULL
    GROUP BY o.order_num, o.paid_date, o.customer_num
END
```

## Ejercicio E
Dada la tabla CUSTOMER y la tabla CUSTOMER_AUDIT
    customer_num smallint not null (PK)
    update_date datetime not null (PK)
    ApeyNom_NEW varchar(40)
    State_NEW char(2)
    customer_num_referedBy_NEW smallint
    ApeyNom_OLD varchar(40)
    State_OLD char(2)
    customer_num_referedBy_OLD smallint
    update_user varchar(30) not null

Ante deletes y updates de los campos lname, fname, state o customer_num_refered de la tabla CUSTOMER, auditar los cambios colocando en los campos NEW los valores nuevos y guardar en los campos OLD los valores que tenían antes de su borrado/modificación.

En los campos ApeyNom se deben guardar los nombres y apellidos concatenados respectivos.

En el campo update_date guardar la fecha y hora actual y en update_user el usuario que realiza el update.

Verificar en las modificaciones la validez de las claves foráneas ingresadas y en caso de error informarlo y deshacer la operación.

Notas: Asumir que ya existe la tabla de auditoría. Las modificaciones pueden ser masivas y en caso de error sólo se debe deshacer la operación actual.

```sql
CREATE TABLE auditoriaCustomer (
    customer_num smallint not null FOREIGN KEY REFERENCES customer (customer_num),
    update_date datetime not null,
    ApeyNom_NEW varchar(40),
    State_NEW char(2) FOREIGN KEY REFERENCES state (state),
    customer_num_referedBy_NEW smallint FOREIGN KEY REFERENCES customer (customer_num),
    ApeyNom_OLD varchar(40),
    State_OLD char(2) char(2) FOREIGN KEY REFERENCES state (state),
    customer_num_referedBy_OLD smallint FOREIGN KEY REFERENCES customer (customer_num),
    update_user varchar(30) not null,
    PRIMARY KEY (customer_num, update_date)
)

ALTER TRIGGER auditarClientes ON customer
AFTER UPDATE, DELETE AS
BEGIN
    BEGIN TRY
        -- Auditoría para DELETE
        IF EXISTS (SELECT * FROM DELETED)
        BEGIN
            INSERT INTO auditoriaCustomer (customer_num, update_date, ApeyNom_OLD, State_OLD, customer_num_referedBy_OLD, ApeyNom_NEW, State_NEW, customer_num_referedBy_NEW, update_user)
            SELECT 
                D.customer_num, GETDATE(), D.fname + ' ' + D.lname ApeyNom_OLD, 
                D.state  State_OLD, D.customer_num_referedBy customer_num_referedBy_OLD,
                NULL ApeyNom_NEW, NULL State_NEW, NULL customer_num_referedBy_NEW, SYSTEM_USER  update_user
            FROM DELETED D;
        END

        -- Auditoría para UPDATE
        IF EXISTS (SELECT * FROM INSERTED)
        BEGIN
            -- Verificación de claves foráneas
            IF EXISTS (
                SELECT 1
                FROM INSERTED I
                WHERE I.customer_num_referedBy IS NOT NULL AND NOT EXISTS (
                    SELECT 1 
                    FROM CUSTOMER C 
                    WHERE C.customer_num = I.customer_num_referedBy
                )
            )
            BEGIN
                THROW 50000, 'Error: Clave foránea no válida, se ingreso un customer_num_referedBy que no existe en la tabla de customers.', 1;
            END

            INSERT INTO auditoriaCustomer (customer_num, update_date, ApeyNom_OLD, State_OLD, customer_num_referedBy_OLD, ApeyNom_NEW, State_NEW, customer_num_referedBy_NEW, update_user)
            SELECT 
                D.customer_num, GETDATE(), D.fname + ' ' + D.lname ApeyNom_OLD, D.state State_OLD, 
                D.customer_num_referedBy customer_num_referedBy_OLD, I.fname + ' ' + I.lname ApeyNom_NEW, I.state State_NEW,
                I.customer_num_referedBy customer_num_referedBy_NEW, SYSTEM_USER  update_user
            FROM DELETED D
            INNER JOIN INSERTED I ON D.customer_num = I.customer_num;
        END
    END TRY

    BEGIN CATCH
        DECLARE @MensajeError NVARCHAR(4000);
        SELECT @MensajeError = ERROR_MESSAGE();
        RAISERROR (@MensajeError, 16, 1);
        ROLLBACK TRANSACTION;
    END CATCH
END;
```