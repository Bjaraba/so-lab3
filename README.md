# Laboratorio 3 Sistemas operativos 

## Punto 1: Espacio de direcciones
Ejecución del programa
![ejecucion del programa](/punto-1/images/program-execution.png)

Mapa de memoria del proceso
![mapa de memoria](/punto-1/images/data-map.png)


Resumen de las regiones de datos
![resumen](/punto-1/images/summary.png)

preguntas 

### Identifique en la salida de `/proc/maps` las regiones text, heap y stack. ¿Qu´e permisos (r/w/x/p) tiene cada una? ¿Por qu´e difieren?

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

### Compare las direcciones impresas con los rangos de `/proc/maps`. ¿A qu´e regi´on pertenece cada variable?

Direcciones de las variables: 

```
Dir. global_var : 0x5c5ce3a62010
Dir. local_var : 0x7ffcb7315c0c
Dir. heap_var : 0x5c5d16eca2a0
```

- `global_var: 0x5c5ce3a62010`: Esta variable esta dentro del rango de `00005c5ce3a62000       4       4       4 rw--- mem_map` que es una parte del codigo con permisos de ejecucion.  Muy probablemente la parte del `.data` donde se guardan las variables globales


- `local_var : 0x7ffcb7315c0c`: Esta varaible esta dentro de este rango `7ffcb72f7000-7ffcb7319000 rw-p 00000000 00:00 0                          [stack]` que esta reservado para el stack lo cual coincide con lo visto en clases *-las variables locales son alojadas en el stack-*

- `heap_var : 0x5c5d16eca2a0`: Esta variable esta dentro del rango `5c5d16eca000-5c5d16eeb000 rw-p 00000000 00:00 0                          [heap]` correspondiente al espacio del heap


### ¿Qu´e otras regiones aparecen en el mapa `(libc, [vdso], [vsyscall])`? ¿Que funcioncumple cada una?

Dentro del mapa se ven las otras regiones como las siguientes: 

- `libc.so.6`: la cual es la biblioteca estandar de `C` que es super importante en linux, donde estan utilidades como el `printf`, `scanf`, `malloc`, `free`, `strlen`, `strcpy`, `open`, `read`, `write` y otras. Este debe tener permisos de lectura y ejecucion para las partes del codigo.

- `ld-linux-x86-64.so.2`: Es un cargador dinamico. que carga las librerias de c antes de que la funcion `main()` se ejecutada, tambien resuelve simpbolos (enlaza las funciones) y prepara el proceso. En pocas palabras este hace posible el uso de las librerias dinamicas.

- `vvar`: es una region donde el kernel pone datos del sistema para que sean accesibles rapidamente. Como el timestamp acutal y la informacion del sistema. Basicamente es una zona de datos compartidos optimizada

- `vdso`: Llamada Virtual Dynamic Shared Object, este permite que algunas de las llamadas al sistema se ejecuten si cambar al modo kernel, tales como `gettimeofday` o `clock_gettime`, esta diseñada para optimizar el rendimiento de estas llamadas a sistema.

### ¿Son las direcciones virtuales iguales a las fisicas? Explique apoyandose en el concepto de address space del OSTEP.

No, el libro OSTEP dice que la memoria virtual es una abstraccion de la memoria, no la memoria fisica en si. Pues cada programa tiene una vista limitada de la memoria que es llamada el `address space` y esta es limitada, el programa no puede ver todo el espacio de memoria fisica.

### Ejecucion de dos instancias del programa al mismo tiempo 
![ejecucion de dos instancias](/punto-1/images/two-instaces.png)


### ¿Son las mismas direcciones virtuales en ambos procesos? ¿Qu´e conclusi´on saca sobre el aislamiento del espacio de direcciones?

En mi caso las direcciones son diferentes y no se translapan. A pesar de que estas direcciones hagan referencia a direcciones virtuales se puede ver que el sistema separa y se encarga de la seguridad de los datos cuando se ejecutan dos programas.

### ¿Podr´ıa el Proceso A leer o modificar la variable global del Proceso B mediante su direcci´on virtual? Justifique.

