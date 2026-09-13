# Patrones de Diseño Estructurales: Guía Técnica y Arquitectura

## 1. Fundamentos de los Patrones Estructurales y Composición de Objetos

Los **Patrones de Diseño Estructurales** se enfocan en cómo se componen y organizan las clases y objetos para formar estructuras más grandes, complejas y flexibles, manteniendo la eficiencia y el desacoplamiento del sistema. Mientras que los patrones creacionales resuelven los problemas asociados a la instanciación de objetos, los patrones estructurales abordan la relación, empaquetamiento y ensamblado entre módulos ya existentes o por diseñar.

### Categorización por Alcance (Scope)

Dentro del catálogo clásico de patrones de diseño, los patrones estructurales se clasifican según su mecanismo de alcance principal:

1. **Alcance de Clase (Class Scope)**: Utilizan la herencia (múltiple o mediante interfaces) para componer estructuras en tiempo de compilación. El ejemplo principal es el *Class Adapter*.
2. **Alcance de Objeto (Object Scope)**: Utilizan la composición de objetos (delegación y referencias internas) para ensamblar nuevas funcionalidades dinámicamente en tiempo de ejecución. Pertenecen a esta categoría la variante *Object Adapter*, *Bridge*, *Composite*, *Decorator*, *Facade*, *Flyweight* y *Proxy*.

| Patrón | Alcance Principal | Propósito Estructural Clave | Mecanismo de Diseño |
| :--- | :--- | :--- | :--- |
| **Adapter** | Clase / Objeto | Convertir interfaces incompatibles | Traducción y delegación / Herencia |
| **Facade** | Objeto | Simplificar el acceso a un subsistema complejo | Interfaz unificada de alto nivel |
| **Decorator** | Objeto | Añadir responsabilidades dinámicamente | Envoltorio (*Wrapper*) recursivo |
| **Composite** | Objeto | Tratar estructuras jerárquicas hoja/rama de forma uniforme | Estructura en árbol inductiva |
| **Proxy** | Objeto | Controlar y proteger el acceso a un objeto | Intermediario con interfaz idéntica |
| **Flyweight** | Objeto | Minimizar el consumo de memoria mediante estado compartido | División Intrínseco / Extrínseco |
| **Bridge** | Objeto | Desacoplar una abstracción de su implementación | Composición entre jerarquías independientes |

### Principios de Diseño Subyacentes

El diseño de patrones estructurales descansa fuertemente en dos pilares fundamentales del software orientado a objetos:
* **Favorecer la composición de objetos frente a la herencia de clases**: La herencia estática genera rigidez y acoplamiento severo en tiempo de compilación. La composición permite alterar la estructura y el comportamiento del sistema dinámicamente mientras el programa está en ejecución.
* **Separación de Incumbencias (Separation of Concerns - SoC)**: Cada componente estructural tiene un rol bien delimitado (traducir, simplificar, decorar, componer, filtrar o compartir), evitando la proliferación de clases "iceberg" o "monolíticas".

---

## 2. Patrón Adapter (Adaptador)

### Explicación Conceptual
El patrón **Adapter** es un patrón estructural cuya finalidad es permitir que dos módulos o clases con interfaces incompatibles puedan comunicarse sin necesidad de modificar el código fuente de ninguno de los dos.

#### Propósito y Funcionamiento
Cuando un sistema existente necesita integrar una nueva librería de terceros, migrar un componente debido a vulnerabilidades de seguridad, o conectarse a un servicio externo cuya firma de métodos no coincide con lo que el cliente espera, modificar el cliente o la librería externa violaría los principios de diseño de software (como el principio Abierto/Cerrado).

La solución consiste en intercalar una clase intermedia denominada **Adaptador** (*Adapter*). El adaptador implementa la interfaz conocida y esperada por el cliente, guarda internamente una referencia al componente no compatible (**Adaptado** o *Adaptee*) y traduce los llamados entrantes transformando firmas, tipos de datos y parámetros.

#### Variantes de Implementación
1. **Object Adapter (por composición)**: El adaptador implementa la interfaz esperada por el cliente y mantiene una instancia del objeto adaptado como atributo interno. Es el enfoque recomendado en la práctica debido a su flexibilidad y apego al principio de favorecer la composición.
2. **Class Adapter (por herencia)**: El adaptador hereda simultáneamente de la clase esperada por el cliente y de la clase del servicio adaptado (en lenguajes con herencia múltiple o implementando interfaces explícitas). Presenta mayor acoplamiento al depender de la jerarquía de herencia.

