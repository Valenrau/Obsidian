### Que puede hacer una computadora?
Esencialmente, ejecuta operaciones aritméticas y lógicas simples en una secuencia ordenada
![[Pasted image 20260720173353.png|378]]

## El problema del conexionado
### Conexionado tipo BUS
Reduce la cantidad de rutas necesarias para la comunicación de los distintos componentes de la computadora, Utiliza un solo canal, **el Bus**
La estructura [[Microarquitectura#Organización del Trayecto de Datos|Bus]] implementa físicamente la lógica de un **mapa de memoria**, donde cada posición se diferencia por un numero, **la Dirección**. Un bus es un sistema de comunicación para la transferencia de datos entre componentes electrónicos.
![[Pasted image 20260720174117.png|420]]

### Como ejecuta los programas el cpu?
1) Buscar en memoria la próxima instrucción a ser ejecutada
2) Decodificar el código de operación de esa instrucción
3) Ejecutar la instrucción
4) Volver a (1)
A esto se lo conoce como ciclo **Fetch**

### Big-Endian vs Little-Endian
Cuando se guardan palabras de más de un byte (ej. 32 bits) , el orden de los bytes en la memoria (que es direccionable por bytes ) depende del procesador:
- **Little-Endian:** El byte **menos** significativo se almacena en la dirección de memoria más baja. (Ej. Intel).
- **Big-Endian:** El byte **más** significativo se almacena en la dirección de memoria más baja. (Ej. Motorola)

## ARC
RISC: Reduced Instruction Set Computer
***ARC: A Risc Computer***
- Es una maquina de 32 bits con capacidad de direccionamiento por bytes
- Puede manejar tipos de datos de 32 bits, pero la información se almacena en memoria en forma de bytes
- Puede manejar tipos de datos de 32 bits, pero la información se almacena en memoria en forma de bytes
- las primeras 2048 direcciones están reservadas para el SO
- El espacio del usuario es el que se reserva para la carga de los programas ensambladores por el usuario
- Pila del sistema crece en direcciones inferiores de memoria
- Zona reservada para dispositivos de entrada-salida. Cada dispositivo tiene asignada una serie de direcciones de memoria para el almacenamiento de sus datos: entrada mapeada en memoria.
- La arquitectura ARC funciona con el byte más significativo almacenado en la menor de las direcciones asignada a la palabra (big endian).
### Registros
Posee 32 registros de 32 bits, nombrados de `%r0` a `%r31`.
- `%r0` siempre contiene el valor cero.
- `%r14` se usa como el Puntero de Pila (`%sp`).
- `%r15` se usa como registro de enlace para guardar la dirección de retorno en llamadas a subrutinas.
- También cuenta con un registro de estado (PSR) y un contador de programa (PC).

#### Los dos registros que están aparte (no son de propósito general)
##### PSR - Program Status Register
También 32 bits, pero no lo usas para guardar datos de tu programa: guarda los **flags** que resultan de las operaciones aritmético-lógicas (n = negativo, z = zero, v = overflow, c = carry). Son los que consultan las instrucciones de salto condicional que aparecían en el resumen: `be` (branch if equal), `bne`, `bcs`, `bneg`, `bvs`, etc. Por eso vas a ver `andcc`, `addcc` con el sufijo `cc` — ese sufijo indica "actualiza los flags del PSR", a diferencia de la versión sin `cc` que no los toca.

##### PC - Program Counter
Apunta a la próxima instrucción a buscar en memoria. Es el que se actualiza automáticamente en cada ciclo de fetch, y el que manipulan directamente las instrucciones de salto (`ba`, `be`, `call`, `jmpl`).
### Instrucciones  y Directivas de ARC
![[Pasted image 20260720181700.png]]

Son instrucciones que le dan al **ensamblador** (el programa que traduce tu código Assembler a código de máquina), no al procesador. Por eso se las llama también "pseudo-instrucciones": no generan necesariamente código ejecutable, sino que le dicen al ensamblador _cómo_ organizar o interpretar el programa.Ejemplos:

- **`.equ`**: Trata un símbolo como un valor constante.
- **`.begin`**: Indica al ensamblador que comience a ensamblar.
- **`.end`**: Indica al ensamblador que deje de ensamblar.
- **`.org`**: Cambia el contador de locación (la dirección de memoria donde se ensamblará la siguiente instrucción) al valor especificado.
- **`.dwb`**: Reserva un bloque de palabras en la memoria.
- **`.global`**: Indica que un símbolo se usa en otro módulo (es global).
- **`.extern`**: Indica que un símbolo está definido en otro módulo (es externo).
- **`.macro`**: Define el inicio de una macro con sus parámetros formales.
- **`.endmacro`**: Marca el final de la definición de una macro

## Subrutinas y Macros
### Subrutinas
Una Subrutina es un bloque de código **reutilizable** al que se accede en **tiempo de ejecución** mediante una instrucción **`call`**, y que termina devolviendo el control con **`jmpl`** 

Puntos claves:
- Se llega con `call` → esto guarda la dirección de retorno en `%r15`.
- Termina con `jmpl %r15+4, %r0` → vuelve a donde fue llamada.
- El código de la subrutina **existe una sola vez en memoria**, sin importar cuántas veces se la invoque.
- Los argumentos se pueden pasar de tres formas
	1) - **Por registros** (`%r1`, `%r2` → resultado en `%r3`)
	2) **Por stack** (usando `%r14` como puntero de pila, con `st`/`ld`)
	3) **Por área reservada en memoria** (se pasa un puntero, ej. `%r5`, a una zona con los operandos y el resultado)

### Macros
Una macro también es código reutilizable, pero la diferencia fundamental es **cuando** y **cómo** se resuelve: se expande en **tiempo de ensamblado**, no de ejecución. El ensamblador literalmente reemplaza cada invocación de la macro por su código equivalente, sustituyendo los parámetros.

Se define así:
```
.macro mcr_add Reg1, Reg2, Reg3
    addcc Reg1, Reg2, Reg3
.endmacro
```
Y se usa como si fuera una instrucción más:
```mcr_add %r1, %r2, %r3```
El ensamblador la reemplaza directamente por `addcc %r1, %r2, %r3` en ese punto del código — no hay `call` ni `jmpl` involucrados.

## RISC vs CISC

### CISC (Complex Instruction Set Computer)
La idea es tener muchas instrucciones, cada una haciendo mas cosas por si sola, aunque eso las haga mas complejas de decodificar, por ejemplo cpus x86 de los 90

### RISC (Reduced Instruction Set Computer)
Acá la filosofía es la opuesta: simplificar al máximo, aunque eso implique más instrucciones para hacer lo mismo, con características como:
- Todas las instrucciones tienen la **misma longitud** (en ARC, 32 bits)
- Todas están en direcciones de memoria **múltiplos de 4**
- **Solo** existen dos instrucciones que tocan memoria: "guardar" (`st`) y "recuperar" (`ld`), el resto opera en los registros
- Los dispositivos de E/S están mapeados en memoria
Lo más importante de RISC, es que cada instrucción assembler se ejecuta en un ciclo de reloj