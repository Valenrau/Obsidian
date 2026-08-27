
## Arquitectura vs Microarquitectura

- **ISA (Instruction Set Architecture):** Es el hardware visible al programador (instrucciones, registros, modos de direccionamiento). Es el "contrato" funcional.
- **Microarquitectura:** Es una implementación especifica en hardware de esa ISA.
	- **Concepto clave:** Un mismo ISA (ARC o x86) puede tener **múltiples microarquitecturas** diferentes
	- **Objetivos de diseño:** Se pueden diseñar microarquitecturas enfocadas en **mayor velocidad**, **menor costo** o **menor consumo**, todas ejecutando el mismo código de maquina

## Organización del Trayecto de Datos

El Trayecto de Datos es el conjunto de unidades funcionales (ALU, Registros) y las interconexiones (Buses) que procesan la información. 

### Componentes generales

Son los componentes que se encargan de llevar a cabo las operaciones que se le indiquen a la cpu.

- **Bus**: Camino por el cual viajan los datos entre los bloques
- **General Purpose Registers (GPR):** El banco de registros de propósito general, donde la cpu guarda datos temporales que el programa esta usando.
- **Program Counter (PC):** Guarda la dirección de la próxima instrucción a ejecutar. Se actualiza automáticamente para apuntar a la siguiente instrucción.
- **Instruction Register (IR):** Guarda la instrucción que se acaba de traer de memoria para que la unidad de control la decodifique y sepa que operación ejecutar.
- **Memory Address Register (MAR):** Guarda la **dirección** de memoria al a que se quiere acceder (leer o escribir). Es el que le dice a la memoria *donde* mirar.
- **Memory Data Register (MDR):** Guarda el **dato** que se lee o se escribe en esa dirección. Es el que efectivamente transporta el contenido (a diferencia del MAR que transporta la dirección)
- **A y B (Registros buffer de operandos):** Son registros intermedios que "sostienen" los operadores justo antes de entrar a la ALU. Son necesarios en función de la cantidad de buses que haya, si hay 1 bus habrá dos buffers, si hay 2 habrá 1, y si hay 3 no habrá ninguno

- **Organización de 1 BUS:** Todos los componentes comparten un solo bus.
	- *Funcionamiento:* Para hacer **A + B**, se requiere leer **A** (1 ciclo), leer **B** (1 ciclo), operar y guardar (1 ciclo). Es lento porque el bus es un cuello de botella.
	
	![[Pasted image 20260723130117.png]]

- **Organización de 2 Buses:** Permite separar el tráfico de entrada y salida en cierta medida, mejorando levemente el flujo.
	
	![[Pasted image 20260723140740.png]]

- Organización de 3 Buses (Caso ARC): Es la mas eficiente y la usada en el diseño ARC presentado.
	- *Estructura:* Dos buses de origen (**Bus A y Bus B**) llevan los datos a la ALU, y un bus de destino (**Bus C**) recoge el resultado.
	- *Ventaja:* permite realizar una operación aritmética completa (leer dos operandos, calcular y guardar el resultado) en **un solo ciclo de reloj**.
	
	![[Pasted image 20260723141128.png]]

## Implementacion Interna de la ALU

### Tablas de Consulta (Look-up Tables - LUT)

En lugar de usar ciurcuitos lógicos complejos dedicados para cada operacion matematica, se propone el uso de LUTs.

- Una LUT es escencialmente una memoria que, dada una combinacion de entradas (operandos y codigo de operacion), "devuelve" el resultado precalculado.
- Se usan 32 LUTs de 1 bit para procesar palabras de 32 bits, generando tambien los flags (*N, Z, V, C*)

### Desplazador Rapido (Barrel Shifter)

Para las operaciones de desplazamiento (shifts), no se usan registros de desplazamiento convencionales (que tardan **N** ciclos para mover **N** posiciones).

- Se implementa un **Barrel Shifter**: un circuito combinacional basado en niveles de multiplexores que permite desplazar o rotar una palabra de 32 bits cualquier cantidad de posiciones (de 0 a 31) en **un solo ciclo de reloj**.

## Microarquitectura Especifica de ARC

Se detalla el circuito completo del procesador ARC:

- **Registros:** Tienen una estructura "Tri-state" para conectarse a los buses A y B, y se escriben desde el bus C.
	- *Excepciones:* El registro `%r0` está cableado a cero (sin flip-flops). El `%pc` (Contador de Programa) solo guarda múltiplos de 4 (los 2 bits menos significativos son 0). El `%ir` (Registro de Instrucción) tiene salidas cableadas específicas para decodificar los campos `op`, `rd`, `rs1`, etc.
- **Sección de Control:** Genera las señales para controlar los MUX, la ALU y la escritura en registros.

## Diseño de una Unidad de Control

Podemos diseñar una **unidad de control** por una lógica microprogramada (firmware) y/o una lógica cableada. El trayecto de datos es esencialmente idéntico en ambos casos

### Funcionamiento

La CPU ejecuta un "programa interno" (microprograma) almacenado en una memoria ROM llamada **Control Store**.

- **Microinstruccion:** Una palabra muy ancha (41 bits en este ejemplo) donde cada bit controla directamente una compuerta o señal del hardware (ej `ALU function`, `Load Register`, `Read Memory`).
- **Secuencia:**
	1. Se lee una microinstruccion del Control Store al registro **MIR** (Microinstruction Register).
	2. Los bits del MIR activan físicamente las señales de control.
	3. Se calcula la direccion de la *siguiente* microinstruccion