## ¿Qué es una system call?

Un programa normal corre en **modo usuario**: no puede tocar directamente el hardware (disco, memoria física, red, otros procesos) porque si pudiera, cualquier programa podría romper todo el sistema o espiar a otros procesos.

Una **system call** (llamada al sistema) es la única puerta de entrada que tiene un programa para pedirle al **kernel** que haga algo en su nombre: abrir un archivo, crear un proceso, escribir en pantalla, reservar memoria, etc.

```
 MODO USUARIO                    MODO KERNEL
┌────────────────┐   syscall    ┌─────────────────────┐
│  tu programa   │ ───────────▶ │  kernel del SO      │
│  (read, fork,  │              │  (maneja hardware,  │
│   write...)    │ ◀─────────── │  procesos, memoria) │
└────────────────┘   resultado  └─────────────────────┘
```

**Cómo funciona por dentro (simplificado):**

1. Tu programa llama a una función de librería, por ejemplo `read()`.
2. Esa función ejecuta una instrucción especial del procesador (una _trap_ o interrupción de software) que cambia el CPU de modo usuario a **modo kernel**.
3. El kernel identifica qué syscall se pidió (hay una tabla de syscalls, cada una con un número) y ejecuta el código correspondiente, ya con permisos totales.
4. Cuando termina, el kernel devuelve el control al proceso, se vuelve a modo usuario, y el programa sigue como si hubiera sido "una función más" — pero por dentro cruzó al kernel y volvió.

Es el kernel el que decide **qué se puede hacer y cómo**, protegiendo al sistema de que un proceso haga cualquier cosa directamente.

Ejemplos típicos de syscalls en Unix: `read`, `write`, `open`, `close`, `fork`, `exec`, `wait`, `pipe`, `kill`, `exit`.

---

## File descriptors (fd)

Un **file descriptor** es un número entero que el kernel le da a un proceso para referirse a "algo abierto": un archivo, una pipe, un socket, la terminal, etc. El proceso nunca maneja el archivo directamente, siempre lo referencia por ese número.

Cada proceso tiene su propia **tabla de file descriptors**, que apunta a una tabla más global del kernel con la info real (posición actual, modo de apertura, etc.).

Por convención, todo proceso arranca con 3 fd ya abiertos:

| fd  | nombre | uso habitual               |
| --- | ------ | -------------------------- |
| 0   | stdin  | entrada estándar (teclado) |
| 1   | stdout | salida estándar (pantalla) |
| 2   | stderr | salida de errores          |

```
Proceso                     Kernel
┌───────────────────┐        ┌─────────────────────────────┐
│ tabla de fd:      │        │ tabla de archivos abiertos  │
│  0 → stdin  ──────┼───────▶│ [terminal]                  │
│  1 → stdout ──────┼───────▶│ [terminal]                  │
│  2 → stderr ──────┼───────▶│ [terminal]                  │
│  3 → archivo.txt ─┼───────▶│ [inode archivo.txt, offset] │
└───────────────────┘        └─────────────────────────────┘
```

Ejemplo en C:

```c
#include <fcntl.h>
#include <unistd.h>

int main() {
    int fd = open("archivo.txt", O_RDONLY); // el kernel devuelve un entero, ej: 3
    char buffer[100];
    read(fd, buffer, 100);   // leo usando el número, no el nombre
    close(fd);               // libero el fd
    return 0;
}
```

---

## Procesos y `fork()`

Un **proceso** es un programa en ejecución: tiene su propio espacio de memoria, sus propios registros, su propia tabla de file descriptors, y un identificador único llamado **PID** (Process ID).

`fork()` es la syscall que crea un proceso nuevo **clonando** al proceso que la llama. El proceso original se llama **padre**, el nuevo se llama **hijo**.

### ¿Qué hace fork() exactamente?

- Crea una copia casi idéntica del proceso: mismo código, mismas variables (con los mismos valores en ese instante), misma tabla de file descriptors (abiertos y apuntando a lo mismo).
- A partir de ese punto, **padre e hijo son procesos totalmente independientes**, cada uno con su propia memoria (aunque al principio tenga los mismos valores). Si uno modifica una variable, el otro no se entera.
- Ambos procesos siguen ejecutando **desde la instrucción siguiente al fork()**, no desde el principio del programa.

### ¿Qué devuelve fork()?

Acá está el detalle clave: `fork()` se llama **una vez** pero **devuelve dos veces** (una en cada proceso), y devuelve valores distintos:

| Quién ejecuta     | Valor que recibe de fork()               |
| ----------------- | ---------------------------------------- |
| Proceso **padre** | el **PID del hijo** (un número positivo) |
| Proceso **hijo**  | **0**                                    |
| Si falla          | **-1** (no se creó ningún hijo)          |

```
              antes del fork()
                    │
                fork()
                    │
        ┌───────────┴────────────┐
        │                        │
   PROCESO PADRE            PROCESO HIJO
   fork() devuelve          fork() devuelve
      PID del hijo               0
        │                        │
   sigue su código          sigue su código
   (puede usar el PID       (mismo código,
   para hacer wait())        desde el mismo punto)
```

Por eso el patrón típico en C es un `if` sobre el valor devuelto:

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid < 0) {
        printf("Error al crear el proceso\n");
    } else if (pid == 0) {
        // Este bloque lo ejecuta SOLO el hijo
        printf("Soy el hijo, mi PID es %d\n", getpid());
    } else {
        // Este bloque lo ejecuta SOLO el padre
        printf("Soy el padre, el PID de mi hijo es %d\n", pid);
    }

    return 0;
}
```

Salida posible (el orden entre padre e hijo no está garantizado, el kernel decide cómo planifica cada uno):

```
Soy el hijo, mi PID es 4021
Soy el padre, el PID de mi hijo es 4021
```

### PID

El **PID** es el número entero único que el kernel asigna a cada proceso al crearlo. Sirve para identificarlo (por ejemplo para mandarle señales, o para esperarlo con `wait()`). `getpid()` devuelve el PID propio, `getppid()` devuelve el PID del padre.

---

## `wait()`

Cuando un proceso hijo termina, no desaparece del todo hasta que el padre "recoge" su estado de salida — mientras tanto queda como **proceso zombie**. `wait()` es la syscall que usa el padre para:

1. Bloquearse hasta que **algún** hijo termine.
2. Recibir el código de salida de ese hijo.
3. Liberar los recursos que quedaban del hijo (evita zombies).

```
PADRE                          HIJO
  │                              │
  │──fork()───────────────────▶  │ (hijo empieza a correr)
  │                              │
  │  wait()  (padre se bloquea)  │  ... hace su trabajo ...
  │      │                       │
  │      │                     exit()  (hijo termina)
  │      ▼                       │
  │  wait() retorna ◀────────────┘
  │  con el PID y el status del hijo
  ▼
sigue ejecutando
```

Ejemplo en C:

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        printf("Hijo trabajando...\n");
        return 42; // este es el código de salida del hijo
    } else {
        int status;
        pid_t hijo_terminado = wait(&status);
        if (WIFEXITED(status)) {
            printf("El hijo %d terminó con código %d\n",
                   hijo_terminado, WEXITSTATUS(status));
        }
    }
    return 0;
}
```

---

## `pipe()`

Una **pipe** (tubería) es un canal de comunicación unidireccional entre procesos, que el kernel implementa como un buffer en memoria con **dos file descriptors**: uno para leer y otro para escribir.

```c
int fd[2];
pipe(fd);
// fd[0] → extremo de LECTURA
// fd[1] → extremo de ESCRITURA
```

```
   escribe                    lee
      │                        │
      ▼                        ▼
 fd[1] ──▶ [ buffer de la pipe, en el kernel ] ──▶ fd[0]
```

Usualmente se combina con `fork()`: como el hijo hereda los fd del padre, ambos quedan conectados por la misma pipe y pueden comunicarse.

```
                    pipe(fd)
                       │
                    fork()
          ┌────────────┴────────────┐
          │                         │
      PADRE                     HIJO
   cierra fd[0]              cierra fd[1]
   escribe en fd[1]  ──────▶ lee de fd[0]
```

Ejemplo en C — el padre le manda un mensaje al hijo por la pipe:

```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>

int main() {
    int fd[2];
    pipe(fd); // fd[0] lectura, fd[1] escritura

    pid_t pid = fork();

    if (pid == 0) {
        // HIJO: lee
        close(fd[1]); // no lo va a usar
        char buffer[50];
        read(fd[0], buffer, sizeof(buffer));
        printf("Hijo recibió: %s\n", buffer);
        close(fd[0]);
    } else {
        // PADRE: escribe
        close(fd[0]); // no lo va a usar
        char *msg = "hola hijo";
        write(fd[1], msg, strlen(msg) + 1);
        close(fd[1]);
    }
    return 0;
}
```

