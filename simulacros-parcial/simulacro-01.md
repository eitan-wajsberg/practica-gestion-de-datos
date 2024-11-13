# Simulacro - 1

Simulacro realizada el día 06 de Noviembre del 2024 para la decimo tercera clase de Gestión de Datos.

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

```sql
```

## Parte 2 - Stored Procedures y Triggers

### Ejercicio D
Crear un procedimiento actualizaPrecios que reciba como parámetro una fecha a partir de la cual procesar los registros de una tabla Novedades que contiene los nuevos precios de Productos con la siguiente estructura/información: FechaAlta, Manu_code, Stock_num, descTipoProducto, Unit_price. Por cada fila de la tabla Novedades:
1. Si no existe el Fabricante, devolver un error de Fabricante inexistente y descartar la novedad. 
2. Si no existe el stock_num (pero existe el Manu_code) darlo de alta en la tabla Product_types.
3. Si ya existe el Producto actualizar su precio. Si no existe, Insertarlo en la tabla de productos. 
Nota: Manejar una transacción por novedad y errores no contemplados.

```sql
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