### Ejemplo mediante Metáfora
El caso clásico de la vida real es el **adaptador de enchufe de viaje**. Un viajero posee un secador de pelo con una ficha de dos patas redondas (el *Cliente* que requiere esa toma). Al viajar a un país cuyas tomas de pared son de tres patas planas (el *Servicio o Adaptee*), no se rompe la pared para recablear la casa ni se rompe el cable del secador. En su lugar, se conecta un adaptador intermedio que recibe las patas redondas en un extremo y encastra las tres patas planas en la pared.

```
[ Cliente (Secador) ] ---> llama a interfaz conocida ---> [ Adaptador de Viaje ] ---> traduce llamado ---> [ Toma de Pared (Adaptee) ]
```

### Ejemplo Práctico en Java

Supóngase un sistema de comercio electrónico que procesa pagos mediante una interfaz nativa de la empresa (`ProcesadorPago`), pero se requiere integrar una nueva pasarela de pagos externa (`PasarelaMercadoPago`) que utiliza firmas de métodos y modelos de datos completamente distintos.

```java
// Interfaz esperada por el sistema (Cliente)
public interface ProcesadorPago {
    boolean realizarPago(String cuentaId, double montoUSD);
}

// Clase externa no compatible que se desea integrar (Adaptee)
public class PasarelaMercadoPago {
    public int ejecutarTransaccion(long idUsuario, double montoARS, String codigoMoneda) {
        System.out.println("Procesando en MercadoPago para usuario " + idUsuario + ": " + montoARS + " " + codigoMoneda);
        // Retorna 200 si la transacción fue exitosa
        return 200;
    }
}

// Adaptador basado en Composición (Object Adapter)
public class MercadoPagoAdapter implements ProcesadorPago {
    private final PasarelaMercadoPago mercadoPago;
    private static final double TIPO_CAMBIO_USD_ARS = 1000.0;

    public MercadoPagoAdapter(PasarelaMercadoPago mercadoPago) {
        this.mercadoPago = mercadoPago;
    }

    @Override
    public boolean realizarPago(String cuentaId, double montoUSD) {
        // Conversión y adaptación de datos entre ambas interfaces
        long idUsuario = Long.parseLong(cuentaId);
        double montoARS = montoUSD * TIPO_CAMBIO_USD_ARS;
        
        int codigoRespuesta = mercadoPago.ejecutarTransaccion(idUsuario, montoARS, "ARS");
        return codigoRespuesta == 200;
    }
}

// Uso por parte del cliente
public class SistemaVentas {
    public static void main(String[] args) {
        PasarelaMercadoPago servicioExterno = new PasarelaMercadoPago();
        ProcesadorPago procesador = new MercadoPagoAdapter(servicioExterno);
        
        // El cliente opera con la interfaz conocida sin saber que por detrás se usa MercadoPago
        boolean exito = procesador.realizarPago("123456", 50.0);
        System.out.println("Resultado de la operación: " + exito);
    }
}
```

---

## 3. Patrón Facade (Fachada)

### Explicación Conceptual
El patrón **Facade** proporciona una interfaz unificada, simplificada y de alto nivel a un conjunto complejo de interfaces pertenecientes a un subsistema o conjunto de módulos.

#### Propósito y Funcionamiento
En sistemas de gran escala, realizar una operación de negocio suele requerir la interacción coordinada con múltiples clases y servicios independientes (por ejemplo, validar un usuario en la base de datos, descontar stock en inventario, procesar la tarjeta de crédito y generar una orden de envío). Si la clase cliente debe llamar directamente a cada uno de estos subsistemas, queda altamente acoplada a sus estructuras internas y se duplica la lógica de coordinación en múltiples puntos del código.

El patrón Facade resuelve esto creando una clase fachada que actúa como único punto de entrada de alto nivel. La fachada encapsula la interacción compleja entre los subsistemas y expone métodos simples para las operaciones más frecuentes.

#### Diferencia Fundamental entre Facade y Adapter
* **Adapter**: Busca hacer compatibles dos interfaces existentes que no coinciden en sus firmas (traducción uno a uno).
* **Facade**: Busca simplificar el acceso a múltiples interfaces complejas ofreciendo una interfaz de mayor abstracción (coordinación y simplificación).

```
[ Cliente ] 
    |
    v (Llamada simplificada)
[ Fachada de Compras ]
    |
    +---> [ Servicio Inventario ]
    +---> [ Servicio Facturación ]
    +---> [ Servicio Logística ]
```

### Ejemplo mediante Metáfora
El caso de la fachada se visualiza claramente en un **centro de atención telefónica o recepcionista de hotel**. Si un huésped desea organizar una estadía, no llama de forma individual al electricista para revisar las luces de la habitación, al cocinero para encargar el desayuno, al chofer del garaje para pedir el auto y al personal de limpieza para cambiar las sábanas. El huésped habla con un único intermediario simplificado: el **Recepcionista (Fachada)**, quien recibe la solicitud y coordina internamente con todos los departamentos especializados.

