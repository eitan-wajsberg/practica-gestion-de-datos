# Simulacro - 5 (Parcial 14/07/2019)

Simulacro realizado el día 24 de Noviembre del 2024 para el parcial de Gestión de Datos.

## Parte 1 - Teoria y SQL

### Ejercicio A
Explique las dos Reglas de Integridad según Edgar Codd. Indique dos formas distintas de implementar la Regla de Integridad Referencial en un Motor de BD.

```
Las dos reglas de integridad definidas por Edgar Codd son:

1. Regla de integridad de entidades: indica que un registro/fila de una tabla debe poder identificado univocamente (no pueden existir duplicados) y no aceptar nulos. 
   En un motor de bases de datos esta regla es implementada por las Primary Keys.

2. Regla de integridad referencial: indica que un registro/fila de una tabla (FK) que hace referencia a un registro/fila de otra tabla (PK), dicho registro debe 
   existir y no ser nulo. De esta forma no se hara referencia a registros inexistentes y no se podra borrarlos si son referenciados, a menos que se indice que se 
   quiere un DELETE ON CASCADE.

Dos formas distintas de implementar la regla de integridad en un motor de bases de datos:
1. La opcion mas comun y recomendable es mediante FKs.
2. Una opcion muy poco recomendable es mediante un trigger, lo podrian hacer es que antes de que se inserte un registo el cual hace referencia a otro que no existe, 
   se crea el registro y por ende no se viola la integridad referencial.
```

### Ejercicio B
Explique y ejemplifique al menos 3 objetos de BD que permitan implementar la funcionalidad de integridad de un Motor de DB

```
Tres objetos de base de datos que permiten implementar la integridad:

1. Triggers: Son procedimientos que se ejecutan automáticamente ante eventos específicos como INSERT, UPDATE o DELETE. 
   Contribuyen a la integridad al aplicar reglas personalizadas según el modelo de negocio. 

2. Vistas: Son consultas almacenadas que presentan una vista limitada o transformada de los datos. Contribuyen a la integridad restringiendo el acceso a 
   información sensible o aplicando validaciones mediante reglas definidas en la consulta. Ejemplo: Una vista que solo exponga clientes activos para evitar 
   modificaciones no autorizadas sobre otros registros (es necesario el uso de WITH CHECK OPTION para esta situacion).

1. Índices: Son estructuras de datos que optimizan la búsqueda y recuperación de información. Ayudan a la integridad al respaldar restricciones como 
   PRIMARY KEY o UNIQUE, garantizando la unicidad y evitando valores duplicados. Ejemplo: Un índice sobre una columna con la restricción UNIQUE para evitar valores repetidos.
```

### Ejercicio C
Mostrar Nombre, Apellido y promedio de orden de compra del cliente referido, nombre Apellido y promedio de orden de compra del cliente referente. De todos aquellos referidos cuyo promedio de orden de compra sea mayor al de su referente. Mostrar la información ordenada por Nombre y Apellido del referido. 
El promedio es el total de monto comprado (p x q) / cantidad de órdenes. 
Si el cliente no tiene referente, no mostrarlo.
Notas: No usar Store procedures, ni funciones de usuarios, ni tablas temporales. 

```sql
SELECT 
    s.fname nombre_referido, s.lname apellido_referido, s.promedio_oc_referido, 
    c1.fname nombre_referente, c1.lname apellido_referente, 
    SUM(i1.quantity * i1.unit_price) / COUNT(DISTINCT o1.order_num) promedio_oc_referente 
FROM customer c1 
    INNER JOIN orders o1 ON o1.customer_num = c1.customer_num
    INNER JOIN items i1 ON i1.order_num = o1.order_num
    INNER JOIN (
        SELECT c2.customer_num_referedBy, c2.fname, c2.lname, SUM(i2.quantity * i2.unit_price) / COUNT(DISTINCT o2.order_num) promedio_oc_referido
        FROM customer c2
            INNER JOIN orders o2 ON o2.customer_num = c2.customer_num
            INNER JOIN items i2 ON i2.order_num = o2.order_num
		GROUP BY c2.customer_num_referedBy, c2.fname, c2.lname
    ) s ON s.customer_num_referedBy = c1.customer_num
GROUP BY s.fname, s.lname, c1.fname, c1.lname, s.promedio_oc_referido
HAVING SUM(i1.quantity * i1.unit_price) / COUNT(DISTINCT o1.order_num) < s.promedio_oc_referido
ORDER BY s.fname, s.lname;
```

## Parte 2 - Stored Procedures y Triggers

