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

```text
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

> **Importante:** los principios SOLID no son reglas absolutas ni funcionan de manera aislada.  
> Un mismo diseño puede aplicar —o violar— varios principios al mismo tiempo.

---

# S — Single Responsibility Principle (SRP)

## Principio de responsabilidad única

La idea central es:

> **Una clase debería tener una única responsabilidad.**

Pero la forma más útil de entenderlo es:

> **Una clase debería tener una única razón importante para cambiar.**

Una responsabilidad no significa necesariamente “un solo método”.

Una clase puede tener varios métodos si todos colaboran con la misma responsabilidad.

## Ejemplo problemático

```java
class Informe {
    void generar() {
        // lógica para generar el informe
    }

    void guardarEnArchivo() {
        // lógica de archivos
    }

    void enviarPorEmail() {
        // lógica de email
    }
}
```

Esta clase puede cambiar por al menos tres motivos diferentes:

- cambia la forma de generar el informe;
- cambia la forma de guardarlo;
- cambia la forma de enviarlo por email.

Por lo tanto, tiene varias razones de cambio.

## Mejor separación

```java
class GeneradorInforme {
    Informe generar() {
        // lógica de generación
        return new Informe();
    }
}
```

```java
class GuardadorInforme {
    void guardar(Informe informe) {
        // persistencia
    }
}
```

```java
class EnviadorInforme {
    void enviar(Informe informe) {
        // envío de email
    }
}
```

Cada clase tiene ahora un motivo de cambio más claro.

## Ejemplo donde NO se viola SRP

```java
class Cuenta {
    private double saldo;

    void depositar(double monto) {
        saldo += monto;
    }

    void retirar(double monto) {
        saldo -= monto;
    }

    double consultarSaldo() {
        return saldo;
    }
}
```

Aunque `Cuenta` tiene varios métodos, todos pertenecen a la misma responsabilidad:

> administrar el estado y las operaciones de una cuenta.

SRP **no significa** “una clase = un método”.

## Pregunta mental

> **¿Por cuántos motivos conceptualmente distintos podría tener que modificar esta clase?**

Si la respuesta incluye cosas como:

- “si cambia la base de datos”;
- “si cambia la lógica de negocio”;
- “si cambia el formato del PDF”;
- “si cambia el proveedor de emails”;

probablemente la clase tenga más de una responsabilidad.

## Cómo diferenciar SRP de ISP

Los dos pueden parecerse porque ambos buscan componentes más enfocados.

Pero miran cosas distintas:

- **SRP:** analiza las **responsabilidades de una clase o módulo**.
- **ISP:** analiza las **interfaces que consumen los clientes**.

Ejemplo:

```java
class ImpresoraMultifuncion {
    void imprimir();
    void escanear();
    void enviarFax();
}
```

Si esta clase realmente representa una multifunción, puede tener sentido que tenga esas capacidades y no necesariamente viola SRP.

Pero si obligamos a una impresora simple a implementar:

```java
interface Dispositivo {
    void imprimir();
    void escanear();
    void enviarFax();
}
```

aparece un problema de **ISP**, porque el cliente queda obligado a depender de operaciones que no necesita.

## Idea clave

> **SRP = una razón principal de cambio.**

Favorece alta cohesión y reduce el impacto de los cambios.

---

# O — Open-Closed Principle (OCP)

## Principio abierto-cerrado

La formulación clásica es:

> **Las entidades de software deberían estar abiertas a extensión pero cerradas a modificación.**

Esto significa:

- **abiertas a extensión:** podemos agregar nuevos comportamientos;
- **cerradas a modificación:** idealmente no necesitamos tocar constantemente código que ya funciona para agregar esos comportamientos.

No significa que “nunca se puede modificar código existente”.

Significa que conviene diseñar los **puntos de variación previsibles** para poder extenderlos.

## Ejemplo problemático

```java
class CalculadorDescuento {