No, porque las direcciones virutales se mapean de forma diferente a la memoria fisica para cada uno de los procesos. Por eso, si dos procesos tienen la misma direccion virtual, esta correspondera a direcciones diferentes en la memoria fisica. Asi que es imposible que un proceso modifique la data de otro proceso.

## Punto 2: Api de memoria

![ejecucion de los comandos](/punto-2/images/execution.png)

### Muestre la salida completa de Valgrind. ¿Reporta errores o fugas de memoria? ¿Que significa el mensaje "All heap blocks were freed"?

```
valgrind --leak-check=full --track-origins=yes ./heap_demo
==16454== Memcheck, a memory error detector
==16454== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==16454== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==16454== Command: ./heap_demo
==16454== 
Arreglo original: 0 1 4 9 16 25 36 49 64 81 
Arreglo ampliado: 0 1 4 9 16 25 36 49 64 81 100 121 144 169 196 225 256 289 324 361 
==16454== 
==16454== HEAP SUMMARY:
==16454==     in use at exit: 0 bytes in 0 blocks
==16454==   total heap usage: 3 allocs, 3 frees, 1,144 bytes allocated
==16454== 
==16454== All heap blocks were freed -- no leaks are possible
==16454== 
==16454== For lists of detected and suppressed errors, rerun with: -s
==16454== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```
Al paracer no hubo fugas de memoria pues hubo 3 allocaciones y 3 liberaciones. Y ademas el mensaje `All heap blocks were freed` significa que toda la memoria dinamica que fue reservada fue liberada por el programa. Asi no hay problemas de fuga de memoria. 

## ¿Por que se usa sizeof(int) en lugar del valor literal 4? ¿Que ventaja ofrece en portabilidad entre arquitecturas?

Pues aunque en la mayoria de los computadores el valor del tamaño del `int` es $4$, pero no es garantia que en todos los casos se igual. Pues en segun la arquitectura, compilador o sistema operativo esto puede cambiar, algunos pueden pasar de $2 bytes$ o en algunos casos estraños $8 \text{ bytes}$. Usar la funcion `sizeof()` garantia la portabilidad entre arquitecturas y la gran ventaja esta en poder ejecutar el mismo codigo en diferentes plataformas sin necesidad de que haya una version diferente dependiendo de la plataforma.

### ¿Qu´e devuelve malloc Cu´ando no hay memoria disponible? ¿Por qu´e es critico verificar ese valor antes de usarlo?

Cuando no hay espacio en el heap disponible y se hace un llamado a la funcion `malloc()` esta devolvera un puntero con valor `null`. En caso de que este no sea verificado, el programa cuando intente acceder a la informacion alojada en este puntero se disparará un fallo de segmentacion -`segmentation fault`-, lo que ocacionara que el programa pare innesperadamente. 


Ejecuion del codigo `buggy_mem.c`