Cerrar el extremo que no se usa es importante: si no, el kernel puede pensar que todavía hay un posible escritor/lector y algunas operaciones (como detectar el "fin de archivo" en la pipe) no funcionan bien.

---

## `exec()` y `execve()`

`fork()` crea una copia del proceso, pero sigue siendo **el mismo programa**. Si querés que un proceso pase a ejecutar **otro programa distinto**, se usa la familia `exec`.

`exec` **reemplaza** el código, los datos y la pila del proceso actual por los de otro programa. **No crea un proceso nuevo** — el PID sigue siendo el mismo, pero todo el contenido de memoria se reemplaza por el nuevo programa.

```
ANTES de exec()                    DESPUÉS de exec()
┌────────────────────┐              ┌────────────────────┐
│ proceso (PID 100)  │              │ proceso (PID 100)  │
│ código: programaA  │  execve()    │ código: programaB  │
│ datos: variables A │ ───────────▶ │ datos: variables B │
└────────────────────┘              └────────────────────┘
   (mismo PID, contenido totalmente distinto)
```

Un detalle clave: si `exec` tiene éxito, **nunca vuelve** — el código que estaba después de la llamada en el programa original ya no existe (fue reemplazado). Solo "vuelve" (devuelve -1) si hubo un error y no pudo reemplazar el proceso.

`execve()` es la syscall real del kernel (la más básica); `execl`, `execv`, `execvp`, etc. son funciones de librería que arman los argumentos de distinta forma y por adentro terminan llamando a `execve()`.

```c
int execve(const char *path, char *const argv[], char *const envp[]);
```

- `path`: qué programa ejecutar.
- `argv`: los argumentos (como el `argv` de `main`), terminado en `NULL`.
- `envp`: las variables de entorno, terminado en `NULL`.

Ejemplo en C:

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    char *argv[] = {"/bin/ls", "-l", NULL};
    char *envp[] = {NULL};

    execve("/bin/ls", argv, envp);

    // esta línea SOLO se ejecuta si execve() falló
    printf("Error: exec no se pudo ejecutar\n");
    return 1;
}
```

### El patrón clásico: fork() + exec() + wait()

Así es como una terminal ejecuta comandos: crea un hijo con `fork()`, el hijo se transforma en el programa pedido con `exec()`, y el padre espera con `wait()`.

```
                     PADRE (ej: la shell)
                          │
                       fork()
             ┌────────────┴────────────┐
             │                         │
         PADRE                        HIJO
     wait()  (se bloquea)       execve("/bin/ls", ...)
             │                  (el hijo AHORA es "ls")
             │                         │
             │                     termina "ls"
             │                         │
     wait() retorna ◀──────────────────┘
             │
        sigue la shell
```

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        // HIJO: se convierte en "ls -l"
        char *argv[] = {"/bin/ls", "-l", NULL};
        execve("/bin/ls", argv, NULL);
        perror("execve falló"); // solo si execve() falla
        return 1;
    } else {
        // PADRE: espera a que el hijo termine
        int status;
        wait(&status);
        printf("El comando terminó\n");
    }
    return 0;
}
```

---

## Todo junto: relación con el kernel

El kernel es el que **hace posible y controla** todo lo anterior:

- Mantiene una **tabla de procesos** con el estado de cada uno (PID, memoria, fd abiertos, estado: corriendo/bloqueado/zombie, etc.).
- Cuando llamás a `fork()`, es el kernel el que reserva estructuras nuevas, copia la información del proceso, y le asigna un PID nuevo.
- Cuando llamás a `exec`, es el kernel el que descarta el contenido de memoria viejo y carga el nuevo programa desde el ejecutable en disco.
- Cuando llamás a `wait()`, el kernel es quien sabe cuándo un hijo terminó (porque fue el propio kernel el que gestionó su finalización) y quien despierta al padre bloqueado.
- Cuando llamás a `pipe()`, `read()` o `write()`, el kernel es quien administra el buffer intermedio y decide cuándo un proceso debe bloquearse (por ejemplo, si intenta leer de una pipe vacía).
- Los **file descriptors** son, en el fondo, índices a estructuras internas del kernel — el proceso solo ve un número, pero es el kernel el que sabe a qué corresponde ese número realmente.

En resumen: el modelo de proceso de Unix (`fork` + `exec` + `wait`, comunicados por `pipe`, identificados por `PID`, con recursos referenciados por `fd`) es en gran parte una interfaz que expone, de forma controlada, las estructuras internas que el kernel ya mantiene para gestionar la multitarea del sistema.