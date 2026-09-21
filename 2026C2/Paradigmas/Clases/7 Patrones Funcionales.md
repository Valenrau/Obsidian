# Patrones de Diseño de Comportamiento: Guía Técnica y Arquitectura

## 1. Fundamentos de los Patrones de Comportamiento y Comunicación entre Objetos

Los **Patrones de Diseño de Comportamiento** (o funcionales) se centran en los algoritmos, la asignación de responsabilidades y la comunicación dinámica entre objetos en tiempo de ejecución. Mientras que los patrones creacionales resuelven los problemas de instanciación y los patrones estructurales organizan la composición de clases y objetos, los patrones de comportamiento caracterizan los flujos complejos de control y la distribución de tareas dentro de un sistema orientado a objetos.

### Propósito y Objetivos de Arquitectura

El objetivo principal de esta categoría de patrones es maximizar la mantenibilidad, extensibilidad y flexibilidad del código al:
1. **Encapsular comportamientos variables**: Aislar algoritmos que cambian con frecuencia detrás de interfaces estables.
2. **Reducir el acoplamiento entre emisores y receptores**: Permitir que los objetos interactúen sin necesidad de conocer las clases concretas de sus interlocutores.
3. **Eliminar condicionales masivos**: Reemplazar estructuras de control extensas (`if-else` o `switch` sobre tipos o estados) por polimorfismo y delegación.

| Patrón | Propósito Principal | Mecanismo de Comunicación |
| :--- | :--- | :--- |
| **Strategy** | Encapsular una familia de algoritmos intercambiables | Delegación a un objeto estrategia polimórfico |
| **Command** | Encapsular una petición como un objeto ejecutable | Invocación diferida a través de un objeto comando |
| **Observer** | Notificar cambios de estado a múltiples interesados | Suscripción 1 a N mediante notificaciones automáticas |
| **State** | Alterar el comportamiento de un objeto según su estado interno | Autómata con delegación del comportamiento al objeto estado |
| **Template Method** | Definir el esqueleto de un algoritmo delegando pasos a subclases | Herencia y métodos abstractos o ganchos en la clase base |
| **Chain of Responsibility** | Encadenar manejadores para procesar una solicitud en orden | Lista enlazada de objetos donde cada nodo procesa o delega |
| **Visitor** | Separar operaciones de la estructura de objetos sobre la que operan | Despacho doble (*Double Dispatch*) mediante un método `accept` |

---

## 2. Patrón Strategy (Estrategia)

### Explicación Conceptual
El patrón **Strategy** busca encapsular una familia de algoritmos dentro de objetos separados que comparten una interfaz común, permitiendo que sean completamente intercambiables entre sí. 

El problema clásico que resuelve este patrón es la violación del principio **Open/Closed (OCP)**. Cuando una clase debe realizar un cálculo u operación que varía según múltiples condiciones o configuraciones, es habitual encontrar métodos con largos bloques `if-else` o `switch`. Cada vez que surge un nuevo requerimiento, es necesario modificar el código existente de la clase contenedora, aumentando el riesgo de introducir errores.

Strategy extrae cada variante del algoritmo a una clase concreta independiente. La clase principal, denominada **Contexto**, ya no implementa la lógica del algoritmo por sí misma; en su lugar, mantiene una referencia a una interfaz **Estrategia** y delega la ejecución del trabajo. La elección o asignación de la estrategia concreta suele quedar a cargo de un tercero o **Cliente** (como un orquestador o un módulo de configuración), liberando al contexto de saber qué algoritmo específico está utilizando en un momento dado.

### Ejemplo mediante Metáfora
Imaginemos una aplicación de navegación GPS estilo Google Maps o Waze. Cuando el usuario solicita ir del punto A al punto B, la aplicación necesita calcular la mejor ruta. Sin embargo, el cálculo cambia drásticamente dependiendo del medio de transporte elegido:
* En **Automóvil**, se priorizan las autopistas y el estado del tráfico vehicular.
* En **Transporte Público**, se evalúan los horarios de colectivos y trenes, junto con las combinaciones.
* A **Pie**, se seleccionan veredas, peatonales y caminos más directos sin importar el sentido de las calles.

En lugar de tener un método `calcularRuta()` gigante dentro de la clase `Navegador` repleto de condicionales según el tipo de transporte, el navegador delega esta tarea a un objeto `EstrategiaRuta`. Si el usuario cambia de "Auto" a "Caminando" en la interfaz, el navegador simplemente reemplaza la estrategia asignada por una de ruta peatonal.

### Ejemplo Práctico en Java

