# Qué es?

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

> Todo lo anterior se sostiene por los cuatro pilares de la programación orientada a objetos

--- 

# Abstracción

La abstracción consiste en representar un objeto mediante las características y comportamientos relevantes para el problema que estamos resolviendo, ocultando o ignorando los detalles que no son necesarios. Escondiendo detalles de **como se hace algo**, mientras que el usuario solo sabe su funcionalidad (**qué hace**)

```
class Auto {
    void acelerar() {
        System.out.println("Acelerando...");
    }

    void frenar() {
        System.out.println("Frenando...");
    }
}
```

Cuando usamos el objeto, nos interesa saber que un auto puede acelerar y frenar, no conocer detalles de *como* lo hace

---

# Encapsulamiento

El encapsulamiento consiste en agrupar los datos y los comportamientos de un objeto y controlar el acceso a su estado interno, evitando que pueda modificarse directamente de forma indebida.

```
class CuentaBancaria {
    private double saldo;

    public CuentaBancaria(double saldo) {
        this.saldo = saldo;
    }

    public void depositar(double monto) {
        if (monto > 0) {
            saldo += monto;
        }
    }

    public double obtenerSaldo() {
        return saldo;
    }
}
```

Aquí, el atributo `saldo` no debería modificarse directamente desde fuera. En cambio, se interactúa con él mediante métodos como `depositar()`

El encapsulamiento consiste en utilizar la clase como una cápsula que agrupa y contiene tanto los datos (atributos) como los comportamientos (métodos) de un objeto, controlando además cómo se puede acceder o modificar su estado interno.

---

# Herencia

La herencia consiste en crear una clase a partir de otra, permitiendo que la nueva clase herede sus atributos y comportamientos y pueda además agregar o modificar los propios.

```
class Animal {
    void comer() {
        System.out.println("Estoy comiendo");
    }
}

class Perro extends Animal {
    void ladrar() {
        System.out.println("Guau");
    }
}
```

En este caso, `Perro` **hereda** de `Animal`, por lo que puede utilizar `comer()`, además de tener su propio comportamiento `ladrar()`

Tipos de herencia:

- Clases concretas: son clases que van a tener una utilidad final y las que van a terminar ejecutando el código que desarrollemos.
- Clases abstractas: son aquellas que declarar ciertos atributos o métodos, pero dejan sin implementar uno o mas de ellos, forzando a que las subclases las implementen.
  Estas clases no se pueden instanciar.

---
# Polimorfismo

El polimorfismo consiste en permitir que distintos objetos respondan de manera diferente ante la misma operación o mensaje.
Si bien no es un concepto propio de la POO, es un concepto que gustó mucho a los desarrolladores de la programación orientada a objetos y decidieron incorporarlo al paradigma.

```
class Animal {
    void hacerSonido() {
        System.out.println("Sonido");
    }
}

class Perro extends Animal {
    @Override
    void hacerSonido() {
        System.out.println("Guau");
    }
}

class Gato extends Animal {
    @Override
    void hacerSonido() {
        System.out.println("Miau");
    }
}
```

Aunque `Perro` y `Gato` responden al mismo método `hacerSonido()`, cada uno lo hace de una manera diferente.

Hay distintas maneras de lograr que un mismo método pueda trabajar con distintos tipos de datos u objetos:

---
### Polimorfismo Ad Hoc

El polimorfismo Ad Hoc ocurre cuando una misma operación tiene distintas implementaciones según el tipo o cantidad de argumentos que recibe.

```
class Calculadora {
    int sumar(int a, int b) {
        return a + b;
    }

    double sumar(double a, double b) {
        return a + b;
    }

    int sumar(int a, int b, int c) {
        return a + b + c;
    }
}
```

En todos los casos utilizamos `sumar`, pero la implementación que se ejecuta depende de los argumentos.

**Idea clave:** mismo nombre/operación → distintas implementaciones determinadas por los parámetros.

---
### Polimorfismo Paramétrico

El polimorfismo paramétrico permite definir una operación o estructura de manera independiente del tipo concreto de los datos con los que trabaja.

```
class Caja<T> {
    private T contenido;

    void guardar(T contenido) {
        this.contenido = contenido;
    }

    T obtener() {
        return contenido;
    }
}
```

Ahora podemos utilizar la misma clase con distintos tipos:

```
Caja<String> cajaTexto = new Caja<>();
Caja<Integer> cajaNumero = new Caja<>();
```

La `Caja<T>` no necesita saber de antemano si va a contener un `String`, un `Integer` u otro tipo.

**Idea clave:** una misma implementación → funciona para muchos tipos mediante un parámetro de tipo.

---
### Polimorfismo por Subtipado

El polimorfismo por subtipado permite tratar un objeto de un subtipo como si fuera un objeto de su supertipo, haciendo que el comportamiento concreto dependa del objeto que realmente se encuentra en ejecución. Cuando 2 objetos que pertenecen a la misma jerarquía de clases (herencia) y sobreescriben el metodo de una clase primaria.

```
class Animal {
    void hacerSonido() {
        System.out.println("Sonido");
    }
}

class Perro extends Animal {
    @Override
    void hacerSonido() {
        System.out.println("Guau");
    }
}

class Gato extends Animal {
    @Override
    void hacerSonido() {
        System.out.println("Miau");
    }
}
```

```
Animal animal1 = new Perro();
Animal animal2 = new Gato();

animal1.hacerSonido(); // Guau
animal2.hacerSonido(); // Miau
```

Aunque las variables son de tipo `Animal`, el método que se ejecuta depende del objeto concreto (`Perro` o `Gato`).

**Idea clave:** un objeto de un subtipo puede utilizarse donde se espera su supertipo.

---
### Polimorfismo mediante Interfaces

El polimorfismo mediante interfaces permite que distintas clases, aunque no estén relacionadas por herencia entre sí, sean tratadas de la misma manera cuando implementan una misma interfaz.

Una interfaz es una descripción de un conjunto de métodos. Un objeto puede implementar una interfaz si este implementa cada uno de los métodos que describe la interfaz. Las interfaces formalizan el polimorfismo, ya que permiten definir polimorfismo
en una forma declarativa, sin detalles de implementación.

```
interface Volador {
    void volar();
}

class Pajaro implements Volador {
    public void volar() {
        System.out.println("El pájaro vuela");
    }
}

class Avion implements Volador {
    public void volar() {
        System.out.println("El avión vuela");
    }
}
```

Podemos hacer:

```
Volador v1 = new Pajaro();
Volador v2 = new Avion();

v1.volar(); // El pájaro vuela
v2.volar(); // El avión vuela
```

Tanto `Pajaro` como `Avion` cumplen el contrato de `Volador`, por lo que podemos tratarlos de la misma manera, aunque conceptualmente sean objetos completamente diferentes.

**Idea clave:** distintas clases → mismo contrato (interfaz) → pueden utilizarse de manera polimórfica.