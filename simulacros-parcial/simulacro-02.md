# Simulacro - 2 - Parcial 12/07/2023

Simulacro realizado el día 13 de Noviembre del 2024 para la decimo cuarta clase de Gestión de Datos.

## Ejercicio 1
Por cada estado (state) seleccionar los dos clientes que mayores montos compraron. Se deberá mostrar el código del estado, nro de cliente, nombre y apellido del cliente y monto total comprado. Mostrar la información ordenada por provincia y por monto comprado en forma descendente. Notas: No se puede usar Store procedures, ni funciones de usuarios, ni tablas temporales.

```sql
```

## Ejercicio 2
Crear un procedimiento BorrarProd que en base a una tabla ProductosDeprecados que  contiene filas con Productos a borrar realice la eliminación de los mismos de la tabla Products. El procedimiento deberá guardar en una tabla de auditoria AuditProd (stock_num, manu_code, Mensaje) el producto y un mensaje que podrá ser: ‘Borrado’, ‘Producto con ventas’ o cualquier mensaje de error que se produjera. Crear las tablas ProductosDeprecados y AuditProd. Deberá manejar una transacción por registro. Ante un error deshacer lo realizado y seguir  procesando los demás registros. Asimismo, deberá manejar excepciones ante cualquier erro que ocurra.

```sql
```

## Ejercicio 3
Crear un trigger que ante un cambio de precios en un producto inserte un nuevo registro con el precio anterior (no el nuevo) en la tabla PRECIOS_HIST. La estructura de la tabla PRECIOS_HIST es (stock_num, manu_code, fechaDesde, fechaHasta, precio_unit). La fecha desde del nuevo registro será la fecha hasta del último cambio de precios de ese producto y su fecha hasta será la fecha del dia. SI no tuviese un registro de precio anterior ingrese como fecha desde ‘2000-01-01’. Nota: Las actualizaciones de precios pueden ser masivas.

```sql
```