```java
// Interfaz común para todas las estrategias de enrutamiento
public interface EstrategiaRuta {
    String construirRuta(String origen, String destino);
}

// Estrategia Concreta 1: Vehículo
public class RutaVehiculo implements EstrategiaRuta {
    @Override
    public String construirRuta(String origen, String destino) {
        return "Ruta por autopista de " + origen + " a " + destino + " (Optimizada para tráfico vehicular).";
    }
}

// Estrategia Concreta 2: Peatonal
public class RutaPeatonal implements EstrategiaRuta {
    @Override
    public String construirRuta(String origen, String destino) {
        return "Ruta peatonal de " + origen + " a " + destino + " (Optimizada para calles peatonales y senderos).";
    }
}

// Estrategia Concreta 3: Transporte Público
public class RutaTransportePublico implements EstrategiaRuta {
    @Override
    public String construirRuta(String origen, String destino) {
        return "Ruta en colectivo/subte de " + origen + " a " + destino + " (Optimizada para transbordos).";
    }
}

// Contexto: Navegador GPS
public class Navegador {
    private EstrategiaRuta estrategiaRuta;

    public void setEstrategiaRuta(EstrategiaRuta estrategiaRuta) {
        this.estrategiaRuta = estrategiaRuta;
    }

    public void navegar(String origen, String destino) {
        if (estrategiaRuta == null) {
            throw new IllegalStateException("No se ha seleccionado ninguna estrategia de ruta.");
        }
        String ruta = estrategiaRuta.construirRuta(origen, destino);
        System.out.println("Navegando: " + ruta);
    }
}

// Uso por el Cliente
public class EjemploStrategy {
    public static void main(String[] args) {
        Navegador navegador = new Navegador();

        // El cliente configura la estrategia deseada
        navegador.setEstrategiaRuta(new RutaVehiculo());
        navegador.navegar("Obelisco", "Aeroparque");

        // Cambio dinámico de estrategia en tiempo de ejecución
        navegador.setEstrategiaRuta(new RutaPeatonal());
        navegarNuevamente(navegador, "Obelisco", "Plaza de Mayo");
    }

    private static void navegarNuevamente(Navegador navegador, String origen, String destino) {
        navegador.navegar(origen, destino);
    }
}
```

---

## 3. Patrón Command (Comando)

### Explicación Conceptual
El patrón **Command** convierte una petición o acción en un objeto independiente que contiene toda la información necesaria para ejecutar dicha acción (el método a llamar, los parámetros y la referencia al objeto receptor si corresponde).

Su propósito principal es desacoplar el objeto que **invoca** la operación (el **Invocador** o *Invoker*) del objeto que sabe cómo **realizarla** (el **Receptor** o *Receiver*). Al transformar las operaciones en objetos de primera clase:
* Se pueden almacenar solicitudes en colas de trabajo (*Job Queues*).
* Se pueden programar o diferir ejecuciones.
* Se pueden implementar fácilmente funcionalidades de deshacer/rehacer (*Undo/Redo* o `Ctrl+Z`) guardando un historial de comandos ejecutados.
* Se pueden componer comandos simples mediante otros patrones (como un *Composite* de comandos).

### Ejemplo mediante Metáfora
Pensémoslo como la configuración de controles en un control remoto universal o en un teclado de computadora para videojuegos. 
Una tecla (por ejemplo, la tecla `W` o un botón del joystick) es el **Invocador**. La tecla no sabe qué lógica de juego debe ejecutarse cuando se presiona; solo sabe que al ser accionada debe llamar al método `execute()` del comando que tiene asignado. 

En la pantalla de configuración del juego, el usuario puede remapear las teclas. La tecla `W` puede tener asignado inicialmente el comando "Mover Hacia Adelante". Si el usuario reconfigura la tecla para asignarle el comando "Saltar", la estructura interna de la clase `Tecla` no cambia en absoluto; simplemente se le asigna un objeto `Comando` distinto.

### Ejemplo Práctico en Java