    double calcular(String tipoCliente, double total) {

        if (tipoCliente.equals("NORMAL")) {
            return total;
        }

        if (tipoCliente.equals("PREMIUM")) {
            return total * 0.90;
        }

        if (tipoCliente.equals("VIP")) {
            return total * 0.80;
        }

        return total;
    }
}
```

Cada vez que aparece un nuevo tipo de descuento tenemos que modificar `CalculadorDescuento`.

Por ejemplo:

```text
EMPRESA
EMPLEADO
SOCIO
```

El módulo cambia constantemente.

## Mejor solución

```java
interface PoliticaDescuento {
    double aplicar(double total);
}
```

```java
class DescuentoNormal implements PoliticaDescuento {
    public double aplicar(double total) {
        return total;
    }
}
```

```java
class DescuentoPremium implements PoliticaDescuento {
    public double aplicar(double total) {
        return total * 0.90;
    }
}
```

```java
class DescuentoVip implements PoliticaDescuento {
    public double aplicar(double total) {
        return total * 0.80;
    }
}
```

El calculador trabaja con la abstracción:

```java
class CalculadorDescuento {

    double calcular(
        PoliticaDescuento politica,
        double total
    ) {
        return politica.aplicar(total);
    }
}
```

Si mañana aparece:

```java
class DescuentoEmpresa implements PoliticaDescuento {
    public double aplicar(double total) {
        return total * 0.85;
    }
}
```

podemos extender el sistema sin modificar la lógica de `CalculadorDescuento`.

## Ejemplo clásico con figuras

### Problema

```java
if (figura instanceof Circulo) {
    dibujarCirculo(figura);
} else if (figura instanceof Rectangulo) {
    dibujarRectangulo(figura);
}
```

Agregar `Triangulo` obliga a editar ese `if`.

### Mejor

```java
interface Figura {
    void dibujar();
}
```

```java
class Circulo implements Figura {
    public void dibujar() { }
}
```

```java
class Rectangulo implements Figura {
    public void dibujar() { }
}
```

Y el cliente:

```java
for (Figura figura : figuras) {
    figura.dibujar();
}
```

Agregar una nueva figura consiste en crear otra implementación.

## Ojo: OCP no significa “crear interfaces para todo”

Si una parte del sistema es estable y no existe una variación razonable, agregar abstracciones puede introducir complejidad sin beneficio.

OCP apunta a proteger las partes que **realmente cambian o pueden variar**.

## Pregunta mental

> **Cuando agrego una variante nueva, ¿tengo que editar una cadena de `if`, `switch` o código estable?**

Si sí, puede haber una oportunidad de aplicar OCP.

## Cómo diferenciar OCP de DIP

Pueden usar abstracciones y parecer similares.

- **OCP:** se pregunta cómo **agregar nuevas variantes sin modificar código estable**.
- **DIP:** se pregunta **de qué depende la lógica de alto nivel**.

Ejemplo:

```java
interface MedioPago {
    void pagar(double monto);
}
```

Permitir agregar:

```java
Tarjeta
MercadoPago
Transferencia
```

sin cambiar el procesador apunta a **OCP**.

Que `ProcesadorCompra` dependa de `MedioPago` y no directamente de `MercadoPagoSDK` apunta a **DIP**.

## Idea clave

> **OCP = agregar comportamiento extendiendo, no reescribiendo constantemente lo existente.**

---

# L — Liskov Substitution Principle (LSP)

## Principio de sustitución de Liskov

La idea central es:

> **Un objeto de un subtipo debería poder reemplazar a un objeto del tipo base sin romper el comportamiento esperado del programa.**

No alcanza con que el código compile.

La clase derivada debe respetar el **contrato** que esperan los clientes de la clase base.

## Ejemplo clásico: rectángulo y cuadrado

Supongamos:

```java
class Rectangulo {
    protected int ancho;
    protected int alto;

    void setAncho(int ancho) {
        this.ancho = ancho;
    }

    void setAlto(int alto) {
        this.alto = alto;
    }

    int area() {
        return ancho * alto;
    }
}
```

Ahora hacemos:

```java
class Cuadrado extends Rectangulo {

