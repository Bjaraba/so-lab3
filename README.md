# Laboratorio 3 - Sistemas Operativos

Integrantes
Kelly Julieth Arango Henao

kjulieth.arangoh@udea.edu.co

1036657098

Brayan Stiven Jaraba Alvarez

b.jaraba@udea.edu.co

1032178608

## Punto 1: Espacio de direcciones

### Ejecución del programa

![Ejecución del programa](/punto-1/images/program-execution.png)

### Mapa de memoria del proceso

![Mapa de memoria](/punto-1/images/data-map.png)

### Resumen de las regiones de datos

![Resumen](/punto-1/images/summary.png)

## Preguntas

### Identifique en la salida de `/proc/maps` las regiones `text`, `heap` y `stack`. ¿Qué permisos (`r/w/x/p`) tiene cada una? ¿Por qué difieren?

1. **Stack**

```text
7ffcb72f7000-7ffcb7319000 rw-p 00000000 00:00 0                          [stack]
```

Se observa que tiene permisos de lectura y escritura (`rw`), ya que esta región almacena datos que pueden ser modificados durante la ejecución del programa, como variables locales y llamadas a funciones.

2. **Heap**

```text
5c5d16eca000-5c5d16eeb000 rw-p 00000000 00:00 0                          [heap]
```

El `heap` también posee permisos de lectura y escritura (`rw`) porque en esta región se almacena memoria dinámica que puede ser reservada y modificada en tiempo de ejecución mediante funciones como `malloc()`.

3. **Text**

```text
00005c5ce3a5f000       4       4       0 r-x-- mem_map
```

La región `text` tiene permisos de lectura y ejecución (`r-x`). Esto se debe a que allí se encuentra el código ejecutable del programa. No posee permisos de escritura por motivos de seguridad y estabilidad.

---

### Compare las direcciones impresas con los rangos de `/proc/maps`. ¿A qué región pertenece cada variable?

Direcciones de las variables:

```text
Dir. global_var : 0x5c5ce3a62010
Dir. local_var : 0x7ffcb7315c0c
Dir. heap_var : 0x5c5d16eca2a0
```

* `global_var: 0x5c5ce3a62010`

  Esta variable se encuentra dentro del rango:

  ```text
  00005c5ce3a62000       4       4       4 rw--- mem_map
  ```

  Corresponde probablemente a la sección `.data`, donde se almacenan las variables globales.

* `local_var : 0x7ffcb7315c0c`

  Esta variable está dentro del rango:

  ```text
  7ffcb72f7000-7ffcb7319000 rw-p 00000000 00:00 0                          [stack]
  ```

  Por lo tanto, pertenece al `stack`, lo cual coincide con el comportamiento esperado para las variables locales.

* `heap_var : 0x5c5d16eca2a0`

  Esta variable pertenece al rango:

  ```text
  5c5d16eca000-5c5d16eeb000 rw-p 00000000 00:00 0                          [heap]
  ```

  Por lo tanto, corresponde a memoria dinámica alojada en el `heap`.

---

### ¿Qué otras regiones aparecen en el mapa (`libc`, `[vdso]`, `[vsyscall]`)? ¿Qué función cumple cada una?

Dentro del mapa también aparecen las siguientes regiones:

* `libc.so.6`

  Es la biblioteca estándar de C en Linux. Contiene funciones fundamentales como `printf`, `scanf`, `malloc`, `free`, `strlen`, `open`, `read` y `write`. Generalmente posee permisos de lectura y ejecución.

* `ld-linux-x86-64.so.2`

  Es el cargador dinámico del sistema. Se encarga de cargar las bibliotecas compartidas antes de ejecutar `main()`, resolver símbolos y preparar el proceso.

* `vvar`

  Es una región utilizada por el kernel para compartir información del sistema de forma eficiente, como datos de tiempo y estadísticas.