```java
// Interfaz del Comando
public interface Comando {
    void execute();
}

// Receptor: El sistema de iluminación del juego o casa inteligente
public class Lampara {
    public void encender() {
        System.out.println("La lámpara se ha ENCENDIDO.");
    }

    public void apagar() {
        System.out.println("La lámpara se ha APAGADO.");
    }
}

// Comando Concreto 1: Encender Lámpara
public class ComandoEncenderLampara implements Comando {
    private final Lampara lampara;

    public ComandoEncenderLampara(Lampara lampara) {
        this.lampara = lampara;
    }

    @Override
    public void execute() {
        lampara.encender();
    }
}

// Comando Concreto 2: Apagar Lámpara
public class ComandoApagarLampara implements Comando {
    private final Lampara lampara;

    public ComandoApagarLampara(Lampara lampara) {
        this.lampara = lampara;
    }

    @Override
    public void execute() {
        lampara.apagar();
    }
}

// Invocador: Botón físico o de interfaz gráfica
public class Boton {
    private Comando comando;

    public void setComando(Comando comando) {
        this.comando = comando;
    }

    public void presionar() {
        if (comando != null) {
            comando.execute();
        }
    }
}

// Cliente que configura y ejecuta
public class EjemploCommand {
    public static void main(String[] args) {
        Lampara sala = new Lampara();
        
        Comando encender = new ComandoEncenderLampara(sala);
        Comando apagar = new ComandoApagarLampara(sala);

        Boton botonControl = new Boton();

        // Asignar y presionar para encender
        botonControl.setComando(encender);
        botonControl.presionar();

        // Remapear el mismo botón para apagar
        botonControl.setComando(apagar);
        botonControl.presionar();
    }
}
```

### Comparación entre Strategy y Command
Aunque los diagramas de UML entre Strategy y Command son estructuralmente casi idénticos (un contexto/invocador asociado a una interfaz implementada por clases concretas), sus intencionalidades difieren:
* **Strategy**: Modela *cómo* se hace algo. Representa diferentes formas (familia de algoritmos) de realizar la misma tarea conceptual (ej. calcular ruta).
* **Command**: Modela *qué* se hace. Representa acciones unitarias y heterogéneas que se empaquetan en un objeto para ejecutarse más tarde, encolarse o deshacerse.

---

## 4. Patrón Observer (Observador / Publisher-Subscriber)

### Explicación Conceptual
El patrón **Observer** define una dependencia de uno a muchos (*1-to-N*) entre objetos, de forma que cuando el objeto principal (el **Sujeto** o **Publisher**) cambia su estado, todos los objetos dependientes que se han registrado (los **Observadores** o **Subscribers**) son notificados y actualizados de manera automática.

Este patrón resuelve el problema del **polling** o consulta periódica ineficiente. Sin Observer, si un cliente necesita reaccionar cuando cambia el estado de otro objeto, tendría que estar consultando contínuamente ese estado dentro de un bucle, consumiendo recursos innecesarios. La alternativa opuesta —que el objeto que cambia conozca a todos los clientes concretos a los que debe avisar— rompería la Inversión de Dependencias y el acoplamiento limpio.

Observer aplica Inversión de Dependencias: el Publisher solo mantiene una lista de referencias a la interfaz `Suscriptor`. Cualquier clase que implemente dicha interfaz puede suscribirse o desuscribirse libremente en tiempo de ejecución.

### Ejemplo mediante Metáfora
Consideremos la venta de un producto con stock limitado muy demandado (como una consola de videojuegos PS5 en una tienda). 
* **Pésima solución (Polling)**: Cada cliente va físicamente a la tienda todos los días o llama por teléfono cada media hora para preguntar: "¿Llegó stock?".
* **Pésima solución opuesta**: La tienda guarda el número personal de cada cliente histórico y los llama individualmente uno por uno cuando llega mercadería.
* **Solución con Observer**: La tienda pone un formulario en su sitio web ("Avisarme cuando haya stock"). Los clientes interesados se anotan (**suscripción**). Cuando la tienda recibe stock, dispara una notificación masiva automática a la lista de correo de suscriptores (**notificación**). Cuando un cliente ya compró su consola, hace clic en "Cancelar suscripción" y deja de recibir avisos.

### Ejemplo Práctico en Java

```java
import java.util.ArrayList;
import java.util.List;

// Interfaz del Suscriptor / Observador
public interface Suscriptor {
    void actualizar(String mensajeEstado);
}

// Publisher / Sujeto: Tienda de productos
public class TiendaPublisher {
    private final List<Suscriptor> suscriptores = new ArrayList<>();
    private String estadoStock;

    public void suscribir(Suscriptor s) {
        suscriptores.add(s);
    }

    public void desuscribir(Suscriptor s) {
        suscriptores.remove(s);
    }

    public void notificarSuscriptores() {
        for (Suscriptor s : suscriptores) {
            s.actualizar(estadoStock);
        }
    }

    public void cambiarStock(String nuevoEstado) {
        this.estadoStock = nuevoEstado;
        System.out.println("Tienda: El estado del stock ha cambiado a -> " + 
        nuevoEstado);
        notificarSuscriptores();
    }
}

// Suscriptor Concreto 1: Cliente particular
public class ClienteUsuario implements Suscriptor {
    private final String nombre;

    public ClienteUsuario(String nombre) {
        this.nombre = nombre;
    }

    @Override
    public void actualizar(String mensajeEstado) {
        System.out.println("Notificación enviada a " + nombre + ": " + 
        mensajeEstado);
    }
}

// Suscriptor Concreto 2: Sistema de compras minorista
public class SistemaInventarioMinorista implements Suscriptor {
    @Override
    public void actualizar(String mensajeEstado) {
        System.out.println("Sistema Minorista actualizó su panel de 
        disponibilidad: " + mensajeEstado);
    }
}

// Ejemplo de Uso
public class EjemploObserver {
    public static void main(String[] args) {
        TiendaPublisher tienda = new TiendaPublisher();

        ClienteUsuario juan = new ClienteUsuario("Juan Pérez");
        ClienteUsuario maria = new ClienteUsuario("María Gómez");
        SistemaInventarioMinorista sistemaMinorista = new 
        SistemaInventarioMinorista();

        // Registrar suscriptores
        tienda.suscribir(juan);
        tienda.suscribir(maria);
        tienda.suscribir(sistemaMinorista);

        // Ocurre un evento de negocio
        tienda.cambiarStock("Stock disponible: 10 unidades de PS5.");

        // Un cliente decide desuscribirse
        tienda.desuscribir(juan);

        // Siguiente cambio de estado
        tienda.cambiarStock("Aviso: Quedan solo 2 unidades.");
    }
}
```