    @Override
    void setAncho(int ancho) {
        this.ancho = ancho;
        this.alto = ancho;
    }

    @Override
    void setAlto(int alto) {
        this.alto = alto;
        this.ancho = alto;
    }
}
```

Parece lógico desde el mundo real:

> un cuadrado es un rectángulo.

Pero veamos un cliente:

```java
void configurar(Rectangulo r) {
    r.setAncho(5);
    r.setAlto(10);

    System.out.println(r.area());
}
```

Con un `Rectangulo`, el área esperada es:

```text
50
```

Pero con un `Cuadrado`, la última operación deja:

```text
ancho = 10
alto = 10
```

y el área será:

```text
100
```

El subtipo cambió una expectativa válida del contrato de `Rectangulo`.

Por lo tanto, esa herencia viola LSP.

## Otro ejemplo más fácil

```java
class Ave {
    void volar() { }
}
```

```java
class Pinguino extends Ave {
    @Override
    void volar() {
        throw new UnsupportedOperationException();
    }
}
```

Si todo `Ave` promete que puede ejecutar:

```java
ave.volar();
```

un `Pinguino` no puede sustituir correctamente a `Ave`.

El problema no es que “el pingüino esté mal”.

El problema es que la abstracción `Ave` fue diseñada con un contrato demasiado fuerte.

Una alternativa:

```java
interface Ave {
}
```

```java
interface AveVoladora extends Ave {
    void volar();
}
```

```java
class Aguila implements AveVoladora {
    public void volar() { }
}
```

```java
class Pinguino implements Ave {
}
```

## Señales típicas de violación de LSP

Un subtipo sospechoso suele:

- lanzar excepciones para métodos que el padre permite;
- no soportar operaciones heredadas;
- necesitar restricciones más fuertes;
- devolver resultados incompatibles con lo esperado;
- exigir al cliente preguntar qué subtipo recibió.

Ejemplo:

```java
if (cuenta instanceof CuentaAhorro) {
    ...
}
```

Ese `instanceof` no siempre es una violación, pero puede ser una señal de que la abstracción no permite sustitución real.

## Pregunta mental

> **Si reemplazo el tipo base por este subtipo, ¿el cliente sigue funcionando bajo las mismas expectativas?**

Si la respuesta es no, probablemente se viola LSP.

## Cómo diferenciar LSP de OCP

OCP suele usar polimorfismo, pero para que esa extensión sea segura los subtipos deben respetar LSP.

Ejemplo:

```java
interface MedioPago {
    void pagar(double monto);
}
```

Agregar una nueva implementación:

```java
class Transferencia implements MedioPago
```

ayuda a **OCP**.

Pero si:

```java
class Transferencia implements MedioPago {
    public void pagar(double monto) {
        throw new UnsupportedOperationException();
    }
}
```

esa implementación rompe **LSP**.

En resumen:

> **OCP permite agregar nuevas implementaciones.**  
> **LSP exige que esas implementaciones sean sustitutos válidos.**

## Idea clave

> **LSP = heredar o implementar no alcanza; hay que respetar el contrato esperado.**

---

# I — Interface Segregation Principle (ISP)

## Principio de segregación de interfaces

La idea central es:

> **Los clientes no deberían ser forzados a depender de métodos que no usan.**

Es preferible tener interfaces más pequeñas y orientadas a cada necesidad que una interfaz enorme con operaciones para todos los casos.

## Ejemplo problemático

```java
interface MaquinaOficina {
    void imprimir();
    void escanear();
    void enviarFax();
}
```

Una multifunción puede implementar todo:

```java
class Multifuncion implements MaquinaOficina {
    public void imprimir() { }
    public void escanear() { }
    public void enviarFax() { }
}
```

Pero una impresora simple queda obligada a hacer esto:

```java
class ImpresoraSimple implements MaquinaOficina {

    public void imprimir() { }

    public void escanear() {
        throw new UnsupportedOperationException();
    }