### Ejemplo Práctico en Java

Considérese el proceso de alta de usuario e inicialización de perfil en una aplicación, el cual involucra coordinar el almacenamiento en la base de datos, la subida de fotos al hosting y la notificación por correo electrónico.

```java
// Subsistema 1: Base de Datos
public class BaseDeDatosService {
    public void crearRegistroUsuario(String email, String nombre) {
        System.out.println("Guardando usuario en BD: " + email);
    }
}

// Subsistema 2: Hosting de Imágenes
public class AlmacenamientoImagenService {
    public String subirFotoPerfil(String email, byte[] bytesImagen) {
        System.out.println("Subiendo foto de perfil a la nube para: " + email);
        return "https://cdn.app.com/avatars/" + email + ".png";
    }
}

// Subsistema 3: Notificaciones
public class EmailNotificationService {
    public void enviarBienvenida(String email, String urlFoto) {
        System.out.println("Enviando mail de bienvenida a " + email + " con avatar " + urlFoto);
    }
}

// Fachada (Facade)
public class UsuarioRegistrationFacade {
    private final BaseDeDatosService bdService;
    private final AlmacenamientoImagenService imagenService;
    private final EmailNotificationService emailService;

    public UsuarioRegistrationFacade() {
        this.bdService = new BaseDeDatosService();
        this.imagenService = new AlmacenamientoImagenService();
        this.emailService = new EmailNotificationService();
    }

    // Método simplificado de alto nivel
    public void registrarUsuarioCompleto(String nombre, String email, byte[] foto) {
        bdService.crearRegistroUsuario(email, nombre);
        String urlFoto = imagenService.subirFotoPerfil(email, foto);
        emailService.enviarBienvenida(email, urlFoto);
        System.out.println("Registro completado con éxito mediante la Fachada.");
    }
}

// Cliente
public class Aplicacion {
    public static void main(String[] args) {
        UsuarioRegistrationFacade fachada = new UsuarioRegistrationFacade();
        // El cliente realiza el proceso complejo con una sola línea de código
        fachada.registrarUsuarioCompleto("Carlos Gómez", "carlos@example.com", new byte[]{0, 1, 0});
    }
}
```

---

## 4. Patrón Decorator (Decorador)

### Explicación Conceptual
El patrón **Decorator** permite añadir responsabilidades y comportamientos adicionales a un objeto de forma dinámica y transparente en tiempo de ejecución, sin modificar la clase original ni recurrir a una multiplicación explosiva de subclases mediante herencia.

#### Propósito y Funcionamiento
Si se intentara añadir funcionalidades opcionales o combinables a un objeto mediante herencia convencional, el número de clases crecería exponencialmente. Por ejemplo, si se tiene un componente visual de texto y se le quiere agregar borde, barra de desplazamiento (*scroll*), sombra o fondo de color, heredar para cada combinación requeriría clases como `TextoConBorde`, `TextoConScroll`, `TextoConBordeYScroll`, etc. Esto es rígido e ineficiente.

El patrón Decorator propone envolver (*wrapper*) al objeto original dentro de un objeto decorador. La estructura clave es:
1. Una interfaz o clase abstracta común (`Component`) definida para el objeto base y los decoradores.
2. Un componente concreto (`ConcreteComponent`) que contiene la funcionalidad básica primaria.
3. Una clase decoradora base (`BaseDecorator`) que implementa la interfaz `Component` y contiene una referencia a otro objeto `Component`.
4. Decoradores concretos (`ConcreteDecorator`) que heredan del decorador base, delegan el llamado al componente envuelto (`super.execute()`) y añaden su comportamiento específico antes o después de la delegación.

Dado que cada decorador implementa la misma interfaz que el componente que envuelve, se pueden anidar múltiples decoradores en capas en tiempo de ejecución de manera imperceptible para el cliente.

```
[ Cliente ] ---> llama a execute() ---> [ Decorador Borde ]
                                              |
                                              v super.execute()
                                        [ Decorador Scroll ]
                                              |
                                              v super.execute()
                                        [ Componente TextoConcreto ]
```

### Ejemplo mediante Metáfora
Una analogía directa es la **vestimenta por capas de una persona ante el clima frío o lluvioso**. En lugar de alterar quirúrgicamente a la persona o definir tipos de personas como "PersonaConSweater" o "PersonaConCamperaYCapucha", se toma a la **Persona** (el componente base) y se la envuelve dinámicamente:
- Se la decora con un **Sweater** (agrega la capacidad de resistir el frío).
- Sobre el sweater, se la decora con una **Campera Impermeable** (agrega la capacidad de resistir la lluvia).