---

## 5. Patrón State (Estado)

### Explicación Conceptual
El patrón **State** permite que un objeto altere su comportamiento cuando su estado interno cambia. El objeto parecerá haber cambiado su clase.

Este patrón se aplica a sistemas que se comportan como una **máquina de estados finitos (autómata)**. Cuando un objeto puede estar en uno de varios estados y su respuesta a cada mensaje o método depende completamente del estado actual, el código convencional suele recurrir a variables de estado enteras o enumerados y condicionales `if-else` o `switch` en *cada uno* de sus métodos.

State extrae cada estado a una clase separada que implementa una interfaz común **Estado**. La clase principal, el **Contexto**, mantiene una referencia a su estado actual y delega en él todas las operaciones sensibles al estado.

### Transiciones entre Estados
A diferencia de Strategy (donde el cliente cambia externamente la estrategia), en State los propios objetos concretos de estado o el contexto gestionan las transiciones automáticas. Un estado concreto sabe qué evento o método provoca el pasaje hacia el siguiente estado válido en el autómata.

### Ejemplo mediante Metáfora
Consideremos la gestión de un pedido en una plataforma de delivery de comida rápida:
1. **Creado**: Si el usuario intenta cancelar, el pedido se cancela sin costo. Si presiona "Pagar", pasa al estado *En Preparación*.
2. **En Preparación**: La cocina está cocinando. Si el usuario intenta cancelar, se le cobra una penalización. Si la cocina termina, pasa al estado *En Camino*.
3. **En Camino**: El repartidor lleva la comida. La acción de cancelar está estrictamente prohibida. Si llega a destino, pasa a *Entregado*.

La acción o método invocado es siempre el mismo (`cancelar()` o `avanzar()`), pero la respuesta del objeto `Pedido` varía completamente según el estado actual en el autómata.

### Ejemplo Práctico en Java

```java
// Interfaz para los estados del pedido
public interface EstadoPedido {
    void avanzar(PedidoContexto pedido);
    void cancelar(PedidoContexto pedido);
}

// Contexto: El Pedido
public class PedidoContexto {
    private EstadoPedido estadoActual;

    public PedidoContexto() {
        this.estadoActual = new EstadoCreado(); // Estado inicial
    }

    public void setEstado(EstadoPedido nuevoEstado) {
        this.estadoActual = nuevoEstado;
    }

    public void avanzar() {
        estadoActual.avanzar(this);
    }

    public void cancelar() {
        estadoActual.cancelar(this);
    }
}

// Estado Concreto 1: Creado
public class EstadoCreado implements EstadoPedido {
    @Override
    public void avanzar(PedidoContexto pedido) {
        System.out.println("El pedido ha sido pagado. Pasando a cocina...");
        pedido.setEstado(new EstadoEnPreparacion());
    }

    @Override
    public void cancelar(PedidoContexto pedido) {
        System.out.println("Pedido cancelado sin cargos adicionales.");
        pedido.setEstado(new EstadoCancelado());
    }
}

// Estado Concreto 2: En Preparación
public class EstadoEnPreparacion implements EstadoPedido {
    @Override
    public void avanzar(PedidoContexto pedido) {
        System.out.println("La comida está lista. Asignando repartidor...");
        pedido.setEstado(new EstadoEnCamino());
    }

    @Override
    public void cancelar(PedidoContexto pedido) {
        System.out.println("Cancelación en preparación: Se cobra el 50% de penalización por ingredientes.");
        pedido.setEstado(new EstadoCancelado());
    }
}

// Estado Concreto 3: En Camino
public class EstadoEnCamino implements EstadoPedido {
    @Override
    public void avanzar(PedidoContexto pedido) {
        System.out.println("El pedido fue entregado al cliente con éxito.");
        pedido.setEstado(new EstadoEntregado());
    }

    @Override
    public void cancelar(PedidoContexto pedido) {
        System.out.println("ERROR: No se puede cancelar un pedido que ya está en camino.");
    }
}

// Estado Concreto 4: Cancelado o Entregado (Estados Finales)
public class EstadoCancelado implements EstadoPedido {
    @Override
    public void avanzar(PedidoContexto pedido) {
        System.out.println("El pedido está cancelado. No se puede avanzar.");
    }

    @Override
    public void cancelar(PedidoContexto pedido) {
        System.out.println("El pedido ya se encuentra cancelado.");
    }
}

public class EstadoEntregado implements EstadoPedido {
    @Override
    public void avanzar(PedidoContexto pedido) {
        System.out.println("El pedido ya fue finalizado.");
    }

    @Override
    public void cancelar(PedidoContexto pedido) {
        System.out.println("No se puede cancelar un pedido ya entregado.");
    }
}

// Ejemplo de Uso
public class EjemploState {
    public static void main(String[] args) {
        PedidoContexto miPedido = new PedidoContexto();

        miPedido.avanzar(); // Pasa de Creado a En Preparación
        miPedido.cancelar(); // Cancela con 50% penalización
        miPedido.avanzar(); // No permite avanzar
    }
}
```