* `vdso`

  Significa *Virtual Dynamic Shared Object*. Permite ejecutar ciertas llamadas al sistema sin cambiar al modo kernel, como `gettimeofday()` o `clock_gettime()`, mejorando el rendimiento.

---

### ¿Son las direcciones virtuales iguales a las físicas? Explique apoyándose en el concepto de *address space* del OSTEP.

No. Según OSTEP, la memoria virtual es una abstracción de la memoria física. Cada proceso posee su propio *address space* o espacio de direcciones virtuales, aislado de los demás procesos.

Las direcciones virtuales deben ser traducidas a direcciones físicas mediante la MMU y las tablas de páginas. Gracias a esto, cada proceso tiene la impresión de poseer toda la memoria para sí mismo.

---

### Ejecución de dos instancias del programa al mismo tiempo

![Ejecución de dos instancias](/punto-1/images/two-instaces.png)

### ¿Son las mismas direcciones virtuales en ambos procesos? ¿Qué conclusión saca sobre el aislamiento del espacio de direcciones?

En este caso, las direcciones virtuales son diferentes y no se traslapan. Esto demuestra que el sistema operativo mantiene aislado el espacio de direcciones de cada proceso, garantizando seguridad y evitando que un proceso interfiera con otro.

---

### ¿Podría el Proceso A leer o modificar la variable global del Proceso B mediante su dirección virtual? Justifique.

No. Las direcciones virtuales se traducen de manera independiente para cada proceso. Aunque dos procesos puedan utilizar direcciones virtuales similares, estas apuntarán a ubicaciones físicas diferentes.

Por esta razón, un proceso no puede acceder directamente a la memoria privada de otro proceso.

---

# Punto 2: API de memoria

![Ejecución de los comandos](/punto-2/images/execution.png)

### Muestre la salida completa de Valgrind. ¿Reporta errores o fugas de memoria? ¿Qué significa el mensaje "All heap blocks were freed"?

```text
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

No se reportan fugas de memoria ni errores. Hubo 3 asignaciones y 3 liberaciones de memoria.

El mensaje:

```text
All heap blocks were freed -- no leaks are possible
```

indica que toda la memoria dinámica reservada fue correctamente liberada.

---

### ¿Por qué se usa `sizeof(int)` en lugar del valor literal `4`? ¿Qué ventaja ofrece en portabilidad entre arquitecturas?

Aunque en muchas arquitecturas un `int` ocupa 4 bytes, esto no está garantizado en todos los sistemas.

El uso de `sizeof(int)` permite que el programa funcione correctamente independientemente de la arquitectura, compilador o sistema operativo.

Esto mejora la portabilidad y evita errores relacionados con tamaños de datos distintos.

---

### ¿Qué devuelve `malloc()` cuando no hay memoria disponible? ¿Por qué es crítico verificar ese valor antes de usarlo?

Cuando no hay memoria disponible, `malloc()` devuelve `NULL`.

Si el programa no verifica este valor e intenta acceder a la memoria apuntada por el puntero, se producirá un fallo de segmentación (*segmentation fault*).

Por ello, siempre es importante validar el resultado de `malloc()` antes de utilizar el puntero.

---

## Ejecución del código `buggy_mem.c`

```text
brayan@DESKTOP-RKQPITI:/mnt/c/Users/brayan/Documents/udea/2026-1/sistemas-operativos/so-lab3/punto-2$ valgrind --leak-check=full --track-origins=yes ./buggy_mem
==3375== Memcheck, a memory error detector
...
```

---

### Transcriba los mensajes que arroja Valgrind. ¿Cuál mensaje corresponde a cada uno de los tres errores clásicos?

1. **Buffer overflow**

Ocurre cuando un programa escribe más allá del espacio reservado.

Mensaje:

```text
Invalid write of size 4
```

2. **Memory leak**

Ocurre cuando el programa reserva memoria y nunca la libera.

Mensaje:

```text
LEAK SUMMARY:
definitely lost: 100 bytes in 1 blocks
```

3. **Use after free**

Ocurre cuando un programa utiliza memoria después de haber sido liberada.

Mensaje:

```text
Invalid read of size 4
```

---

### Corrija el programa (`buggy_mem_fixed.c`) y verifique con Valgrind que no queda ningún error ni fuga.

## Código corregido

```c
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

