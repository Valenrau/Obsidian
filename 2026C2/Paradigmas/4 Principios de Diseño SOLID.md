# ¿Cómo evaluar un diseño?

La presentación propone varias características para analizar la calidad de un diseño:

- **Escalabilidad:** cómo puede crecer el diseño a lo largo del tiempo.
    
- **Fragilidad:** cómo se comporta frente a casos no esperados.
    
- **Rigidez:** cómo se comporta frente a cambios.
    
- **Inmovilidad:** qué tan reutilizable es.
    

También se pueden analizar las relaciones entre módulos mediante:

- **Cohesión**
    
- **Acoplamiento**
    

## Objetivo general

> **Alta cohesión y bajo acoplamiento.**

### Cohesión

Indica qué tan relacionadas están entre sí las responsabilidades de un módulo o clase.

Una clase con alta cohesión tiene elementos que trabajan juntos para una responsabilidad clara.

### Acoplamiento

Indica cuánto depende un módulo de otros módulos.

Un bajo acoplamiento permite modificar una parte del sistema con menor impacto sobre las demás.

---

# S.O.L.I.D.

SOLID es un acrónimo atribuido a **Robert C. Martin** y reúne cinco principios de diseño orientados a objetos.

```
S → Single Responsibility Principle
O → Open-Closed Principle
L → Liskov Substitution Principle
I → Interface Segregation Principle
D → Dependency Inversion Principle
```

El objetivo general de estos principios es ayudar a construir sistemas con mejor:

- mantenibilidad;
    
- extensibilidad;
    
- reutilización;
    
- flexibilidad ante cambios.
    

---

# S — Single Responsibility Principle (SRP)

## Principio de responsabilidad única

La idea central es:

> Una clase debería tener una única responsabilidad.

En la presentación, una **responsabilidad** se define como una **razón de cambio**.

Por lo tanto:

> Si una clase tiene más de un motivo para cambiar, entonces probablemente tiene más de una responsabilidad.

## Ejemplo

Supongamos una clase:

```
class Informe {
    void generar();
    void guardarEnArchivo();
    void enviarPorEmail();
}
```

Esta clase puede cambiar por diferentes razones:

- cambia la forma de generar el informe;
    
- cambia la forma de guardar archivos;
    
- cambia la forma de enviar emails.
    

Eso indica que contiene varias responsabilidades.

Una posible separación sería:

```
class GeneradorInforme {
    Informe generar();
}

class GuardadorInforme {
    void guardar(Informe informe);
}

class EnviadorInforme {
    void enviar(Informe informe);
}
```

## Idea clave

Una clase debería concentrarse en **una razón principal para cambiar**.

Esto favorece una mayor cohesión y hace que los cambios estén más aislados.

---

# O — Open-Closed Principle (OCP)

## Principio abierto-cerrado

La formulación presentada es:

> Las entidades de software deberían estar **abiertas a extensiones pero cerradas a modificaciones**.

Esto parece contradictorio inicialmente.

## Abierto para extensión

Significa que el comportamiento de un módulo puede ampliarse.

Podemos agregar nuevas formas de comportamiento cuando cambian los requisitos.

## Cerrado para modificación

Significa que no debería ser necesario modificar el código existente cada vez que queremos agregar un nuevo comportamiento.

## Ejemplo

Supongamos:

```
if (figura instanceof Circulo) {
    dibujarCirculo(figura);
} else if (figura instanceof Rectangulo) {
    dibujarRectangulo(figura);
}
```

Si aparece una nueva figura:

```
Triangulo
```

tenemos que modificar este código.

El módulo no está cerrado a cambios.

Una alternativa es utilizar polimorfismo:

```
interface Figura {
    void dibujar();
}
```

```
class Circulo implements Figura {
    public void dibujar() {
        // ...
    }
}
```

```
class Rectangulo implements Figura {
    public void dibujar() {
        // ...
    }
}
```

Ahora el cliente puede trabajar con:

```
for (Figura figura : figuras) {
    figura.dibujar();
}
```

Agregar una nueva figura implica extender el sistema:

```
class Triangulo implements Figura {
    public void dibujar() {
        // ...
    }
}
```

sin modificar la lógica existente que trabaja con `Figura`.

## Idea clave

> Agregar comportamiento nuevo debería requerir, idealmente, extender el sistema y no modificar constantemente código que ya funciona.

---

# L — Liskov Substitution Principle (LSP)

## Principio de sustitución de Liskov

La idea central presentada es:

> Las funciones que utilizan referencias a clases base deberían poder utilizar objetos de clases derivadas sin necesidad de conocer que fueron sustituidos.

En otras palabras:

Si un programa funciona con un objeto de un tipo base, debería poder utilizar una instancia de un subtipo sin que el comportamiento esperado del programa se rompa.

## Ejemplo conceptual

Supongamos:

```
void procesar(Figura figura) {
    figura.dibujar();
}
```

El método debería poder recibir:

```
Circulo
Rectangulo
Triangulo
```

si todos son subtipos válidos de `Figura`.

El código que utiliza `Figura` no debería necesitar hacer:

```
if (figura instanceof Circulo) {
    ...
}
```

para que el programa funcione correctamente.

## Idea clave

> Un subtipo debe poder reemplazar al tipo base respetando el comportamiento que el cliente espera de esa abstracción.

No alcanza simplemente con que exista una relación de herencia en el código: la sustitución debe mantener el comportamiento esperado.

---

# I — Interface Segregation Principle (ISP)

## Principio de segregación de interfaces

La presentación lo resume como:

> Los clientes no deberían ser forzados a depender de interfaces que no usan.

Una interfaz demasiado grande puede obligar a una clase a conocer o implementar operaciones que no necesita.

## Ejemplo

Supongamos:

```
interface Trabajador {
    void trabajar();
    void comer();
    void dormir();
}
```

Puede haber implementaciones que solamente necesiten una parte de esa interfaz.

En lugar de una interfaz grande, se pueden separar responsabilidades:

```
interface Trabajable {
    void trabajar();
}

interface Comible {
    void comer();
}

interface Dormible {
    void dormir();
}
```

Cada cliente depende solamente de lo que realmente necesita.

## Ejemplo conceptual de la presentación

La presentación utiliza una interfaz de usuario de cajero y distintos tipos de transacciones:

- depósito;
    
- extracción;
    
- transferencia.
    

La idea es que las distintas partes del sistema dependan de las interfaces específicas que necesitan, en lugar de una interfaz enorme que reúna todos los comportamientos posibles.

## Idea clave

> Es preferible tener varias interfaces pequeñas y específicas antes que una interfaz enorme que obligue a depender de funcionalidades innecesarias.

Esto reduce el acoplamiento.

---

# D — Dependency Inversion Principle (DIP)

## Principio de inversión de dependencias

La presentación establece dos ideas fundamentales.

### A

> Los módulos de alto nivel no deben depender de módulos de bajo nivel. Ambos deben depender de abstracciones.

### B

> Las abstracciones no deben depender de detalles. Los detalles deben depender de abstracciones.

## ¿Qué significa?

Un módulo de alto nivel contiene una lógica más general o importante para el negocio.

Un módulo de bajo nivel suele encargarse de detalles concretos de implementación.

El principio propone que la lógica general no dependa directamente de detalles concretos.

## Ejemplo

Supongamos:

```
class ServicioPedidos {
    private BaseDeDatosMySQL baseDeDatos;
}
```

`ServicioPedidos` depende directamente de un detalle concreto: `BaseDeDatosMySQL`.

Una alternativa es depender de una abstracción:

```
interface RepositorioPedidos {
    void guardar(Pedido pedido);
}
```

```
class ServicioPedidos {
    private RepositorioPedidos repositorio;

    public ServicioPedidos(RepositorioPedidos repositorio) {
        this.repositorio = repositorio;
    }
}
```

Ahora podemos tener diferentes detalles:

```
class RepositorioMySQL implements RepositorioPedidos {
    // ...
}
```

```
class RepositorioMemoria implements RepositorioPedidos {
    // ...
}
```

El módulo de alto nivel depende de:

```
RepositorioPedidos
```

y las implementaciones concretas también dependen de esa abstracción.

## Idea clave

> La lógica importante del sistema no debería quedar atada directamente a detalles concretos de implementación.

Esto facilita:

- cambiar implementaciones;
    
- realizar pruebas;
    
- reutilizar componentes;
    
- reducir acoplamiento.
    

---

# Relación entre los principios

Los principios SOLID no son reglas aisladas.

Se relacionan entre sí.

## SRP + Cohesión

Una clase con una única razón de cambio suele tener una responsabilidad más clara y, por lo tanto, mayor cohesión.

## OCP + Polimorfismo

El polimorfismo puede permitir agregar nuevos comportamientos extendiendo el sistema sin modificar código existente.

## LSP + Herencia

La herencia debe representar una relación en la que el subtipo pueda reemplazar correctamente al tipo base.

## ISP + Bajo acoplamiento

Interfaces pequeñas permiten que los clientes dependan solamente de lo que necesitan.

## DIP + Abstracciones

Las abstracciones reducen la dependencia directa entre la lógica general y los detalles concretos.