---

## 6. Patrón Template Method (Método Plantilla)

### Explicación Conceptual
El patrón **Template Method** define el esqueleto o estructura general de un algoritmo en un método de una clase base (generalmente abstracta), delegando la implementación de ciertos pasos específicos a las subclases derivadas.

Permite que las subclases redefinan ciertos pasos de un algoritmo sin modificar la estructura ni el orden de ejecución general del algoritmo. Este patrón promueve la reutilización de código al colocar la lógica común en la clase padre y forzar a las subclases a completar exclusivamente las partes específicas o variables.

El método que contiene la plantilla se implementa de manera concreta en la clase base y coordina llamadas a:
1. **Pasos concretos**: Métodos con implementación común ya definidos en la clase base.
2. **Pasos abstractos**: Métodos que cada subclase debe implementar obligatoriamente.
3. **Métodos gancho (*Hooks*)**: Métodos opcionales que proveen un comportamiento por defecto en la clase base pero pueden ser sobrescritos por la subclase si se desea alterar el flujo.

### Ejemplo mediante Metáfora
Imaginemos el manual de entrenamiento para los baristas de una cafetería (como el *Starbucks Barista Training Manual*). El procedimiento para preparar infusiones calientes sigue una secuencia invariable de pasos:
1. Hervir agua.
2. Infusionar la sustancia en el agua caliente.
3. Servir el líquido en una taza.
4. Agregar los condimentos requeridos.

El algoritmo es el mismo para preparar **Café** o **Té**. Los pasos 1 y 3 (hervir agua y servir en taza) son idénticos para cualquier bebida caliente. Sin embargo, los pasos 2 y 4 difieren: para el café se destila café molido y se agrega leche o azúcar; para el té se sumerge un saquito de té y se agrega limón.

### Ejemplo Práctico en Java

```java
// Clase base abstracta que define la plantilla
public abstract class PreparadorBebida {

    // Método Plantilla (final para evitar que las subclases alteren la estructura)
    public final void prepararReceta() {
        hervirAgua();
        prepararInfusion(); // Paso abstracto
        servirEnTaza();
        if (clienteDeseaCondimentos()) { // Gancho / Hook
            agregarCondimentos(); // Paso abstracto
        }
    }

    private void hervirAgua() {
        System.out.println("1. Hirviendo agua a 90°C...");
    }

    private void servirEnTaza() {
        System.out.println("3. Sirviendo en la taza...");
    }

    // Métodos abstractos que deben implementar las subclases
    protected abstract void prepararInfusion();
    protected abstract void agregarCondimentos();

    // Hook (Método gancho) con implementación por defecto
    protected boolean clienteDeseaCondimentos() {
        return true;
    }
}

// Subclase Concreta 1: Café
public class PreparadorCafe extends PreparadorBebida {
    @Override
    protected void prepararInfusion() {
        System.out.println("2. Pasando agua caliente por el grano de café molido...");
    }

    @Override
    protected void agregarCondimentos() {
        System.out.println("4. Agregando leche descremada y azúcar...");
    }
}

// Subclase Concreta 2: Té
public class PreparadorTe extends PreparadorBebida {
    @Override
    protected void prepararInfusion() {
        System.out.println("2. Sumergiendo el saquito de té en agua hirviendo...");
    }

    @Override
    protected void agregarCondimentos() {
        System.out.println("4. Agregando una rodaja de limón...");
    }

    @Override
    protected boolean clienteDeseaCondimentos() {
        // Podríamos personalizar la respuesta del gancho
        return true;
    }
}

// Ejemplo de Uso
public class EjemploTemplateMethod {
    public static void main(String[] args) {
        System.out.println("--- Preparando Café ---");
        PreparadorBebida cafe = new PreparadorCafe();
        cafe.prepararReceta();

        System.out.println("\n--- Preparando Té ---");
        PreparadorBebida te = new PreparadorTe();
        te.prepararReceta();
    }
}
```