Cualquier observador externo (el cliente) interactúa con la persona decorada haciendo llamados simples (como "caminar bajo la lluvia"), y la combinación de capas responderá coordinadamente en cadena.

### Ejemplo Práctico en Java

Visualización de un componente gráfico de ventana donde un texto básico (`TextView`) se puede decorar dinámicamente con bordes de color y barras de desplazamiento.

```java
// Interfaz Componente
public interface ComponenteVisual {
    void dibujar();
}

// Componente Concreto
public class TextView implements ComponenteVisual {
    private final String texto;

    public TextView(String texto) {
        this.texto = texto;
    }

    @Override
    public void dibujar() {
        System.out.println("Renderizando texto: "" + texto + """);
    }
}

// Decorador Base Abstracto
public abstract class DecoradorVisual implements ComponenteVisual {
    protected final ComponenteVisual componenteEnvuelto;

    public DecoradorVisual(ComponenteVisual componente) {
        this.componenteEnvuelto = componente;
    }

    @Override
    public void dibujar() {
        // Delega la ejecución al componente envuelto
        componenteEnvuelto.dibujar();
    }
}

// Decorador Concreto 1: Borde
public class BorderDecorator extends DecoradorVisual {
    private final String colorBorde;

    public BorderDecorator(ComponenteVisual componente, String colorBorde) {
        super(componente);
        this.colorBorde = colorBorde;
    }

    @Override
    public void dibujar() {
        super.dibujar(); // Dibuja el elemento interno
        dibujarBorde();  // Agrega el comportamiento propio
    }

    private void dibujarBorde() {
        System.out.println(" -> Agregando borde de color " + colorBorde);
    }
}

// Decorador Concreto 2: Scrollbar
public class ScrollDecorator extends DecoradorVisual {

    public ScrollDecorator(ComponenteVisual componente) {
        super(componente);
    }

    @Override
    public void dibujar() {
        super.dibujar();
        dibujarBarraDesplazamiento();
    }

    private void dibujarBarraDesplazamiento() {
        System.out.println(" -> Agregando barra de desplazamiento vertical");
    }
}

// Cliente
public class DemostradorDecorator {
    public static void main(String[] args) {
        // 1. Texto simple
        ComponenteVisual textoSimple = new TextView("Hola Mundo");
        
        // 2. Texto decorado dinámicamente con Borde y Scroll
        ComponenteVisual textoDecorado = new ScrollDecorator(
                                            new BorderDecorator(textoSimple, "Rojo")
                                         );

        System.out.println("--- Dibujando componente decorado ---");
        // Se ejecuta la cadena: ScrollDecorator -> BorderDecorator -> TextView
        textoDecorado.dibujar();
    }
}
```

---

## 5. Patrón Composite (Objeto Compuesto)

### Explicación Conceptual
El patrón **Composite** permite componer objetos en estructuras arborescentes para representar jerarquías del tipo **parte-todo**. Su propósito principal es permitir que los clientes traten a los objetos individuales y a las composiciones de objetos de manera uniforme a través de una misma interfaz común.

#### Propósito y Funcionamiento
En dominios donde existen elementos simples (hojas) y contenedores agrupadores que pueden albergar tanto elementos simples como otros contenedores (ramas o nodos intermedios), la programación tradicional solía llenar el código de condicionales `if (objeto instanceof Carpeta)` o `if (objeto instanceof Archivo)`. Esto destruye el polimorfismo y complica la mantenibilidad.

El patrón Composite resuelve este dilema definiendo una interfaz común (`Component`) que declara las operaciones operativas (por ejemplo, `calcularTamaño()`, `dibujar()` o `ejecutar()`).
- La clase **Hoja** (*Leaf*) implementa la interfaz y ejecuta el trabajo directo sobre el elemento indivisible.
- La clase **Composite** implementa la misma interfaz, pero además sostiene una colección de hijos de tipo `Component`. Al recibir un llamado operativo, la clase Composite itera inductivamente sobre su lista de hijos y retransmite la petición a cada uno, logrando que el árbol completo procese la solicitud de forma transparente.

```
                  [ Component (Interfaz) ]
                            ^
              +-------------+-------------+
              |                           |
        [ Leaf (Hoja) ]         [ Composite (Nodo) ]
                                 - children: List<Component>
```

### Ejemplo mediante Metáfora
Un ejemplo claro es la **estructura del sistema de archivos de una computadora**.
- Un **Archivo** (imagen, documento de texto) es una **Hoja**: tiene un tamaño específico en MB y no puede contener nada dentro.
- Una **Carpeta** es un **Composite**: tiene un nombre, pero internamente contiene una lista de `Componentes` (que pueden ser Archivos u otras Carpetas anidadas).

