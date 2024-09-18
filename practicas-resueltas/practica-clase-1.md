# Práctica - Clase 1

Práctica realizada el día 12 de Agosto del 2024 para la primera clase de Gestión de Datos.

## Ejercicio 1
Valores S# para proveedores que proveen el proyecto J1.
```
Project{S#}  (select VPY where J# = J1)
```

## Ejercicio 2
Valores S# para proveedores que proveen el proyecto J1 c/la parte P1.
```
Project{S#}  (select VPY where J# = J1 INTERSECTION select VPY where P# = P1)
```

## Ejercicio 3
Valores JNAME para proyectos suministrados por el proveedor S1.
```
Project{Color} (P  JOIN select VPY where S# = S1)
                    P#
```

## Ejercicio 4
Valores de Color para partes suministradas por el proveedor S1.
```
Project{Color} (P  JOIN select VPY where S# = S1)
                    P#
```

## Ejercicio 5
Valores S# para proveedores que suministren los proyectos J1 y J2.
```
Project{S#}  (select VPY where J# = J1 INTERSECTION select VPY where J# = J2)
```

## Ejercicio 6
Valores S# para prov. que proveean el proyecto J1 con una parte roja.
```
Project{S#}  (select VPY where J# = J1 JOIN SELECT P WHERE color = Red)
                                        P#
```

## Ejercicio 7
Valores P# para partes suministradas a cualquier proyecto en London.
```
Project{P#}  (VPY JOIN (SELECT J where ciudad = London))
                   J#
```

## Ejercicio 8
Valores S# para proveedores que suministren a proyectos de London o Paris con una parte roja.
```
Project{S#}  (S JOIN VPY JOIN (SELECT J where ciudad = London UNION SELECT J where ciudad = Paris) JOIN (SELECT P WHERE Color = Red))
                 S#       J#                                                                        P#
```

## Ejercicio 9
Valores P# para partes suministradas a cualquier proyecto por cualquier proveedor en una misma ciudad.
```
PROJECT (PROJECT (S  JOIN  J)  JOIN  SPJ)
    P#    S#,J#     Ciudad     S#,J#

```

## Ejercicio 10
Valores S# para proveedores que suministren la misma parte a todos los proyectos. 
```
Project{S#} (Project {P#, S#} select VPY where J# = J1 JOIN SELECT P WHERE color = Red)
                                                        P#
```

## Ejercicio 11
Valores J# para proyectos que UTILICEN solo partes del provedor S1. 
```
PROJECT (SPJ) % PROJECT(SELECT(SPJ))
    J#,P#         P#      S#='S1'
```