    public void enviarFax() {
        throw new UnsupportedOperationException();
    }
}
```

La interfaz obliga a la clase a depender de operaciones que no necesita.

## Mejor solución

```java
interface Imprimible {
    void imprimir();
}
```

```java
interface Escaneable {
    void escanear();
}
```

```java
interface EnviadorFax {
    void enviarFax();
}
```

Entonces:

```java
class ImpresoraSimple implements Imprimible {
    public void imprimir() { }
}
```

y:

```java
class Multifuncion
        implements Imprimible, Escaneable, EnviadorFax {

    public void imprimir() { }
    public void escanear() { }
    public void enviarFax() { }
}
```

Cada cliente depende únicamente de las capacidades que necesita.

## Otro ejemplo: sistema de cajero

Una interfaz demasiado grande:

```java
interface OperacionesCajero {
    void depositar();
    void extraer();
    void transferir();
    void solicitarPrestamo();
    void comprarDolares();
}
```

Un módulo que solamente procesa extracciones no debería quedar acoplado a todas las demás operaciones.

Podemos separar:

```java
interface Extraible {
    void extraer();
}
```

```java
interface Transferible {
    void transferir();
}
```

etc.

## Pregunta mental

> **¿Esta clase o cliente está obligado a conocer o implementar métodos que no necesita?**

Si sí, pensá en ISP.

## Cómo diferenciar ISP de SRP

Esta confusión es muy común.

### SRP

Pregunta:

> **¿La clase tiene más de una razón de cambio?**

Mira la **responsabilidad interna** del componente.

### ISP

Pregunta:

> **¿El cliente depende de más operaciones de las que realmente necesita?**

Mira el **contrato que se expone**.

Ejemplo:

```java
interface Repositorio {
    guardar();
    eliminar();
    exportarPDF();
    enviarEmail();
}
```

Puede haber un problema de **ISP** porque ciertos clientes sólo necesitan `guardar()` y `eliminar()`.

Y si una única clase implementa persistencia, PDF y email, también podría haber un problema de **SRP**.

Son problemas relacionados, pero no idénticos.

## Relación entre ISP y LSP

Si una interfaz obliga a implementar métodos que una clase no puede soportar y termina haciendo:

```java
throw new UnsupportedOperationException();
```

puede aparecer:

- **ISP**, porque la interfaz exige demasiado;
- **LSP**, porque el subtipo no cumple el contrato esperado.

## Idea clave

> **ISP = cada cliente debería ver únicamente el contrato que necesita.**

---

# D — Dependency Inversion Principle (DIP)

## Principio de inversión de dependencias

La formulación clásica tiene dos partes:

> **Los módulos de alto nivel no deben depender de módulos de bajo nivel. Ambos deben depender de abstracciones.**

y:

> **Las abstracciones no deben depender de detalles. Los detalles deben depender de abstracciones.**

## ¿Qué significa?

Un módulo de **alto nivel** expresa reglas importantes del negocio.

Un módulo de **bajo nivel** suele encargarse de detalles técnicos.

Ejemplos de detalles:

- MySQL;
- PostgreSQL;
- archivos;
- APIs externas;
- SMTP;
- SDK de Mercado Pago;
- librerías concretas.

La lógica de negocio no debería quedar fuertemente atada a uno de esos detalles.

## Ejemplo problemático

```java
class ServicioPedidos {

    private BaseDeDatosMySQL baseDeDatos;

    ServicioPedidos() {
        baseDeDatos = new BaseDeDatosMySQL();
    }

    void crearPedido(Pedido pedido) {
        // reglas de negocio
        baseDeDatos.guardar(pedido);
    }
}
```

`ServicioPedidos` depende directamente de:

```text
BaseDeDatosMySQL
```

Si queremos usar:

```text
PostgreSQL
MongoDB
RepositorioEnMemoria
```

tenemos que modificar el servicio.

## Mejor solución

Creamos una abstracción:

```java
interface RepositorioPedidos {
    void guardar(Pedido pedido);
}
```

La lógica de negocio depende de ella:

```java
class ServicioPedidos {

    private RepositorioPedidos repositorio;