Cuando el usuario hace clic derecho en la Carpeta Raíz y pide "Calcular Tamaño Total", no necesita saber cuántas subcarpetas hay adentro. La carpeta raíz le pide el tamaño a cada elemento que contiene; las subcarpetas a su vez se lo piden a sus hijos, hasta llegar a las hojas. El cliente interactúa con la raíz exactamente de la misma forma que interactuaría con un único archivo.

### Ejemplo Práctico en Java

Modelado de un sistema de renderizado gráfico de figuras vectoriales donde elementos individuales (líneas, rectángulos) se pueden agrupar en gráficos compuestos (*Pictures*), los cuales pueden a su vez formar agrupaciones superiores.

```java
import java.util.ArrayList;
import java.util.List;

// Interfaz Componente
public interface Grafico {
    void dibujar();
}

// Hoja 1: Línea
public class Linea implements Grafico {
    @Override
    public void dibujar() {
        System.out.println("Dibujando una Línea");
    }
}

// Hoja 2: Rectángulo
public class Rectangulo implements Grafico {
    @Override
    public void dibujar() {
        System.out.println("Dibujando un Rectángulo");
    }
}

// Composite: Agrupador de gráficos
public class GraficoCompuesto implements Grafico {
    private final List<Grafico> hijos = new ArrayList<>();

    public void agregar(Grafico grafico) {
        hijos.add(grafico);
    }

    public void remover(Grafico grafico) {
        hijos.remove(grafico);
    }

    @Override
    public void dibujar() {
        System.out.println("== Iniciando Grupo de Gráficos ==");
        for (Grafico hijo : hijos) {
            hijo.dibujar(); // Delegación inductiva a cada hijo (Hoja o Composite)
        }
        System.out.println("== Fin Grupo de Gráficos ==");
    }
}

// Cliente
public class EditorGrafico {
    public static void main(String[] args) {
        // Hojas simples
        Grafico l1 = new Linea();
        Grafico r1 = new Rectangulo();

        // Primer nivel de composición
        GraficoCompuesto grupo1 = new GraficoCompuesto();
        grupo1.agregar(l1);
        grupo1.agregar(r1);

        // Segundo nivel de composición (Jerarquía arborescente)
        GraficoCompuesto canvasRaiz = new GraficoCompuesto();
        canvasRaiz.agregar(grupo1);
        canvasRaiz.agregar(new Linea());

        // El cliente dibuja todo el árbol unificadamente
        canvasRaiz.dibujar();
    }
}
```

---

## 6. Patrón Proxy (Apoderado / Intermediario)

### Explicación Conceptual
El patrón **Proxy** proporciona un sustituto, intermediario o marcador de posición para otro objeto, con el propósito de controlar, filtrar, demorar o proteger el acceso a él.

#### Propósito y Funcionamiento
Existen múltiples escenarios donde acceder directamente a un objeto real no es deseable o resulta ineficiente:
- **Proxy de Protección (Protection Proxy)**: Controla permisos de acceso y seguridad antes de permitir la ejecución.
- **Proxy Virtual (Lazy Loading)**: Retarda la instanciación de un objeto costoso (como una imagen en alta resolución o una conexión pesada) hasta que sea estrictamente necesario.
- **Proxy Remoto**: Representa a un objeto que reside en otro espacio de dirección o servidor (como en servicios RPC o cliente/servidor).
- **Proxy de Caché**: Almacena temporalmente los resultados de operaciones costosas para evitar llamados repetitivos al servicio real.

Para lograr la transparencia, el Proxy debe **implementar exactamente la misma interfaz** que el servicio real que protege. De este modo, la clase cliente no puede ni necesita distinguir si está interactuando con el objeto real o con su apoderado.

#### Diferencia Fundamental entre Proxy y Decorator
* **Decorator**: Modifica o agrega nuevas funcionalidades al comportamiento base del objeto envuelto. El objeto decorado altera el resultado final.
* **Proxy**: No agrega ni altera la funcionalidad de negocio del servicio real; únicamente gestiona, controla y protege el ciclo de vida o las condiciones de acceso.

```
[ Cliente ] ---> llama a operacion() ---> [ Proxy ] 
                                             | (Verifica acceso / permisos / caché)
                                             v
                                   [ Servicio Real (Base de Datos) ]
```

