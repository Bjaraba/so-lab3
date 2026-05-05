# Laboratorio 3 Sistemas operativos 

## Punto 1 
Ejecución del programa
![ejecucion del programa](/punto-1/images/program-execution.png)

Mapa de memoria del proceso
![mapa de memoria](/punto-1/images/data-map.png)


Resumen de las regiones de datos
![resumen](/punto-1/images/summary.png)

preguntas 

*Identifique en la salida de `/proc/maps` las regiones text, heap y stack. ¿Qu´e permisos
(r/w/x/p) tiene cada una? ¿Por qu´e difieren?*

1. **Para el stack:** 

```
7ffcb72f7000-7ffcb7319000 rw-p 00000000 00:00 0                          [stack]
```

se ve que tiene los permiso de leer y escribir pues este hace parte de la data que puede ser modficada y leida durante la ejecucion

2. **Para el heap:** 
```
7ffcb72f7000-7ffcb7319000 rw-p 00000000 00:00 0                          [stack]
```
tenemos los mismos permisos `r` y `w` para lectura y escritura porque estos datos necesitan ser leidos y modificados durante la ejecucion

3. **Para el text:**

```
00005c5ce3a5f000       4       4       0 r-x-- mem_map
```

Este tiene permisos para lectura y ejecucion, pues el codigo debe tener permiso de ejecucion para poder ser ejecutado en el computador

*Compare las direcciones impresas con los rangos de `/proc/maps`. ¿A qu´e regi´on pertenece
cada variable?*

Direcciones de las variables: 

```
Dir. global_var : 0x5c5ce3a62010
Dir. local_var : 0x7ffcb7315c0c
Dir. heap_var : 0x5c5d16eca2a0
```

- `global_var: 0x5c5ce3a62010`: Esta variable esta dentro del rango de `00005c5ce3a62000       4       4       4 rw--- mem_map` que es una parte del codigo con permisos de ejecucion.  Muy probablemente la parte del `.data` donde se guardan las variables globales


- `local_var : 0x7ffcb7315c0c`: Esta varaible esta dentro de este rango `7ffcb72f7000-7ffcb7319000 rw-p 00000000 00:00 0                          [stack]` que esta reservado para el stack lo cual coincide con lo visto en clases *-las variables locales son alojadas en el stack-*

- `heap_var : 0x5c5d16eca2a0`: Esta variable esta dentro del rango `5c5d16eca000-5c5d16eeb000 rw-p 00000000 00:00 0                          [heap]` correspondiente al espacio del heap


*¿Qu´e otras regiones aparecen en el mapa `(libc, [vdso], [vsyscall])`? ¿Que funcion
cumple cada una?*