    ServicioPedidos(RepositorioPedidos repositorio) {
        this.repositorio = repositorio;
    }

    void crearPedido(Pedido pedido) {
        // reglas de negocio
        repositorio.guardar(pedido);
    }
}
```

Los detalles concretos implementan la abstracción:

```java
class RepositorioMySQL implements RepositorioPedidos {

    public void guardar(Pedido pedido) {
        // SQL
    }
}
```

```java
class RepositorioMemoria implements RepositorioPedidos {

    public void guardar(Pedido pedido) {
        // memoria
    }
}
```

Ahora la dirección conceptual es:

```text
ServicioPedidos ──> RepositorioPedidos <── RepositorioMySQL
```

La regla de negocio no depende de MySQL.

MySQL se adapta al contrato que necesita la regla de negocio.

## Ejemplo con notificaciones

### Problema

```java
class ServicioRegistro {

    private EmailSender emailSender = new EmailSender();

    void registrar(Usuario usuario) {
        // registrar usuario
        emailSender.enviarBienvenida(usuario);
    }
}
```

### Mejor

```java
interface Notificador {
    void enviarBienvenida(Usuario usuario);
}
```

```java
class ServicioRegistro {

    private Notificador notificador;

    ServicioRegistro(Notificador notificador) {
        this.notificador = notificador;
    }

    void registrar(Usuario usuario) {
        // registrar usuario
        notificador.enviarBienvenida(usuario);
    }
}
```

Ahora podemos usar:

```java
EmailNotificador
SmsNotificador
NotificadorFake
```

sin cambiar la lógica del registro.

## DIP no es exactamente lo mismo que inyección de dependencias

Este punto conviene distinguirlo.

### Dependency Inversion Principle

Es un **principio de diseño**:

> la lógica importante depende de abstracciones y no de detalles concretos.

### Dependency Injection

Es una **técnica** para suministrar dependencias desde afuera.

Ejemplo:

```java
ServicioPedidos(RepositorioPedidos repositorio)
```

Esto es inyección por constructor.

La inyección de dependencias ayuda a aplicar DIP, pero no son exactamente lo mismo.

Por ejemplo:

```java
ServicioPedidos(BaseDeDatosMySQL mysql)
```

la dependencia es inyectada, pero el servicio todavía depende de una implementación concreta.

Hay inyección, pero DIP todavía es débil o inexistente.

## Pregunta mental

> **¿La lógica principal del sistema conoce directamente una tecnología o implementación concreta que podría cambiar?**

Si sí, pensá en DIP.

## Cómo diferenciar DIP de OCP

Los dos suelen usar interfaces.

### OCP

Pregunta:

> **¿Puedo agregar una variante nueva sin modificar código estable?**

### DIP

Pregunta:

> **¿La lógica de alto nivel depende de una abstracción o de un detalle concreto?**

Ejemplo:

```java
interface MedioPago {
    void pagar();
}
```

- Poder agregar `Tarjeta`, `Transferencia`, `MercadoPago` sin cambiar `Checkout` → **OCP**.
- Que `Checkout` dependa de `MedioPago` y no del SDK concreto de Mercado Pago → **DIP**.

## Cómo diferenciar DIP de ISP

- **ISP:** qué tan grande o específica es la interfaz para un cliente.
- **DIP:** en qué dirección están organizadas las dependencias.

Podemos tener una interfaz abstracta pero demasiado grande:

```java
interface SistemaExterno {
    void guardar();
    void enviarEmail();
    void generarPDF();
    void cobrar();
}
```

Depender de esa interfaz en lugar de una clase concreta puede ayudar a DIP.

Pero obligar a un cliente que sólo guarda datos a depender de todas las operaciones viola ISP.

## Idea clave

> **DIP = la lógica importante depende de contratos; los detalles concretos se adaptan a esos contratos.**

---

# Comparaciones que suelen confundirse

## SRP vs ISP

### SRP

```java
class Factura {
    void calcularTotal();
    void guardarEnBD();
    void enviarEmail();
}
```

Problema principal:

> la clase tiene varias razones diferentes para cambiar.

### ISP

```java
interface OperacionesFactura {
    void calcularTotal();
    void guardarEnBD();
    void enviarEmail();
}
```

Si un cliente sólo necesita calcular el total pero debe depender de toda la interfaz:

> el contrato es más grande de lo necesario.

### Regla mental

> **SRP = responsabilidades internas.**  
> **ISP = contrato que consume el cliente.**

---

## OCP vs LSP

### OCP

Queremos agregar:

```java
class PagoTransferencia implements MedioPago
```

sin modificar el código que procesa pagos.

Eso favorece:

> extensión sin modificación.

### LSP

Pero la implementación debe cumplir realmente el contrato:

```java
class PagoTransferencia implements MedioPago {

