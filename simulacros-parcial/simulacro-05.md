# Simulacro - 3 (Parcial 14/07/2019)

Simulacro realizado el día 20 de Noviembre del 2024 para la decimo cuarta clase de Gestión de Datos.

## Parte 1 - Teoria y SQL

### Ejercicio A
Explique las dos Reglas de Integridad según Edgar Codd. Indique dos formas distintas de implementar la Regla de Integridad Referencial en un Motor de BD.

```
```

### Ejercicio B
Explique y ejemplifique al menos 3 objetos de BD que permitan implementar la funcionalidad de integridad de un Motor de DB

```
```

### Ejercicio C
Mostrar Nombre, Apellido y promedio de orden de compra del cliente referido, nombre Apellido y promedio de orden de compra del cliente referente. De todos aquellos referidos cuyo promedio de orden de compra sea mayor al de su referente. Mostrar la información ordenada por Nombre y Apellido del referido. 
El promedio es el total de monto comprado (p x q) / cantidad de órdenes. 
Si el cliente no tiene referente, no mostrarlo.
Notas: No usar Store procedures, ni funciones de usuarios, ni tablas temporales. 

```sql

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

```