### Ejemplo mediante Metáfora
El patrón Proxy se ejemplifica con la función de un **Guardia de Seguridad o Asistente Ejecutivo**. En una corporación, un cliente o visitante no ingresa directamente a la oficina del Director General (*Servicio Real*). En su lugar, debe interactuar con el **Asistente (Proxy)** en la recepción. El asistente verifica si el visitante tiene turno agendado y credenciales de acceso válidas. Si la verificación es positiva, el asistente abre la puerta y permite la reunión con el Director; si es negativa, rechaza la solicitud sin interferir en el trabajo del Director.

### Ejemplo Práctico en Java

Implementación de un Proxy de Protección y Caché para controlar la ejecución de consultas intensivas a un repositorio de base de datos.

```java
import java.util.HashMap;
import java.util.Map;

// Interfaz del Servicio
public interface ServicioConsultas {
    String consultarDatos(String usuario, String query);
}

// Servicio Real (Objeto protegido costoso)
public class ServicioBaseDatosReal implements ServicioConsultas {
    @Override
    public String consultarDatos(String usuario, String query) {
        System.out.println("Ejecutando SQL costoso en la BD para la consulta: " + query);
        return "Resultado de [" + query + "]";
    }
}

// Proxy de Protección y Caché
public class ServicioConsultasProxy implements ServicioConsultas {
    private final ServicioBaseDatosReal servicioReal;
    private final Map<String, String> cacheResultados;

    public ServicioConsultasProxy() {
        this.servicioReal = new ServicioBaseDatosReal();
        this.cacheResultados = new HashMap<>();
    }

    @Override
    public String consultarDatos(String usuario, String query) {
        // 1. Control de Seguridad (Protection Proxy)
        if (!esUsuarioAutorizado(usuario)) {
            System.out.println("ACCESO DENEGADO: El usuario " + usuario + " no tiene permisos.");
            return "Error 403: No Autorizado";
        }

        // 2. Control de Caché (Caching Proxy)
        if (cacheResultados.containsKey(query)) {
            System.out.println("Retornando resultado desde la CACHÉ para: " + query);
            return cacheResultados.get(query);
        }

        // 3. Delegación al Servicio Real si supera los controles
        String resultado = servicioReal.consultarDatos(usuario, query);
        cacheResultados.put(query, resultado);
        return resultado;
    }

    private boolean esUsuarioAutorizado(String usuario) {
        return "admin".equalsIgnoreCase(usuario) || "analista".equalsIgnoreCase(usuario);
    }
}

// Cliente
public class ConsolaSistema {
    public static void main(String[] args) {
        ServicioConsultas servicio = new ServicioConsultasProxy();

        // Intento 1: Usuario no autorizado
        servicio.consultarDatos("invitado", "SELECT * FROM ventas");

        // Intento 2: Usuario autorizado (Ejecuta en BD real)
        servicio.consultarDatos("admin", "SELECT * FROM ventas");

        // Intento 3: Misma consulta de nuevo (Resuelve por Caché sin ir a la BD)
        servicio.consultarDatos("admin", "SELECT * FROM ventas");
    }
}
```

---

## 7. Patrón Flyweight (Peso Ligero)

### Explicación Conceptual
El patrón **Flyweight** es un patrón de diseño enfocado en la optimización del rendimiento y el consumo de memoria RAM. Su propósito es permitir el uso de una gran cantidad de objetos compartiendo eficientemente las partes comunes de su estado en lugar de mantener todos los datos en cada objeto individual.

#### Estado Intrínseco vs. Estado Extrínseco
Para aplicar Flyweight, es imprescindible separar el estado de los objetos en dos categorías:

1. **Estado Intrínseco**: Datos constantes, inmutables y pesados que son compartidos de manera idéntica entre muchas instancias (por ejemplo, mallas 3D, texturas, tipografías, códigos de color). Este estado se almacena de forma única dentro del objeto **Flyweight**.
2. **Estado Extrínseco**: Datos variables y livianos que dependen del contexto de cada objeto en particular (por ejemplo, coordenadas X e Y, orientación, velocidad). Este estado NO se guarda en el Flyweight, sino que es administrado por el cliente y se pasa por parámetro al Flyweight cuando se invoca una operación.

Una fábrica de Flyweights (**FlyweightFactory**) gestiona una piscina (*pool*) o caché de objetos Flyweight creados, garantizando que no se instancien objetos repetidos para estados intrínsecos idénticos.

```
                                  [ FlyweightFactory ]
                                           |
                                           v (Busca o crea)
[ Contexto 1 (Posición X1, Y1) ] ---> [ Flyweight Compartido ] <--- [ Contexto 2 (Posición X2, Y2) ]
                                      (Textura, Modelo 3D)
```