### Comprobación con Valgrind

![Comprobación con Valgrind](/punto-2/images/valgrind.png)

---

### ¿Qué consecuencias puede tener un *use-after-free* en un programa real en términos de seguridad y estabilidad?

Un *use-after-free* genera un *dangling pointer*, es decir, un puntero que apunta a memoria ya liberada.

Esto puede provocar:

* Ejecución de código arbitrario.
* Corrupción de datos.
* Fugas de información.
* Caídas inesperadas del programa.
* Vulnerabilidades de seguridad explotables por atacantes.

---

# Punto 3: Traducción de direcciones

### Compile y ejecute. Muestre la salida completa. ¿Qué ocurre al acceder a `VA=64` y `VA=100` en el Proceso A? ¿Qué haría el SO real ante esta excepción?

![Ejecución del programa base bounds](/punto-3/images/execution.png)

El proceso A define el registro `bounds` como 64, por lo que únicamente son válidas las direcciones virtuales entre `0` y `63`.

Las direcciones `64` y `100` están fuera del rango permitido. En un sistema operativo real, esto produciría un `segmentation fault`.

---

### Agregue un Proceso C (`base=0`, `bounds=32`) al programa y traduzca las mismas VAs. ¿Puede el Proceso A acceder a las direcciones del Proceso C directamente? Justifique.