```
brayan@DESKTOP-RKQPITI:/mnt/c/Users/brayan/Documents/udea/2026-1/sistemas-operativos/so-lab3/punto-2$ valgrind --leak-check=full --track-origins=yes ./buggy_mem
==3375== Memcheck, a memory error detector
==3375== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==3375== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==3375== Command: ./buggy_mem
==3375== 
==3375== Invalid write of size 4
==3375==    at 0x1091E3: main (in /mnt/c/Users/brayan/Documents/udea/2026-1/sistemas-operativos/so-lab3/punto-2/buggy_mem)
==3375==  Address 0x4a75054 is 0 bytes after a block of size 20 alloc'd
==3375==    at 0x4846828: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==3375==    by 0x1091BE: main (in /mnt/c/Users/brayan/Documents/udea/2026-1/sistemas-operativos/so-lab3/punto-2/buggy_mem)
==3375== 
hola mundo
==3375== Invalid read of size 4
==3375==    at 0x109231: main (in /mnt/c/Users/brayan/Documents/udea/2026-1/sistemas-operativos/so-lab3/punto-2/buggy_mem)
==3375==  Address 0x4a75040 is 0 bytes inside a block of size 20 free'd
==3375==    at 0x484988F: free (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==3375==    by 0x10922C: main (in /mnt/c/Users/brayan/Documents/udea/2026-1/sistemas-operativos/so-lab3/punto-2/buggy_mem)
==3375==  Block was alloc'd at
==3375==    at 0x4846828: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==3375==    by 0x1091BE: main (in /mnt/c/Users/brayan/Documents/udea/2026-1/sistemas-operativos/so-lab3/punto-2/buggy_mem)
==3375== 
p[0] = 0
==3375== 
==3375== HEAP SUMMARY:
==3375==     in use at exit: 100 bytes in 1 blocks
==3375==   total heap usage: 3 allocs, 2 frees, 1,144 bytes allocated
==3375== 
==3375== 100 bytes in 1 blocks are definitely lost in loss record 1 of 1
==3375==    at 0x4846828: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==3375==    by 0x1091F8: main (in /mnt/c/Users/brayan/Documents/udea/2026-1/sistemas-operativos/so-lab3/punto-2/buggy_mem)
==3375== 
==3375== LEAK SUMMARY:
==3375==    definitely lost: 100 bytes in 1 blocks
==3375==    indirectly lost: 0 bytes in 0 blocks
==3375==      possibly lost: 0 bytes in 0 blocks
==3375==    still reachable: 0 bytes in 0 blocks
==3375==         suppressed: 0 bytes in 0 blocks
==3375== 
==3375== For lists of detected and suppressed errors, rerun with: -s
==3375== ERROR SUMMARY: 3 errors from 3 contexts (suppressed: 0 from 0)
```

### Transcriba los mensajes que arroja Valgrind. ¿Cu´al mensaje corresponde a cada uno de los tres errores cl´asicos?

1. `buffer overflow`: Ese error ocurre cuando un programa intenta escribir mas del espaci reservado por el puntero. y el mesnaje que manda valgrind cuando esete error es detecatado es estse 
```
Invalid write of size 4
```

2. `memory leak`: Este error ocurre cuando el programa reserva memoria y la uitiliza pero despues no la libera para que el sistema puede vovler a utilizarla. Esto causa problemas de rendimiento. Acontinuacion el mensaje de error de valgrind

```
==3375== LEAK SUMMARY:
==3375==    definitely lost: 100 bytes in 1 blocks
==3375==    indirectly lost: 0 bytes in 0 blocks
==3375==      possibly lost: 0 bytes in 0 blocks
==3375==    still reachable: 0 bytes in 0 blocks
==3375==         suppressed: 0 bytes in 0 blocks
```

3. `use after free`: Este error ocurre cuando un programa continua utilizando un putero despues de que la direccion de memoria a la que apuntaba haya sido liberada. Y el mensaje que manda valgrind es el siguiente.

```
Invalid read of size 4
```

### Corrija el programa (buggy mem fixed.c) y verifique con Valgrind que no queda ning´un error ni fuga.

Codigo corregido
```{c}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main()
{

  int *p = malloc(5 * sizeof(int));
  for (int i = 0; i < 5; i++)
    p[i] = i;

  char *q = malloc(100);
  strcpy(q, "hola mundo");
  printf("%s\n", q);
  printf("p[0] = %d\n", p[0]);
  free(q);
  free(p);
  return 0;
}
``` 
Demostracion de que no hay errores con valgrind

![comprobacion con valgrind](/punto-2/images/valgrind.png)


### ¿Qu´e consecuencias puede tener un use-after-free en un programa real en t´erminos de seguridad y estabilidad del sistema?

Cuando ocurre un use affter free se crea un `dangling pointer` que es un puntero que contiene una direccion de memoria que ya ha sido liberada o esta fuera de alcance. Los atacantes pueden utilizar esta vulnerabilidad para: 
- **Ejecucion de codigo arbirtrario:** los atacantes reemplazan la data original con codigo malicioso, el cual va a ser ejecutado por la aplicacion cuando haga referencia al `dalgling pointer`
 - **Corrupcion de datos y fuga de datos:** los atacantes pueden leer informacion que luego sera alojada en la direccion de este puntero colgante.

