 
## Qué es?

Es un paradigma de programacion, que se basa en organizar partes del codigo en objetos

Cada objeto tiene un **comportamiento y estado**. Ambos son invocados y/o modificados al interactuar entre ellos, mediante mensajes que un objeto pueda darle a otro.

## Qué es un objeto?

Es una herramienta que nos proveen ciertos lenguajes de programacion para organizar el estado y funcionamiento del programa en un tipo de estructura particular

Los objetos se crean a partir de una plantilla llamada **clase**, cada objeto es una **instancia** de su clase
Clase "Perro" -> instanciacion -> Objeto "Firu"

Cada objeto tiene sus atributos que son las variables o datos que pertenecen a una clase u objeto; representan su **estado** o características y los métodos, que son las funciones o bloques de código definidos dentro de una clase; representan el **comportamiento** o las acciones que el objeto puede realizar

# Objeto

Un **objeto** es una entidad en el modelo de computacion que busca representar un concepto que se comunica mediante mensajes. Los mensajes a los que respode un objeto se denomia **interfaz o protocolo**.

La forma en que un objeto realizara una operacion disparada por un mensaje recibido, se describe en un **metodo**. La ejecucion de estas operaciones puede estar condicionada por un **estado** interno del objeto. Dicho estado se ve reflejado en **atributos** del objeto

## Mensajes

Es una forma de invocar comportamiento en un software donde 2 actores se comunican mediante mensajes. En general en POO el pasaje de mensajes es sincronico y local (invocar una funcion en un programa).

Esta es la unica manera de interactuar con un objeto, por lo que un objeto no depende de como otro realiza X accion, sino de lo que devuelve (Que hace y no como).
El proceso de establecer la asociacion de un mensaje recibido con el metodo que debe ejecutarse se lo llama **method dispatch**

Los atributos del estado interno de los objetos, por defecto son **privados**

## Variantes de la POO

La creación de objetos e indican cómo será implementada la entidad se hace con:

- Clasificacion:
	- Existen “moldes” abstractos de entidades del dominio, denominados clases, que definen el comportamiento y forma de un conjunto de objetos.
	- Cada objeto se crea -o bien, se instancia- a partir de uno de estos moldes. Todo objeto es una instancia
- Prototipado:
	- Se construyen instancias concretas de un conjunto de objetos, llamadas **prototipos**.
	- Los objetos se crean a partir de clonar estos prototipos y luego se los modifica.

## Porque POO

Refuerza atributos de calidad del desarrollo de software:

- Modularidad: los lenguajes orientados a objetos incentivan la modularizacion, haciendo que la implementacion de cada objeto quede en su propio módulo
- Reusabilidad: dada la modularidad misma, una vez ya implementada la interfaz, es facil la re-utilizacion y evitar repetir código
- Extensibilidad: es posible seguir agregando funcionalidad, extendiendo interfaces y creando nuevos objetos que sepan operar con las interfaces actuales
- Mantenibilidad: gracias a la modularidad y reusabilidad facilita la mantenibilidad. Mas objetos, menos complejos. Implementacion y testing de cada objeto por separado

> Todo lo anterior se sostiene por los cuatro pilares de la programacion orientada a objetos

--- 

# Abstraccion

La abstracción consiste en representar un objeto mediante las características y comportamientos relevantes para el problema que estamos resolviendo, ocultando o ignorando los detalles que no son necesarios. Escondiendo detalles de **como se hace algo**, mientras que el usuario solo sabe su funcionalidad (**qué hace**)

```
class Auto:
    def acelerar(self):
        pass

    def frenar(self):
        pass
```

Cuando usamos el objeto, nos interesa saber que un auto puede acelerar y frenar, no conocer detalles de *como* lo hace