```c
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

![Ejecución con el proceso C](/punto-3/images/process_c_execution.png)

Las direcciones mayores o iguales a `32` generan una excepción porque exceden el valor de `bounds`.

El Proceso A no puede acceder directamente a las direcciones del Proceso C, ya que ambos poseen espacios de direcciones aislados.

* **Proceso A:** `base=32`, `bounds=64` → rango físico `[32, 95]`
* **Proceso C:** `base=0`, `bounds=32` → rango físico `[0, 31]`

Los rangos no se traslapan, por lo que el aislamiento de memoria se mantiene.

---

### ¿Cuál es la limitación principal del esquema *base & bounds* que motiva el surgimiento de la segmentación?

La principal limitación es la fragmentación interna.

El sistema reserva bloques contiguos de memoria para cada proceso, lo que puede desperdiciar espacio cuando el proceso no utiliza completamente la memoria asignada.

---
# Punto 4:  Segmentacion

**1. Muestre el cálculo paso a paso para cada VA.**

**VA = 0x03A0** 
-----
Selector: 00  Code

Offset: 0x3A0=(3×256)+(10×16)+0=928

Tamaño Code: 2KB=2×1024=2048

928<2048   válido

Calcular PA:

PA=base+offset
PA=0x4000+0x3A0=0x43A0
PA 0x43A0


**VA = 0x1800**
-----
Selector: 01 Heap

Offset: 0x800=(8×256)+0+0=2048

Tamaño Heap: 3KB=3×1024=3072

2048<3072 válido

Calcular PA:

PA=base+offset
PA=0x6000+0x800=0x6800
PA 0x6800

**VA = 0x3C00**
-----
Selector:11 Stack 

Offset:0xC00=3072  

Tamaño Stack: 2KB= 2x1024=2048

3072>2048  no válido

EXCEPCIÓN (segmentation fault)

**VA = 0x0C00**
-----
Selector: 00 Code

Offset: 0xC00=(12×256)+0+0=3072 

Tamaño Code: 2KB= 2x1024=2048

3072>2048  no válido

EXCEPCIÓN 

**VA = 0x2200** 
-----
Selector: 10 

Offset: 0x200=512 

Segmento no definido

EXCEPCIÓN 

**2. ¿Por qué el Stack crece en dirección negativa? ¿Qué ajuste especial requiere la fórmula al calcular el PA?**

El Stack crece en dirección negativa porque normalmente inicia en direcciones altas de memoria y va descendiendo hacia direcciones menores a medida que se agregan variables locales, parámetros y llamadas a funciones. Esto permite que el Stack y el Heap crezcan en direcciones opuestas y aprovechen mejor el espacio de memoria disponible.

En los segmentos que crecen positivamente (como Code y Heap), la dirección física se calcula así:

PA=base+offset

Sin embargo, como el Stack crece hacia abajo, el cálculo cambia a:

PA=base−offset

Es decir, el offset se resta en lugar de sumarse.

**3. ¿Qué ventaja tiene la segmentación frente a base & bounds en cuanto a utilización de la memoria física?**

La principal ventaja de la segmentación es que divide el espacio de direcciones en varios segmentos lógicos independientes, como código, heap y stack, cada uno con su propio tamaño y permisos.
En base & bounds todo el proceso debe ocupar una única región contigua de memoria, lo que puede desperdiciar espacio y dificultar el crecimiento dinámico.

Con segmentación:

- Cada segmento puede ubicarse en diferentes partes de la memoria física.
- Se aprovecha mejor el espacio disponible.
- Es posible proteger segmentos individualmente.
- Se facilita el crecimiento independiente del heap y del stack.

Por esta razón, la segmentación ofrece una utilización más flexible y eficiente de la memoria física.

**4. Qué es la fragmentación externa? ¿Por qué surge con segmentación? Ilustre con un diagrama de bloques de memoria.**

La fragmentación externa ocurre cuando existe suficiente memoria libre total, pero está dividida en pequeños bloques no contiguos. Debido a esto, una solicitud grande de memoria puede fallar aunque la suma de espacios libres sea suficiente.
En segmentación, cada segmento tiene tamaño variable y puede crecer o liberarse en diferentes momentos. Esto provoca que queden huecos dispersos en la memoria física.

Ejemplo:

 Libre 100 | Ocupado   | Libre 80  | Ocupado   | Libre 90  |

En este caso:

100+80+90=270 bytes libres

Pero una solicitud de 200 bytes continuos podría fallar porque no existe un bloque contiguo suficientemente grande.

La fragmentación externa es uno de los principales problemas de la segmentación y motivó el uso de paginación en sistemas modernos.

# Punto 5:  Paginación

**5.1 Actividad: Calculo de la tabla de paginas**

**1. ¿Cuántos bits se necesitan para el VPN y cuántos para el offset?**

El tamaño de página es:4KB=4096=2^12

Por lo tanto:

- Se necesitan 12 bits para el offset.
- El espacio virtual es de 32 bits.

Entonces:

VPN=32−12=20extbits

Resultado
Campo	Bits
VPN	20 bits
Offset	12 bits

**2. ¿Cuántas entradas tiene la tabla de páginas de un proceso?**

Cada página virtual necesita una entrada en la tabla.

Como el VPN tiene 20 bits:

2^20 =1,048,576


Resultado

La tabla de páginas tiene: 1,048,576 entradas


**3. ¿Cuanta memoria ocupa la tabla de paginas completa? ¿Es razonable ese tamaño para cada proceso?**

Cada entrada de la tabla de páginas (PTE) ocupa: 4 bytes y la tabla tiene: 1,048,576 entradas

Entonces el tamaño total es:

1,048,576×4=4,194,304 bytes

4,194,304 bytes ≈ 4 MB

Resultado

La tabla de páginas completa ocupa aproximadamente: 4 MB

El tamaño no es razonable completamenta, ya que tener una tabla de páginas de 4 MB para cada proceso consumiría demasiada memoria en sistemas con muchos procesos ejecutándose al mismo tiempo.

**4. ¿Cuántos bits necesita el PFN dentro de la PTE? ¿Qué información almacenan los bits restantes? Mencione al menos 3 bits de control y su función.**

El espacio físico es de 20 bits = 1MB=2^20

El tamaño de página es: 4KB = 2^12

Entonces, los bits necesarios para el offset son 12 bits. Los bits restantes corresponden al PFN (Physical Frame Number): PFN=20−12=8 bits

Resultado

El PFN necesita: 8 bits porque la memoria física tiene: 2^8=256 marcos físicos posibles.

Los demás bits de la entrada de tabla de páginas (PTE) se utilizan como bits de control y protección.

- Present/Valid: Indica si la página está cargada en memoria RAM
- Dirty: Indica si la página fue modificada desde que se cargó
- -Referenced/Accessed: Indica si la página fue utilizada recientemente
- 
**5.2 Actividad: Simulador de paginación**

[Ver Programa simulador de paginacion](https://github.com/Bjaraba/so-lab3/blob/main/punto-5/paging%20sim.c)

**5.3 Actividad: Simulador — Analisis**

**1. Compile y ejecute el simulador. Muestre la salida completa.**

<img width="571" height="188" alt="image" src="https://github.com/user-attachments/assets/c05898d8-c3a2-4919-805c-6dfaefabe386" />

**2. ¿Qué ocurre con las VAs 0x10 y 0xA3? ¿Qué debería hacer el SO real ante un page fault?**

La dirección virtual 0x10 produce un page fault porque la página virtual a la que pertenece no se encuentra cargada en memoria RAM. Al calcular el VPN (Virtual Page Number), se obtiene el valor 1, y en la tabla de páginas la posición 1 contiene -1, lo que indica que la página no está presente en memoria física.

En cambio, la dirección virtual 0xA3 no produce un page fault. Su VPN corresponde a la página virtual 10 y, según la tabla de páginas, esta sí está asociada a un marco físico válido (PFN = 4). Por ello, la dirección puede traducirse correctamente a una dirección física.

Cuando ocurre un page fault, el sistema operativo debe interrumpir temporalmente la ejecución del proceso y verificar si la dirección solicitada es válida. Si la página existe pero no está cargada en RAM, el SO la busca en disco, la carga en un marco libre de memoria física, actualiza la tabla de páginas y luego continúa la ejecución del programa. Si la dirección es inválida o el proceso no tiene permisos de acceso, el sistema operativo termina el proceso generando un segmentation fault.

**3. ¿Cuántos accesos a memoria física requiere completar una instrucción load con tabla de paginas de un solo nivel? ¿Por qué es costoso y que solución de hardware existe?**

Con una tabla de páginas de un solo nivel, una instrucción load necesita dos accesos a memoria física. El primero se hace para buscar en la tabla de páginas la traducción de la dirección virtual y obtener el PFN correspondiente. El segundo acceso se realiza para leer el dato real desde la memoria física.

Esto es costoso porque por cada acceso a memoria que hace el programa, el procesador debe realizar dos lecturas en RAM, lo que aumenta el tiempo de ejecución y hace más lento el sistema.

Para mejorar el rendimiento existe una solución de hardware llamada TLB (Translation Lookaside Buffer). El TLB funciona como una memoria caché que guarda traducciones recientes de direcciones virtuales a físicas. Si la traducción ya está en el TLB, no es necesario consultar la tabla de páginas nuevamente, reduciendo el tiempo de acceso a memoria..

**4. ¿Qué ventaja tiene la paginación sobre la segmentación en cuanto al fenomeno de fragmentación?**

La principal ventaja de la paginación sobre la segmentación es que evita la fragmentación externa. En la paginación, tanto las páginas virtuales como los marcos de memoria física tienen un tamaño fijo, por lo que cualquier página puede almacenarse en cualquier frame libre de la memoria RAM.

En cambio, en la segmentación los segmentos tienen tamaños variables, lo que provoca que con el tiempo queden espacios libres pequeños y dispersos en memoria. Aunque exista suficiente memoria total disponible, puede no haber un bloque contiguo lo suficientemente grande para almacenar un nuevo segmento, generando fragmentación externa.

La paginación reduce este problema porque trabaja con bloques de tamaño fijo. Sin embargo, puede producir fragmentación interna, ya que una página puede no utilizar completamente el espacio del frame asignado.

# Punto 6:  Gestión de espacio libre

**6.1 Actividad: Simulación de estrategias de asignación**

First Fit selecciona el primer bloque libre que sea suficientemente grande.

Solicitud 1: malloc(212)
-----
100 → no cabe
500 → sí cabe

Se asigna el bloque de 500 bytes.

Espacio restante: 500−212=288

Lista libre:

Dirección	Tamaño
0x0100	   100
0x02D4	   288
0x0400	   200
0x0500	   300
0x0700	   600

Solicitud 2: malloc(417)
-----
100 → no cabe
288 → no cabe
200 → no cabe
300 → no cabe
600 → sí cabe

Se asigna el bloque de 600 bytes.

Espacio restante: 600−417=183

Lista libre:

Dirección	Tamaño
0x0100	   100
0x02D4     288
0x0400	   200
0x0500	   300
0x08A1	   183


Solicitud 3: malloc(98)
-----

100 → sí cabe

Espacio restante: 100−98=2

Lista libre:

Dirección 	Tamaño
0x0162      	2
0x02D4      	288
0x0400      	200
0x0500	      300
0x08A1	      183

Solicitud 4: malloc(426)
-----

2 → no cabe
288 → no cabe
200 → no cabe
300 → no cabe
183 → no cabe

Resultado:

FALLO DE ASIGNACIÓN
Lista libre final con First Fit

Dirección	Tamaño
0x0162	   2
0x02D4	   288
0x0400	   200
0x0500	   300
0x08A1	   183



**2. Repita con best fit. ¿Cambia el resultado?**

Best Fit selecciona el bloque más pequeño posible que pueda satisfacer la solicitud.

malloc(212)
-----
Bloques posibles:

500
300
600

El más ajustado es: 300 bytes

Resto: 300−212=88


malloc(417)
-----
Bloques posibles:

500
600

El mejor ajuste es: 500 bytes

Resto: 500−417=83


malloc(98)
-----
Bloques posibles:

100
200
600

El mejor ajuste es: 100 bytes

Resto: 100−98=2

malloc(426)
-----
Solo queda disponible: 600 bytes

Resto: 600−426=174


Resultado con Best Fit

Todas las solicitudes pueden asignarse correctamente.

Lista libre final:

Dirección	      Tamaño
bloque restante	2
bloque restante	83
bloque restante	200
bloque restante	88
bloque restante	174

Si cambia el resultado, ya que con First Fit la última asignación falla porque el bloque grande de 600 bytes ya había sido fragmentado antes. Con Best Fit las solicitudes se distribuyen mejor y todas pueden satisfacerse.

**3. ¿Cuál estrategia genera mas fragmentación externa en este caso? ¿Cuál la minimiza?**

En este caso, la estrategia que genera más fragmentación externa es First Fit, porque asigna el primer bloque disponible que encuentre, sin buscar el más adecuado. Esto hace que queden espacios libres pequeños y dispersos que después son difíciles de reutilizar.

La estrategia que minimiza la fragmentación externa es Best Fit, ya que busca el bloque más pequeño posible que pueda satisfacer la solicitud. De esta forma se aprovecha mejor el espacio disponible y se evita desperdiciar bloques grandes innecesariamente.

**4. ¿Qué es el coalescing? Ilustre un caso donde su ausencia provoca que una solicitud de 250 bytes falle aunque haya suficiente memoria total libre.**

El coalescing es el proceso de unir bloques libres contiguos para formar un bloque más grande. Esto ayuda a reducir la fragmentación externa y mejorar el aprovechamiento de la memoria.

Por ejemplo, si existen tres bloques libres consecutivos de:

100 bytes
100 bytes
100 bytes

la memoria libre total sería de 300 bytes. Sin embargo, si no se realiza coalescing, una solicitud de malloc(250) fallaría porque no existe un único bloque continuo de 250 bytes, aunque la suma total de memoria libre sí sea suficiente.

Si se aplica coalescing, los tres bloques se unen formando un bloque de 300 bytes y la solicitud podría realizarse correctamente.

**5. ¿Qué es la fragmentación interna? ¿Cuándo aparece tipicamente al usar un slab allocator?

La fragmentación interna ocurre cuando se asigna un bloque de memoria más grande de lo que realmente necesita el programa, dejando espacio desperdiciado dentro del bloque asignado.

Esto sucede típicamente en un slab allocator, porque este sistema trabaja con bloques de tamaños fijos. Por ejemplo, si un proceso necesita 50 bytes y el slab allocator solo dispone de bloques de 64 bytes, se asigna el bloque completo y quedan 14 bytes sin utilizar dentro de ese espacio. Aunque el programa no use esos bytes, tampoco pueden ser aprovechados por otros procesos.


**6.2 Actividad: Fragmentación**

[Ver programa fragmentacion](https://github.com/Bjaraba/so-lab3/blob/main/punto-6/fragmentation.c)


<img width="463" height="151" alt="image" src="https://github.com/user-attachments/assets/108a78f8-07d2-4e30-9313-f70c872d9c46" />

**1.¿Son consecutivas en memoria las direcciones asignadas? ¿Qué patrón de separación observa entre bloques contiguos?**

Las direcciones asignadas por malloc() generalmente aparecen cercanas entre sí, pero no completamente consecutivas. Entre bloques contiguos se observa una pequeña separación debido a la información de control que el allocator de glibc guarda internamente para administrar cada bloque de memoria, como tamaño, estado y enlaces de la lista libre.

Además, el allocator puede aplicar alineación de memoria para mejorar el rendimiento del procesador, por lo que las direcciones no aumentan exactamente según el tamaño solicitado.

**2. ¿Tiene éxito la asignación final de 1500 bytes? Explique el resultado en términos de fragmentación?**

Sí tiene éxito,aunque durante la ejecución se liberan varios bloques y se generan huecos en memoria, el allocator de glibc puede solicitar más memoria al sistema operativo si no encuentra un bloque continuo suficientemente grande dentro del heap actual. Por esta razón, la asignación logra completarse.

Este comportamiento muestra que la fragmentación externa puede dificultar reutilizar los espacios libres existentes, pero el allocator puede ampliar el heap para satisfacer solicitudes grandes.

**3. Consulta: ¿Cuál es la diferencia entre el allocator de usuario (malloc/glibc) y el del kernel(buddy system, slab)? ¿Por qué existen dos niveles de gestión de memoria?**

El allocator de usuario, como malloc() de glibc, administra la memoria dinámica utilizada por los programas en espacio de usuario. Su función es entregar bloques de memoria a las aplicaciones y reutilizar los espacios liberados.

Por otro lado, el allocator del kernel administra la memoria interna del sistema operativo. El buddy system se utiliza para asignar bloques de páginas físicas y el slab allocator optimiza la creación de objetos frecuentes del kernel, como estructuras de procesos o buffers.

Existen dos niveles de gestión porque el sistema operativo y las aplicaciones tienen necesidades diferentes. El kernel necesita un control más eficiente y seguro de la memoria física, mientras que las aplicaciones requieren una interfaz más sencilla y flexible para manejar memoria dinámica.

# Punto 7: TLB (*Translation Lookaside Buffer*)

## Ejecución del código durante 3 veces

![Ejecución del código 3 veces](/punto-7/images/3-execution.png)

### ¿Cuántas veces más lento es el acceso aleatorio frente al secuencial? Muestre el promedio de 3 ejecuciones.

* Acceso secuencial: `[13.53, 18.73, 19.38] ms`
* Acceso aleatorio: `[41.01, 57.59, 42.92] ms`

Promedio secuencial:

```math
\frac{51.64}{3} \approx 17.213
```

Promedio aleatorio:

```math
\frac{141.52}{3} \approx 47.173
```

Porcentaje de diferencia:

```math
\frac{47.173 - 17.213}{17.213} \times 100\% \approx 174\%
```

El acceso aleatorio es aproximadamente un `174 %` más lento que el acceso secuencial.

---

### Explique con el modelo del TLB por qué el acceso aleatorio es más lento. ¿Qué ocurre con el *hit rate* en cada caso?

En el acceso secuencial, los datos suelen encontrarse en páginas cercanas entre sí. Esto favorece la localidad espacial y aumenta la probabilidad de que las traducciones ya estén almacenadas en el TLB.

Por esta razón, el *hit rate* es alto.

En cambio, en el acceso aleatorio las referencias a memoria están dispersas, lo que incrementa los fallos de TLB (*TLB misses*). Como consecuencia, el procesador debe consultar con mayor frecuencia las tablas de páginas en memoria principal, aumentando el tiempo de acceso.

---

### Si el tamaño de página fuera 64 KB en lugar de 4 KB, ¿mejoraría o empeoraría la situación con accesos aleatorios?

Desde el punto de vista del TLB, un tamaño de página mayor podría mejorar ligeramente el *hit rate*, ya que cada entrada cubriría una región más amplia de memoria.

Sin embargo, también aumentaría el desperdicio de memoria por fragmentación interna.

Además, si los accesos aleatorios siguen siendo muy dispersos, la mejora en el rendimiento podría no ser significativa.

---

### Un TLB con 64 entradas (*fully associative*) y páginas de 4 KB. ¿Cuánta memoria puede cubrir sin generar *misses*? ¿Es suficiente para un proceso moderno típico?

La cobertura total sería:

```math
64 \times 4\text{ KB} = 256\text{ KB}
```

Por lo tanto, el TLB puede cubrir hasta `256 KB` de memoria sin generar fallos.

Para programas modernos, esta cantidad suele ser insuficiente, ya que las aplicaciones actuales utilizan varios megabytes o incluso gigabytes de memoria.

---

### ¿Qué es un *TLB shootdown* y en qué situación ocurre en sistemas multiprocesador? ¿Por qué es una operación costosa?

Un *TLB shootdown* es un mecanismo mediante el cual un procesador obliga a otros procesadores a invalidar entradas de sus TLB.

Esto ocurre cuando el sistema operativo modifica las tablas de páginas y necesita garantizar consistencia entre todos los núcleos.

Es costoso porque:

* Requiere interrupciones entre procesadores.
* Obliga a sincronizar múltiples núcleos.
* Los núcleos deben detener temporalmente su ejecución.
* Se pierden entradas del TLB, aumentando futuros accesos a memoria.

---

### Explique la diferencia entre un TLB gestionado por hardware (CISC/x86) y uno gestionado por software (RISC/MIPS). ¿Cuál ofrece mayor flexibilidad al diseñador del SO y por qué?

| Característica           | TLB gestionado por hardware | TLB gestionado por software |
| ------------------------ | --------------------------- | --------------------------- |
| Manejo de TLB miss       | Hardware                    | Sistema operativo           |
| Intervención del SO      | Mínima                      | Alta                        |
| Atención de fallos       | Automática                  | Mediante excepción          |
| Rendimiento              | Mayor                       | Menor                       |
| Complejidad del hardware | Alta                        | Baja                        |
| Flexibilidad             | Menor                       | Mayor                       |
| Cambio a modo kernel     | Generalmente no necesario   | Necesario                   |
| Ejemplos                 | x86, x86-64                 | MIPS                        |

El TLB gestionado por software ofrece mayor flexibilidad al diseñador del sistema operativo, ya que el kernel puede implementar distintas políticas de reemplazo y manejo de memoria.

Sin embargo, esta flexibilidad tiene el costo de un mayor tiempo de atención ante los *TLB misses*.