## Punto 3: Traduccion de direcciones.


###  Compile y ejecute. Muestre la salida completa. ¿Que ocurre al acceder a VA=64 y VA=100 en el Proceso A? ¿Que haria el SO real ante esta excepcion?
![ejecucion del programa base bounds](/punto-3/images/execution.png)

El proceso A define el registro bunds como 64 asi que todas las direcciones virtuales se mapearan desde la 0 hasta la 63. Y las direcciones 64 y 100 estan fuera del rango. El sistema operativo en este caso debera enviar un `segmentation fault` pues el proceso esta intentando acceder a una direccion de memoria que no le pertenece.

### Agregue un Proceso C (base=0, bounds=32) al programa y traduzca las mismas VAs. ¿Puede el Proceso A acceder a las direcciones del Proceso C directamente? Justifique.

Codigo agregando el proceso C
```{c}
// base_bounds.c
#include <stdio.h>

typedef struct
{
  int base;
  int bounds;
} Registro;

/* Traduce VA -> PA; imprime excepcion si viola bounds */
int traducir(Registro r, int va)
{
  if (va < 0 || va >= r.bounds)
  {
    printf(" [EXCEPCION] VA=%d viola bounds=%d\n",
           va, r.bounds);
    return -1;
  }
  return r.base + va;
}

int main()
{
  Registro procC = {0, 32};
  Registro procA = {32, 64};  /* base=32, bounds=64 */
  Registro procB = {128, 80}; /* base=128, bounds=80 */
  int vas[] = {0, 10, 63, 64, 100};
  int n = sizeof(vas) / sizeof(vas[0]);

  printf("--- Proceso A (base=%d, bounds=%d) ---\n",
         procA.base, procA.bounds);
  for (int i = 0; i < n; i++)
  {
    int pa = traducir(procA, vas[i]);
    if (pa != -1)
      printf(" VA=%3d -> PA=%3d\n", vas[i], pa);
  }
  
  printf("--- Proceso B (base=%d, bounds=%d) ---\n",
         procB.base, procB.bounds);
  for (int i = 0; i < n; i++)
  {
    int pa = traducir(procB, vas[i]);
    if (pa != -1)
      printf(" VA=%3d -> PA=%3d\n", vas[i], pa);
  }

  printf("--- Proceso C (base=%d, bounds=%d) ---\n",
         procC.base, procC.bounds);
  for (int i = 0; i < n; i++)
  {
    int pa = traducir(procC, vas[i]);
    if (pa != -1)
      printf(" VA%3d -> PA=%d\n", vas[i], pa);
  }
  return 0;
}
```

Ejecucion del script con el añadiendo el proceso C 
![ejecucion con el proceso c](/punto-3/images/process_c_execution.png)

En la imagen se ve que las direcciones tal que 

$$ \text{direction} \geq 32 $$

mandarán una exepcion pues estas sobrepasan el limite del registro `bounds`.

Ahora ¿Puede el proceso A acceder a las direcciones de C directamente? La respuesta corta es no. Segun el base and bounds, las direecciones se mapean a la memoria fisica tal que todas las direcciones son contiguas.

- **Proceso A**: $base=32, bounds=64$, si mapeamos a la memoria fisica, la memoria del proceso A iniciaria en el registro 32 y se cuentan 64 incluyendo el 32 hacia adelante. asi que el rango que este proceso tendria seria: $[32, 95]$

- **Proceso C**: $base=0, bounds=32$, haciendo lo mismo que con el proceso A, el rango de direcciones del proceso C en la memoria fisica es: $[0, 31]$

Como se ve en los rangos no se translapan ni se crusan, por esta razon el proces A es incapas de acceder a los datos que estan en el proceso C.

### ¿Cuál es la limitacion principal del esquema base & bounds que motiva el surgimiento de la segmentacion?

El prinicipal problema del base and bounds, es la **fragmentacion interna** que es la perdida de memoria en por la parte no usada de memoria cuando se asignan particiones con tamaños fijos, para todo el bloque de memoria que usará un proceso