### Relación entre Template Method y Factory Method
Existe una relación directa de jerarquía conceptual entre ambos patrones: **Factory Method es un caso particular de Template Method**. 
En un Factory Method, el paso abstracto que se difiere a la subclase es específicamente la creación/instanciación de un objeto. En Template Method, los pasos diferidos pueden ser cualquier comportamiento o rutina del algoritmo general.

---

## 7. Patrón Chain of Responsibility (Cadena de Responsabilidad)

### Explicación Conceptual
El patrón **Chain of Responsibility** desacopla al emisor de una solicitud de sus posibles receptores, al dar a más de un objeto la oportunidad de responder a la solicitud. Los objetos receptores se encadenan formando una lista enlazada; la solicitud viaja a lo largo de la cadena hasta que uno de los objetos la procesa o la cadena finaliza.

Cada nodo o **Manejador** (*Handler*) de la cadena posee una referencia al siguiente manejador. Al recibir una petición, el manejador evalúa si cumple las condiciones para resolverla:
* Si la puede resolver, la procesa (y opcionalmente puede detener la propagación o permitir que continúe).
* Si no la puede resolver, delega automáticamente la petición al siguiente manejador de la cadena.

Este patrón evita el acoplamiento directo entre el cliente que emite una orden y la clase concreta que termina atendiéndola, además de permitir reconfigurar el orden jerárquico de filtros en tiempo de ejecución.

### Ejemplo mediante Metáfora
Un pipeline de atención de reclamos o filtrado de correos electrónicos entrantes en un servidor:
Cuando llega un nuevo correo, pasa primero por el manejador de **Spam**. Si el correo es marcado como spam, se elimina y se corta la cadena. Si no es spam, pasa al manejador de **Autenticación y Firma Digital**. Si no supera la firma, se rechaza. Si es válido, pasa al manejador de **Filtro de Contenido/Soporte**.

Cada filtro actúa autónomamente como un nodo de una lista enlazada. El cliente solo envía el mensaje al primer nodo del pipeline sin saber cuál de todos los filtros terminará atendiéndolo o rechazándolo.

### Ejemplo Práctico en Java

```java
// Clase solicitud de correo
public class Correo {
    private final String emisor;
    private final String contenido;
    private final boolean esSpam;

    public Correo(String emisor, String contenido, boolean esSpam) {
        this.emisor = emisor;
        this.contenido = contenido;
        this.esSpam = esSpam;
    }

    public String getEmisor() { return emisor; }
    public String getContenido() { return contenido; }
    public boolean esSpam() { return esSpam; }
}

// Interfaz del Manejador (Handler)
public interface ManejadorCorreo {
    void setSiguiente(ManejadorCorreo siguiente);
    void procesar(Correo correo);
}

// Clase Base Abstracta
public abstract class ManejadorBase implements ManejadorCorreo {
    private ManejadorCorreo siguiente;

    @Override
    public void setSiguiente(ManejadorCorreo siguiente) {
        this.siguiente = siguiente;
    }

    protected void pasarAlSiguiente(Correo correo) {
        if (siguiente != null) {
            siguiente.procesar(correo);
        } else {
            System.out.println("Fin de la cadena: El correo fue procesado por defecto.");
        }
    }
}

// Manejador Concreto 1: Filtro de Spam
public class FiltroSpam extends ManejadorBase {
    @Override
    public void procesar(Correo correo) {
        if (correo.esSpam()) {
            System.out.println("Filtro Spam: Correo de " + correo.getEmisor() + " bloqueado y descartado.");
        } else {
            System.out.println("Filtro Spam: Correo limpio. Derivando al siguiente...");
            pasarAlSiguiente(correo);
        }
    }
}

// Manejador Concreto 2: Reclamos
public class FiltroReclamos extends ManejadorBase {
    @Override
    public void procesar(Correo correo) {
        if (correo.getContenido().toLowerCase().contains("reclamo")) {
            System.out.println("Filtro Reclamos: Correo derivado al área de Atención al Cliente.");
        } else {
            System.out.println("Filtro Reclamos: No es un reclamo. Derivando al siguiente...");
            pasarAlSiguiente(correo);
        }
    }
}

// Ejemplo de Uso
public class EjemploChainOfResponsibility {
    public static void main(String[] args) {
        ManejadorCorreo filtroSpam = new FiltroSpam();
        ManejadorCorreo filtroReclamos = new FiltroReclamos();

        // Construir la cadena / lista enlazada de handlers
        filtroSpam.setSiguiente(filtroReclamos);

        Correo mail1 = new Correo("ofertas@malicioso.com", "Gana dinero ya", true);
        Correo mail2 = new Correo("cliente@usuario.com", "Tengo un reclamo sobre mi factura", false);

        System.out.println("--- Procesando Correo 1 ---");
        filtroSpam.procesar(mail1);

        System.out.println("\n--- Procesando Correo 2 ---");
        filtroSpam.procesar(mail2);
    }
}
```