### Ejercicio D
Dada la siguiente tabla de auditoria:
```sql
CREATE TABLE audit_fabricante(
    nro_audit BIGINT IDENTITY PRIMARY KEY,
    fecha DATETIME DEFAULT getDate(),
    accion CHAR(1) CHECK (accion IN ('I','O','N','D')),
    manu_code char(3),
    manu_name varchar(30),
    lead_time smallint,
    state char(2),
    usuario VARCHAR(30) DEFAULT USER,
);
```
Se pide realizar un proceso de “rollback” que realice las operaciones inversas a las leídas en la tabla de auditoría hasta una fecha y hora enviada como parámetro.
Si es una accion de Insert ("I"), se deberá hacer un Delete.
Si es una accion de Update, se deberán modificar la fila actual con los datos cuya accion sea "O" (Old). 
Si la acción es un delete "D", se deberá insertar el registro en la tabla.
Las filas a “Rollbackear” deberán ser tomados desde el instante actual hasta la fecha y hora pasada por parámetro.
En el caso que por cualquier motivo haya un error, se deberá cancelar la operación completa e informar el mensaje de error.

```sql
CREATE PROCEDURE rollbackAuditoriaFabricante (@fechaHasta DATETIME) AS
BEGIN
    BEGIN TRANSACTION
    DECLARE cursorAuditoriaFabricante CURSOR FOR
        SELECT accion, manu_code, manu_name, lead_time, state  
        FROM audit_fabricante 
        WHERE fecha BETWEEN GETDATE() AND @fechaHasta
     
    DECLARE @accion CHAR(1);
    DECLARE @manuCode CHAR(3);
    DECLARE @manuName VARCHAR(30);
    DECLARE @leadTime SMALLINT;
    DECLARE @state CHAR(2);

    OPEN cursorAuditoriaFabricante;
    FETCH cursorAuditoriaFabricante INTO @accion, @manuCode, @manuName, @leadTime, @state

    WHILE @@FETCH_STATUS = 0 
    BEGIN
        BEGIN TRY
            IF @accion = 'I'
                DELETE FROM manufact 
                WHERE manu_code = @manuCode
            ELSE IF @accion = 'O'
                UPDATE manufact 
                SET manu_name = @manuName, lead_time = @leadTime, state = @state
                WHERE manu_code = @manuCode
            ELSE IF @accion = 'D'
                INSERT INTO manufact (manu_code, manu_name, lead_time, state)
                VALUES (@manuCode, @manuName, @leadTime, @state)
            ELSE
                THROW 5000, 'Ocurrio un error al realizar un rollback de la tabla manufact.', 1 
        END TRY

        BEGIN CATCH
            ROLLBACK
            PRINT 'Mensaje error: ' + ERROR_MESSAGE()
        END CATCH

        FETCH cursorAuditoriaFabricante INTO @accion, @manuCode, @manuName, @leadTime, @state 
    END
    COMMIT

    CLOSE cursorAuditoriaFabricante
    DEALLOCATE cursorAuditoriaFabricante
END
```

## Ejercicio E
El responsable del área de ventas nos informó que necesita cambiar el sistema para que a partir de ahora no se borren físicamente las órdenes de compra sino que el borrado sea lógico.

Nuestro gerente solicitó que este requerimiento se realice con triggers pero sin modificar el código del sistema actual.

Para ello se agregaron 3 atributos a la tabla ORDERS, flag_baja (0 false / 1 baja lógica), fecha_baja (fecha de la baja), user_baja (usuario que realiza la baja).

Se requiere realizar un trigger que cuando se realice una baja que involucre uno o más filas de la tabla ORDERS, realice la baja lógica de dicha/s fila/s.

Solo se podrán borrar las órdenes que pertenezcan a clientes que tengan menos de 5 órdenes.

Para los clientes que tengan 5 o más ordenes se deberá insertar en una tabla BorradosFallidos el customer_num, order_num, fecha_baja y user_baja.

Nota: asumir que ya existe la tabla BorradosFallidos y la tabla ORDERS está modificada. Ante algún error informarlo y deshacer todas las operaciones.

```sql
ALTER TABLE orders ADD flag_baja BIT
ALTER TABLE orders ADD fecha_baja DATETIME
ALTER TABLE orders ADD user_baja VARCHAR(100)

CREATE TABLE borradosFallidos (
    customer_num SMALLINT FOREIGN KEY REFERENCES customer (customer_num),
    order_num SMALLINT FOREIGN KEY REFERENCES orders (order_num),
    fecha_baja DATETIME,
    user_baja VARCHAR(100)
)

CREATE TRIGGER realizarBajaLogica ON orders
INSTEAD OF DELETE AS
BEGIN
    UPDATE orders SET flag_baja = 1, fecha_baja = GETDATE(), user_baja = USER_NAME()
    WHERE order_num IN (
        SELECT d.order_num
        FROM deleted d
        WHERE (
            SELECT COUNT(*) 
            FROM orders 
            WHERE customer_num = d.customer_num AND flag_baja = 0
        ) < 5
    );

    INSERT INTO borradosFallidos (customer_num, order_num, fecha_baja, user_baja)
    SELECT d.customer_num, d.order_num, GETDATE(), USER_NAME()
    FROM deleted d
    WHERE (
        SELECT COUNT(*) 
        FROM orders 
        WHERE customer_num = d.customer_num AND flag_baja = 0
    ) >= 5;
END;

-- Como los triggers ya implementan transacciones no es necesario manejarlas dentro del mismo. 
-- Por ende, una vez que ocurre un error esta garantizado que se hara un rollback de las operaciones
-- hechas hasta el momento.
```