    public void pagar(double monto) {
        throw new UnsupportedOperationException();
    }
}
```

Esto rompe la sustitución.

### Regla mental

> **OCP = puedo sumar nuevos tipos.**  
> **LSP = esos nuevos tipos se comportan correctamente como el tipo esperado.**

---

## OCP vs DIP

Ambos suelen introducir interfaces.

### OCP

El foco está en:

> poder agregar variantes.

### DIP

El foco está en:

> evitar que la lógica importante dependa de detalles concretos.

### Regla mental

> **OCP = cómo extiendo.**  
> **DIP = de qué dependo.**

---

## ISP vs LSP

Supongamos:

```java
interface Ave {
    void volar();
}
```

y:

```java
class Pinguino implements Ave {
    public void volar() {
        throw new UnsupportedOperationException();
    }
}
```

### ISP

La interfaz le está exigiendo al pingüino una capacidad que no necesita o no posee.

### LSP

El pingüino no puede sustituir correctamente a cualquier `Ave` si el contrato exige `volar()`.

### Regla mental

> **ISP = el contrato exige demasiado.**  
> **LSP = el subtipo no puede cumplir correctamente el contrato.**

---

## DIP vs Dependency Injection

```java
class Servicio {
    Servicio(BaseDeDatosMySQL db) {
        ...
    }
}
```

La dependencia llega desde afuera:

> hay **inyección de dependencias**.

Pero el servicio sigue dependiendo de:

```text
BaseDeDatosMySQL
```

por lo que no necesariamente aplica DIP.

En cambio:

```java
class Servicio {
    Servicio(Repositorio repositorio) {
        ...
    }
}
```

y:

```java
class MySQLRepositorio implements Repositorio
```

representa mejor la inversión de dependencia.

### Regla mental

> **Inyección = cómo recibo la dependencia.**  
> **DIP = hacia qué tipo de dependencia apunto.**

---

# Una misma situación puede violar varios principios

Supongamos:

```java
class ServicioCompra {

    void procesar(String tipoPago, Compra compra) {

        BaseDeDatosMySQL db = new BaseDeDatosMySQL();

        if (tipoPago.equals("TARJETA")) {
            // pagar con tarjeta
        } else if (tipoPago.equals("TRANSFERENCIA")) {
            // pagar por transferencia
        }

        db.guardar(compra);

        // generar PDF

        // enviar email
    }
}
```

Acá aparecen varios problemas.

## SRP

La clase se ocupa de:

- procesar pagos;
- persistencia;
- generación de PDF;
- email.

Tiene varias razones de cambio.

## OCP

Agregar un nuevo medio de pago exige editar el `if`.

## DIP

La lógica depende directamente de:

```text
BaseDeDatosMySQL
```

un detalle concreto.

## Posible rediseño

```java
interface MedioPago {
    void pagar(Compra compra);
}
```

```java
interface RepositorioCompras {
    void guardar(Compra compra);
}
```

```java
interface NotificadorCompra {
    void notificar(Compra compra);
}
```

```java
class ServicioCompra {

    private MedioPago medioPago;
    private RepositorioCompras repositorio;
    private NotificadorCompra notificador;

    ServicioCompra(
        MedioPago medioPago,
        RepositorioCompras repositorio,
        NotificadorCompra notificador
    ) {
        this.medioPago = medioPago;
        this.repositorio = repositorio;
        this.notificador = notificador;
    }