---

## 8. Patrón Visitor (Visitante) y Double Dispatch

### Explicación Conceptual
El patrón **Visitor** permite definir una nueva operación sobre una estructura de objetos sin cambiar las clases de los elementos sobre los que opera. 

Resuelve el escenario donde se cuenta con una jerarquía de clases heterogéneas que ya están cerradas a modificaciones (por ejemplo, clases de un modelo central de dominio). Si se necesita agregar múltiples operaciones distintas que no forman parte de la responsabilidad primaria de esas clases (como exportar a XML/JSON, auditar, o calcular impuestos/descuentos específicos), agregar estos métodos directamente en cada clase del dominio violaría el principio **Single Responsibility (SRP)** y abriría clases estables a modificaciones innecesarias.

Visitor separa los algoritmos de la estructura de datos. Para lograr que se invoque la operación adecuada en tiempo de ejecución combinando el tipo concreto del elemento y el tipo concreto del visitante, el patrón utiliza el mecanismo de **Despacho Doble (*Double Dispatch*)**.

### El Mecanismo de Double Dispatch
Muchos lenguajes con tipos dinámicos o polimorfismo estándar soportan únicamente *Single Dispatch* (la resolución del método llamado depende únicamente del tipo en tiempo de ejecución del objeto receptor, `this`). En lenguajes como Java, la sobrecarga de métodos se resuelve estáticamente en tiempo de compilación según la firma declarada.

Double Dispatch combina dos llamadas polimórficas consecutivas:
1. El cliente llama a `elemento.accept(visitor)`. Esta primera llamada resuelve polimórficamente el tipo concreto del **Elemento**.
2. Dentro de la implementación de `accept`, el elemento responde invocando `visitor.visit(this)`. Como `this` dentro del elemento conoce su tipo exacto en tiempo de compilación, Java llama al método sobrecargado `visit` específico del **Visitor**, resolviendo el tipo concreto del visitante.

### Ejemplo mediante Metáfora
Imaginemos una inspección o aplicación de beneficios para diferentes categorías de ciudadanos: **Jubilados**, **Estudiantes** y **Docentes**.
Cada tipo de ciudadano tiene reglas totalmente distintas para obtener descuentos en transporte, entradas a museos o bonos trimestrales.
En lugar de agregar métodos `calcularDescuentoTransporte()`, `calcularDescuentoMuseo()`, etc., dentro de las clases de ciudadanos, el objeto ciudadano simplemente acepta a un inspector o liquidador (`visitor`). Cuando el ciudadano dice `aceptar(visitante)`, el visitante aplica la regla exacta que corresponde a ese tipo de ciudadano.

### Ejemplo Práctico en Java