### Ejemplo mediante Metáfora
El patrón Flyweight se comprende intuitivamente observando la **impresión mediante tipos móviles (imprenta clásica) o la representación de caracteres en un procesador de texto**. Si un libro contiene 500.000 letras 'a', la imprenta no fabrica 500.000 piezas metálicas de la letra 'a'. Fabrica un conjunto pequeño de moldes físicos de la letra 'a' (**Estado Intrínseco** compartido). La página y la posición (fila 10, columna 4) donde se estampa cada letra representan el **Estado Extrínseco**, el cual cambia constantemente sin duplicar el molde metálico en memoria.

### Ejemplo Práctico en Java

Simulación de un bosque virtual en un videojuego que debe renderizar 1.000.000 de árboles minimizando el impacto en la memoria RAM.

```java
import java.util.HashMap;
import java.util.Map;

// Objeto Flyweight (Estado Intrínseco Inmutable)
public class TipoArbol {
    private final String nombre;
    private final String color;
    private final String textura3D; // Atributo pesado simulación

    public TipoArbol(String nombre, String color, String textura3D) {
        this.nombre = nombre;
        this.color = color;
        this.textura3D = textura3D;
    }

    public void renderizar(int x, int y) {
        // Recibe el estado extrínseco (x, y) por parámetro
        System.out.println("Dibujando " + nombre + " [" + color + "] en coordenadas (" + x + ", " + y + ")");
    }
}

// Fábrica Flyweight
public class ArbolFactory {
    private static final Map<String, TipoArbol> tiposArboles = new HashMap<>();

    public static TipoArbol getTipoArbol(String nombre, String color, String textura3D) {
        String clave = nombre + "-" + color;
        if (!tiposArboles.containsKey(clave)) {
            tiposArboles.put(clave, new TipoArbol(nombre, color, textura3D));
            System.out.println(">>> Creando NUEVO TipoArbol Flyweight en memoria: " + clave);
        }
        return tiposArboles.get(clave);
    }
}

// Contexto (Almacena el estado extrínseco y la referencia al Flyweight)
public class Arbol {
    private final int x;
    private final int y;
    private final TipoArbol tipo;

    public Arbol(int x, int y, TipoArbol tipo) {
        this.x = x;
        this.y = y;
        this.tipo = tipo;
    }

    public void dibujar() {
        tipo.renderizar(x, y);
    }
}

// Cliente / Mundo
public class Bosque {
    public static void main(String[] args) {
        // Se plantan 4 árboles en distintas posiciones
        // Pero solo se creará 1 objeto TipoArbol de Pino y 1 objeto TipoArbol de Roble
        Arbol a1 = new Arbol(10, 20, ArbolFactory.getTipoArbol("Pino", "Verde Oscuro", "TexturaPino.png"));
        Arbol a2 = new Arbol(15, 25, ArbolFactory.getTipoArbol("Pino", "Verde Oscuro", "TexturaPino.png"));
        Arbol a3 = new Arbol(50, 80, ArbolFactory.getTipoArbol("Roble", "Marrón/Verde", "TexturaRoble.png"));
        Arbol a4 = new Arbol(55, 85, ArbolFactory.getTipoArbol("Pino", "Verde Oscuro", "TexturaPino.png"));

        System.out.println("--- Renderizando Bosque ---");
        a1.dibujar();
        a2.dibujar();
        a3.dibujar();
        a4.dibujar();
    }
}
```

---

## 8. Patrón Bridge (Puente)

### Explicación Conceptual
El patrón **Bridge** busca desacoplar una abstracción de su implementación de manera que ambas puedan evolucionar y variar de forma completamente independiente.

#### Propósito y Funcionamiento
Cuando una clase posee múltiples variaciones ortogonales (por ejemplo, diferentes tipos de formas geométricas como *Círculo* y *Cuadrado*, y diferentes plataformas de renderizado como *Vectorial* y *Rasterizada*), la herencia pura llevaría a una multiplicación cartesiana de clases (`CirculoVectorial`, `CirculoRasterizado`, `CuadradoVectorial`, `CuadradoRasterizado`).

El patrón Bridge resuelve esta rigidez dividiendo la estructura en dos jerarquías separadas vinculadas por composición:
1. **Abstracción**: Define la interfaz de control de alto nivel que utiliza el cliente. Guarda una referencia interna a la interfaz de implementación.
2. **Implementación**: Declara la interfaz para todas las clases de bajo nivel o plataformas concretas.

La Abstracción delega el trabajo operativo a su objeto de Implementación asignado, actuando la referencia como el **puente** entre ambas jerarquías.

```
   [ Jerarquía Abstracción ]                [ Jerarquía Implementación ]
          Abstraccion -------------------------> InterfazImplementacion
              ^                                         ^
              |                                  +------+------+
     AbstraccionRefinada                         |             |
                                            ImplConcretaA ImplConcretaB
```

