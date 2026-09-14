Identificar y justificar todos los _principios de diseño_ que se violan. Proponer e implementar una posible solución usando los patrones vistos en clase. Justificar por qué el patrón elegido resuelve el problema.

1.

```
public class Notificador {
    public Notificacion crearNotificacion(String tipo) {
        if (tipo.equals("email")) {
            return new Email();
        }
        if (tipo.equals("sms")) {
            return new Sms();
        } 
        if (tipo.equals("wh")) {
            return new Wh();
        }
    }
}
```

2.

```
public class TiendaDeMuebles {
    private Silla silla;
    private Sofa sofa;
    private Mesa mesa;

    public TiendaDeMuebles(String tipoEstilo) {
        if (tipoEstilo.equals("moderno")) {
            silla = new SillaModerna();
            sofa = new SofaModerno();
            mesa = new MesaModerna();
        } else if (tipoEstilo.equals("victoriano")) {
            silla = new SillaVictoriana();
            sofa = new SofaVictoriano();
            mesa = new mesaVictoriana();
        } else if (tipoEstilo.equals("artdeco")) {
            silla = new SillaArtDeco();
            sofa = new SofaArtDeco();
            mesa = new MesaArtDeco();
        } 
    }
}
```

3.

```
class PedidoRestaurant {
    private String principal;
    private String entrada;
    private String bebida;
    private String postre;

    public PedidoRestaurant(String entrada, String principal, String bebida, String postre) {
        this.entrada = entrada;
        this.principal = principal;
        this.bebida = bebida;
        this.postre = postre;
    }

    // Código asociado a responsabilidades del pedido de restaurant
}

public class Main {
    public static void main(String[] args) {
        Cocina cocina = new Cocina();

        PedidoRestaurant pedidoNacho = PedidoRestaurant new PedidoRestaurant(null, "Milanesa", "Coca", "Flan");
        PedidoRestaurant pedidoMati = PedidoRestaurant new PedidoRestaurant("Empanada de Carne", "Pastel de Papa", "Agua", null);
        PedidoRestaurant pedidoFede = PedidoRestaurant new PedidoRestaurant(null, "Asado", "Agua saborizada", null);

        cocina.prepararPedido(pedidoNacho);
        cocina.prepararPedido(pedidoMati);
        cocina.prepararPedido(pedidoFede);
    }
}

```

4.

```
public class Bebida {
    private boolean conAzucar;
    private boolean conLeche;
    private boolean conCrema;

    public Bebida(boolean conAzucar, boolean conLeche, boolean conCrema) {
        this.conAzucar = conAzucar;
        this.conLeche = conLeche;
        this.conCrema = conCrema;
    }

    public String descripcion() {
        String desc = "Café";
        if (conAzucar) desc += " con azúcar";
        if (conLeche) desc += " con leche";
        if (conCrema) desc += " con crema";
        return desc;
    }

    public double costo() {
        double precio = 10;
        if (conAzucar) precio += 1;
        if (conLeche) precio += 2;
        if (conCrema) precio += 3;
        return precio;
    }
}

```

5.

```
public class CalculadoraDePrecio {
    public double calcularPrecio(double precioBase, String tipoCliente) {
        if (tipoCliente.equals("regular")) {
            return precioBase;
        } else if (tipoCliente.equals("estudiante")) {
            return precioBase * 0.9;
        } else if (tipoCliente.equals("vip")) {
            return precioBase * 0.8;
        } 
    }
}
```

6.

```
public class ControlRemoto {
    private Luz luz;

    public ControlRemoto(Luz luz) {
        this.luz = luz;
    }

    public void presionarBoton(String boton) {
        if (boton.equals("LUZ_ON")) {
            luz.encender();
        } else if (boton.equals("LUZ_OFF")) {
            luz.apagar();
        }
    }
}

```

7.

```
public class Semaforo {

    private Color colorActual;

    public Semaforo() {
        colorActual = Color.ROJO;
    }

    public void cambiarColor() {
        if (this.colorActual == Color.ROJO) {
            this.colorActual = Color.VERDE;
        } else if (this.colorActual == Color.AMARILLO) {
            this.colorActual = Color.ROJO;
        } else if (this.colorActual == Color.VERDE) {
            this.colorActual = Color.AMARILLO;
        }
    }
}
```