```java
// Interfaces para Elementos y Visitantes

public interface VisitanteDescuento {
    void visitarJubilado(ClienteJubilado jubilado);
    void visitarEstudiante(ClienteEstudiante estudiante);
}

public interface ElementoCliente {
    void aceptar(VisitanteDescuento visitante);
}

// Elemento Concreto 1: Jubilado
public class ClienteJubilado implements ElementoCliente {
    @Override
    public void aceptar(VisitanteDescuento visitante) {
        // Primera llamada polimórfica resuelve el elemento y pasa 'this' con tipo exacto
        visitante.visitarJubilado(this);
    }

    public double getPension() { return 50000; }
}

// Elemento Concreto 2: Estudiante
public class ClienteEstudiante implements ElementoCliente {
    @Override
    public void aceptar(VisitanteDescuento visitante) {
        visitante.visitarEstudiante(this);
    }

    public String getUniversidad() { return "FIUBA"; }
}

// Visitante Concreto 1: Liquidación de Descuento de Transporte
public class DescuentoTransporteVisitor implements VisitanteDescuento {
    @Override
    public void visitarJubilado(ClienteJubilado jubilado) {
        System.out.println("Descuento Transporte Jubilado: 90% de bonificación.");
    }

    @Override
    public void visitarEstudiante(ClienteEstudiante estudiante) {
        System.out.println("Descuento Transporte Estudiante (" + estudiante.getUniversidad() + "): 50% de bonificación.");
    }
}

// Visitante Concreto 2: Entradas a Museos
public class DescuentoMuseoVisitor implements VisitanteDescuento {
    @Override
    public void visitarJubilado(ClienteJubilado jubilado) {
        System.out.println("Acceso a Museos para Jubilados: 100% Gratuito.");
    }

    @Override
    public void visitarEstudiante(ClienteEstudiante estudiante) {
        System.out.println("Acceso a Museos para Estudiantes: 80% de bonificación.");
    }
}

// Ejemplo de Uso
public class EjemploVisitor {
    public static void main(String[] args) {
        ElementoCliente[] clientes = new ElementoCliente[] {
            new ClienteJubilado(),
            new ClienteEstudiante()
        };

        VisitanteDescuento transporte = new DescuentoTransporteVisitor();
        VisitanteDescuento museo = new DescuentoMuseoVisitor();

        System.out.println("--- Aplicando Descuento de Transporte ---");
        for (ElementoCliente c : clientes) {
            c.aceptar(transporte);
        }

        System.out.println("\n--- Aplicando Beneficios de Museo ---");
        for (ElementoCliente c : clientes) {
            c.aceptar(museo);
        }
    }
}
```

---

## 9. Otros Patrones de Comportamiento Relevantes

Además de los patrones principales analizados, existen otros patrones funcionales en el catálogo clásico que resuelven problemas específicos de interacción:

### Iterator (Iterador)
Permite recorrer secuencialmente los elementos de una colección o estructura de datos compleja (listas, árboles, grafos) sin exponer su representación o estructura interna subyacente. La abstracción separa los detalles del recorrido de los objetos contenedores.

### Mediator (Mediador)
Sustituye redes complejas y caóticas de dependencias directas entre múltiples objetos por un objeto mediador centralizado. En lugar de que decenas de componentes conozcan y llamen directamente a otros componentes (generando un grafo denso de dependencias), todos se comunican exclusivamente a través del mediador.

### Memento (Recordatorio)
Permite capturar y exteriorizar el estado interno de un objeto en un momento dado, sin violar su encapsulamiento, de modo que el objeto pueda volver a ese estado más tarde. Es la base técnica para la implementación de instantáneas (*snapshots*) y operaciones de restauración.

### Null Object (Objeto Nulo)
Proporciona un objeto con comportamiento neutro o nulo (*no-operation*) en lugar de utilizar una referencia `null`. Evita la necesidad de llenar el código de comprobaciones defensivas `if (objeto != null)` antes de invocar métodos, promoviendo la ejecución limpia del polimorfismo.

---

## 10. Cuadro Comparativo Integrador

| Patrón | Problema Clave Resuelto | Solución de Arquitectura | Cambio Principal en el Sistema |
| :--- | :--- | :--- | :--- |
| **Strategy** | Violación de OCP por condicionales múltiples de un algoritmo | Familia de clases de algoritmos detrás de una interfaz | Se encapsula *cómo* se realiza un cálculo intercambiable |
| **Command** | Acoplamiento entre el invocador de una acción y el receptor | Petición empaquetada dentro de un objeto `execute()` | Se parametriza, encola o deshace *qué* tarea se ejecuta |
| **Observer** | Polling ineficiente o acoplamiento directo en notificaciones 1 a N | Registro dinámico de suscriptores e Inversión de Dependencias | El sujeto avisa a los interesados automáticamente sin conocerlos |
| **State** | Comportamiento que depende de un autómata de estados finitos | Objetos de estado que contienen la lógica y gestionan transiciones | El objeto delega su comportamiento según su estado actual |
| **Template Method** | Duplicación de código en la estructura general de algoritmos | Esqueleto de algoritmo en clase base abstracta con pasos diferidos | Las subclases redefinen pasos específicos sin cambiar la estructura |
| **Chain of Responsibility** | Un receptor de una solicitud debe elegirse dinámicamente | Lista enlazada de manejadores independientes | La petición fluye hasta que un nodo decide atenderla o rechazarla |
| **Visitor** | Añadir operaciones a una jerarquía estable sin modificar clases | Despacho doble (*Double Dispatch*) separando algoritmo de estructura | Las operaciones se mueven fuera de las clases de dominio |