### Ejemplo mediante Metáfora
Un ejemplo cotidiano es el **control remoto y el televisor**. El **Control Remoto** es la *Abstracción* (posee botones de alto nivel como "encender", "bajar volumen", "cambiar canal"). El **Televisor** (marca Sony, Samsung o LG) es la *Implementación* (posee los circuitos y componentes electrónicos de bajo nivel).

El control remoto no necesita saber cómo la electrónica del televisor convierte las microondas en imagen; simplemente contiene un "puente" (la señal de infrarrojo) que envía comandos a la interfaz del televisor. Se puede crear un nuevo modelo de control remoto (por ejemplo, con comandos de voz) o comprar un nuevo televisor sin romper la compatibilidad entre ambos.

### Ejemplo Práctico en Java

Diseño de una aplicación de dibujo donde las formas geométricas (Abstracción) pueden dibujarse utilizando diferentes motores gráficos (Implementación).

```java
// Interfaz Implementación (Bajo Nivel)
public interface MotorRenderizado {
    void renderizarCirculo(double x, double y, double radio);
}

// Implementación Concreta 1: Gráficos Vectoriales
public class MotorVectorial implements MotorRenderizado {
    @Override
    public void renderizarCirculo(double x, double y, double radio) {
        System.out.println("Dibujando Círculo VECTORIAL en (" + x + ", " + y + ") con radio " + radio);
    }
}

// Implementación Concreta 2: Gráficos Rasterizados (Píxeles)
public class MotorRasterizado implements MotorRenderizado {
    @Override
    public void renderizarCirculo(double x, double y, double radio) {
        System.out.println("Dibujando Círculo RASTERIZADO (Matriz de Píxeles) en (" + x + ", " + y + ")");
    }
}

// Abstracción (Alto Nivel)
public abstract class Forma {
    protected final MotorRenderizado motor; // El "Puente" hacia la implementación

    protected Forma(MotorRenderizado motor) {
        this.motor = motor;
    }

    public abstract void dibujar();
}

// Abstracción Refinada: Círculo
public class CirculoForma extends Forma {
    private final double x, y, radio;

    public CirculoForma(double x, double y, double radio, MotorRenderizado motor) {
        super(motor);
        this.x = x;
        this.y = y;
        this.radio = radio;
    }

    @Override
    public void dibujar() {
        // La abstracción delega la renderización a la implementación puenteada
        motor.renderizarCirculo(x, y, radio);
    }
}

// Cliente
public class AplicacionDibujo {
    public static void main(String[] args) {
        MotorRenderizado vectorial = new MotorVectorial();
        MotorRenderizado raster = new MotorRasterizado();

        // Forma combinada con motor Vectorial
        Forma circulo1 = new CirculoForma(5.0, 10.0, 2.5, vectorial);
        // Misma Forma combinada con motor Rasterizado
        Forma circulo2 = new CirculoForma(5.0, 10.0, 2.5, raster);

        circulo1.dibujar();
        circulo2.dibujar();
    }
}
```

---

## 9. Cuadro Comparativo Integrador de Patrones Estructurales

Para facilitar la diferenciación y la toma de decisiones de diseño en arquitectura de software, a continuación se presenta una comparativa sintética de las intenciones de los patrones estructurales analizados:

| Patrón | Intención Principal | Problema que Resuelve | Relación con el Cliente |
| :--- | :--- | :--- | :--- |
| **Adapter** | Traducir interfaces | Incompatibilidad de firmas entre dos componentes existentes | El cliente usa la interfaz que conoce; el adaptador traduce al servicio real. |
| **Facade** | Simplificar interfaz | Exceso de complejidad y acoplamiento con múltiples subsistemas | El cliente invoca un método único de alto nivel en la fachada. |
| **Decorator** | Agregar funcionalidad | Extender capacidades dinámicamente sin usar herencia | El cliente trata al decorador igual que al componente base (misma interfaz). |
| **Composite** | Jerarquía de árbol | Tratar colecciones (nodos) y elementos individuales (hojas) por igual | El cliente ejecuta operaciones sin distinguir si trabaja con una hoja o un grupo. |
| **Proxy** | Controlar acceso | Necesidad de seguridad, caché, carga diferida o representación remota | El cliente usa la interfaz exacta del servicio sin saber que pasa por un filtro. |
| **Flyweight** | Compartir memoria | Exceso de consumo de RAM al instanciar millones de objetos similares | El cliente suministra el estado extrínseco variable a la fábrica compartida. |
| **Bridge** | Separar abstracción/impl | Explosión combinatoria de clases al variar control y plataforma | La abstracción contiene una referencia a la interfaz de implementación. |