    void procesar(Compra compra) {
        medioPago.pagar(compra);
        repositorio.guardar(compra);
        notificador.notificar(compra);
    }
}
```

Ahora el diseño permite analizar cada principio por separado:

- responsabilidades más enfocadas → **SRP**;
- nuevos medios de pago sin modificar `ServicioCompra` → **OCP**;
- implementaciones de `MedioPago` deben respetar su contrato → **LSP**;
- si cada interfaz contiene sólo lo que cada cliente necesita → **ISP**;
- la lógica depende de abstracciones → **DIP**.

---

# Tabla mental para estudiar

| Principio | Pregunta que ayuda a detectarlo |
|---|---|
| **SRP** | ¿Esta clase tiene más de una razón conceptualmente distinta para cambiar? |
| **OCP** | ¿Agregar una variante obliga a modificar código estable existente? |
| **LSP** | ¿Un subtipo puede reemplazar realmente al tipo base sin romper expectativas? |
| **ISP** | ¿El cliente está obligado a depender de métodos que no necesita? |
| **DIP** | ¿La lógica importante depende directamente de detalles concretos? |

---

# Señales rápidas en código

## SRP

Buscá clases que mezclen cosas como:

```text
negocio + base de datos + email + PDF + interfaz
```

Pregunta:

> “¿Cuántos motivos distintos hacen cambiar esta clase?”

---

## OCP

Buscá cadenas repetitivas:

```java
if (tipo.equals(...))
else if (...)
else if (...)
```

o:

```java
switch(tipo)
```

que deban modificarse cada vez que aparece una nueva variante.

Pregunta:

> “¿Cada nueva variante obliga a editar este código?”

---

## LSP

Buscá subtipos que:

```java
throw new UnsupportedOperationException();
```

o clientes que deban preguntar:

```java
if (objeto instanceof Subtipo)
```

para poder operar correctamente.

Pregunta:

> “¿Este subtipo cumple realmente el contrato?”

---

## ISP

Buscá interfaces enormes o implementaciones con métodos vacíos:

```java
public void metodoQueNoUso() {
}
```

o:

```java
throw new UnsupportedOperationException();
```

Pregunta:

> “¿Esta clase necesita realmente todo lo que exige esta interfaz?”

---

## DIP

Buscá lógica de negocio que haga:

```java
new BaseDeDatosMySQL()
new EmailSender()
new MercadoPagoSDK()
```

directamente dentro de sus clases.

Pregunta:

> “¿Estoy atando la lógica importante a una tecnología concreta?”

---

# Cómo reconocerlos en un ejercicio

Cuando te den código y te pregunten qué principio se viola, podés recorrer estas preguntas:

1. **¿La clase mezcla varias razones de cambio?**  
   → **SRP**

2. **¿Agregar un nuevo caso obliga a modificar una estructura ya existente?**  
   → **OCP**

3. **¿Una subclase o implementación no puede cumplir lo que promete el tipo base?**  
   → **LSP**

4. **¿Una interfaz obliga a implementar o conocer operaciones innecesarias?**  
   → **ISP**

5. **¿La lógica principal depende directamente de una implementación concreta?**  
   → **DIP**

---

# Idea final

Los cinco principios pueden recordarse así:

> **SRP:** una razón principal de cambio.  
> **OCP:** extender sin modificar constantemente.  
> **LSP:** un subtipo debe ser un sustituto válido.  
> **ISP:** no obligar a depender de lo que no se usa.  
> **DIP:** depender de abstracciones y no de detalles.

Una forma especialmente útil de estudiarlos es no memorizar solamente la definición, sino aprender qué **pregunta de diseño** responde cada principio.

```text
SRP → ¿de cuántas cosas se ocupa?
OCP → ¿qué pasa cuando agrego una variante?
LSP → ¿puedo sustituir un tipo por otro?
ISP → ¿me obligan a depender de cosas que no uso?
DIP → ¿dependo de una abstracción o de un detalle?
```
