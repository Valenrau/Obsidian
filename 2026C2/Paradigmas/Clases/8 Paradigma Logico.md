# Paradigma Lógico y Programación en Prolog: Guía Técnica y Práctica

## 1. Fundamentos del Paradigma Lógico y Declarativo

El **paradigma de programación lógica** se enmarca dentro del enfoque **declarativo**. A diferencia del paradigma imperativo —donde el programador especifica la secuencia paso a paso de instrucciones y transformaciones de estado que la computadora debe ejecutar para llegar a un resultado—, en el paradigma lógico la programación consiste en **describir el dominio del problema**. El programador declara un conjunto de hechos (relaciones verdaderas del mundo) y reglas de inferencia (relaciones condicionales), delegando en un **motor de resolución lógico** automático el proceso de deducción para encontrar la solución a una consulta o búsqueda.

```
Paradigma Imperativo:   Instrucciones (Cómo resolver el problema)
Paradigma Declarativo:  Relaciones y Reglas (Qué es el problema)  --->  Motor Lógico (Inferencia)
```

### El Supuesto del Mundo Cerrado (Closed World Assumption)
Una propiedad semántica de Prolog es el **Supuesto del Mundo Cerrado** (*Closed World Assumption*). En esta lógica de primer orden aplicada, el sistema asume que la base de conocimientos contiene **todo el conocimiento verdadero existente sobre el dominio**.

* **Verdadero (`true`)**: Un enunciado es considerado verdadero únicamente si está explícitamente afirmado como un hecho en la base de conocimientos o si puede ser deducido lógicamente a partir de las reglas declaradas.
* **Falso (`false`)**: Todo aquello que no esté registrado explícitamente ni pueda ser inferido de manera lógica a partir de la base de datos se presume **falso**. La falta de información o imposibilidad de prueba equivale lógicamente a la falsedad de la proposición.

### Estructura General de un Programa
Un programa en Prolog se compone de dos componentes fundamentales:
1. **Base de Conocimientos**: Archivo o conjunto de cláusulas formado por **hechos** y **reglas de inferencia**.
2. **Consultas u Objetivos (*Goals*)**: Preguntas dirigidas al motor de ejecución mediante la consola o la interfaz de evaluación para probar si una hipótesis es cierta o para obtener los valores de las variables que la hacen verdadera.

---

### Metáfora Didáctica: El Detective Jurídico y el Archivo Central
Imaginemos una oficina de archivos judiciales de una ciudad. Los **hechos** son las actas de nacimiento y documentos firmados e inalterables guardados en los estantes. Las **reglas** son las leyes vigentes del código civil redactadas por los juristas (por ejemplo, "si una persona figura como progenitor del progenitor de otra, entonces es su abuelo"). 

Cuando un ciudadano realiza una **consulta** ("¿Es Don Carlos el abuelo de Mateo?"), el juez no se pone a redactar nuevas leyes ni a inventar eventos: actúa como el **motor de Prolog**. Revisa las leyes, busca los expedientes en los estantes y, si encuentra la evidencia en los archivos, emite un dictamen favorable (`true`). Si en los archivos no existe constancia alguna sobre Don Carlos, el juez concluye que la relación no existe (`false`), no porque le conste la enemistad, sino porque dentro del archivo cerrado de la corte esa afirmación no es demostrable.

---

### Ejemplo Práctico / Técnico: Dominio Académico

En el siguiente bloque se define un conjunto de hechos sobre la situación académica de estudiantes y una regla para la promoción de la materia:

```prolog
% ==============================================================================
% BASE DE CONOCIMIENTOS
% ==============================================================================

% Hechos: Proposiciones incondicionales sobre exámenes aprobados
aprobo_parcial1(jose).
aprobo_parcial2(jose).
aprobo_parcial1(laura).

% Regla: Una persona X promociona si aprobó el Parcial 1 Y aprobó el Parcial 2
promociona(Persona) :- 
    aprobo_parcial1(Persona), 
    aprobo_parcial2(Persona).

% ==============================================================================
% CONSULTAS (GOALS)
% ==============================================================================

% Consulta 1: ¿Promociona José?
% ?- promociona(jose).
% Respuesta: true.

% Consulta 2: ¿Promociona Laura? (Laura no tiene registrado haber aprobado el parcial 2)
% ?- promociona(laura).
% Respuesta: false.

% Consulta 3: ¿Quiénes promocionan? (Uso de variable para deducción)
% ?- promociona(X).
% Respuesta: X = jose.
```

---

## 2. Sintaxis y Tipos de Datos: Los Términos

Prolog presenta un sistema de tipos unificado e ideológicamente simple: existe un **único tipo de dato** denominado **Término**. Toda constante, variable o estructura dentro del lenguaje es formalmente un término.

### Clasificación de Términos

| Categoría de Término | Definición y Reglas Sintácticas | Ejemplos |
| :--- | :--- | :--- |
| **Átomos** | Constantes simbólicas o texto plano. Comienzan obligatoriamente con **letra minúscula** o se delimitan con comillas simples. | `ale`, `ruso`, `ingresante`, `'Buenos Aires'` |
| **Números** | Valores numéricos enteros o de punto flotante. | `10`, `15.6`, `-42` |
| **Variables** | Identificadores de valores que aún no han sido ligados. Comienzan con **letra mayúscula** o con guion bajo (`_`). | `X`, `Persona`, `_aux` |
| **Términos Compuestos** | Formados por un **functor** (un átomo) seguido de $n$ argumentos encerrados entre paréntesis. El número $n$ de argumentos determina la **aridad** del término ($f/n$). | `father(bill, jake)` (aridad 2: `father/2`), `fecha(4, septiembre, 2026)` |

#### La Variable Anónima (`_`)
El guion bajo en soledad (`_`) representa la **variable anónima**. Se utiliza cuando en una regla o hecho se requiere una posición sintáctica válida pero cuyo valor específico resulta irrelevante para el cálculo. A diferencia de las variables normales con nombre, cada aparición de `_` en una misma regla representa una **variable independiente diferente**, evitando advertencias del compilador por variables no utilizadas (*singleton variables*).

---

### Cláusulas de Horn
Los programas en Prolog se estructuran matemáticamente como **Cláusulas de Horn**, las cuales constituyen un subconjunto estricto de la lógica de predicados de primer orden.

Una **Cláusula de Horn** es una disyunción de literales en la que existe como **máximo un literal positivo** (afirmativo):

$$
eg p_1 \lor 
eg p_2 \lor \dots \lor 
eg p_n \lor u$$

Aplicando equivalencia lógica sobre la implicación ($A \implies B \equiv 
eg A \lor B$), la expresión se transforma en una implicación directa donde el antecedente es una conjunción de condiciones:

$$(p_1 \land p_2 \land \dots \land p_n) \implies u$$

En la sintaxis de Prolog, el condicional se escribe invirtiendo el orden (poniendo la consecuencia a la izquierda) mediante el símbolo `:-` (que representa la flecha $\impliedby$), y las conjunciones ($\land$) se separan mediante comas `,`:

```prolog
u :- p1, p2, ..., pn.
```

* **Cabeza (Consecuente, $u$)**: Es el término único que se desea probar o inferir.
* **Cuerpo (Antecedente, $p_1, \dots, p_n$)**: Es la conjunción de condiciones que deben satisfacerse simultáneamente para que la cabeza sea verdadera.

#### Representación de la Disyunción ($\lor$)
Prolog no admite cláusulas de Horn con múltiples términos positivos a la izquierda. Para expresar una condición disyuntiva ("$X$ es padre de $Y$ si es su padre biológico **O** si es su padre adoptivo"), la buena práctica en Prolog consiste en definir **múltiples reglas con la misma cabeza**:

```prolog
progenitor(X, Y) :- madre(X, Y).
progenitor(X, Y) :- padre(X, Y).
```

---

### Metáfora Didáctica: La Plantilla de Rotulación y las Múltiples Ventanillas
Imaginemos los términos compuestos como **formularios impresos estandarizados**. El nombre del formulario impreso arriba en la hoja es el **functor** (por ejemplo, `transferencia_bancaria`), y los casilleros en blanco para completar son sus argumentos (aridad 3: origen, destino, monto).

Por otro lado, la disyunción expresada en múltiples reglas con el mismo nombre se asemeja a un trámite público que cuenta con **múltiples ventanillas habilitadas**. Si un ciudadano desea obtener el permiso de circulación, puede presentarse en la *Ventanilla A* (presentando certificado médico) **O** en la *Ventanilla B* (presentando pase laboral). El ciudadano prueba primero en la Ventanilla A; si no cumple los requisitos, no se retira a su casa, sino que prueba en la Ventanilla B.

---

### Ejemplo Práctico / Técnico: Comunicación e Idiomas

El siguiente código define hechos idiomáticos y determina cuándo dos personas pueden comunicarse entre sí, garantizando que una persona no sea comparada consigo misma:

```prolog
% ==============================================================================
% HECHOS DE IDIOMAS HABLADOS
% ==============================================================================
habla(ale, ruso).
habla(juan, ingles).
habla(maria, ruso).
habla(maria, ingles).

% ==============================================================================
% REGLA DE COMUNICACIÓN (Cláusula de Horn con disyunción implícita de identidades)
% Dos personas distintas X e Y se comunican si ambas hablan un mismo idioma L.
% ==============================================================================
se_comunica_con(X, Y) :- 
    habla(X, L), 
    habla(Y, L), 
    X \= Y.

% ==============================================================================
% CONSULTA Y EVALUACIÓN
% ==============================================================================
% ?- se_comunica_con(maria, ale).
% 1. habla(maria, L) -> L unifica con ruso.
% 2. habla(ale, ruso) -> Éxito (Ale habla ruso).
% 3. maria \= ale -> Éxito (son personas distintas).
% Resultado: true.
```

---

## 3. Mecanismo de Inferencia y Ejecución: Unificación y Resolución

El motor de Prolog procesa las consultas aplicando el método de **Resolución de Robinson** combinado con una estrategia de **Encadenamiento Hacia Atrás** (*Backward Chaining*).

```
Forward Chaining (Bottom-Up):  Hechos ---> Reglas ---> Nuevos Hechos ---> Objetivo final
Backward Chaining (Top-Down):  Objetivo Consulta ---> Reglas (Hacia atrás) ---> Hechos de la base
```

### Proceso de Búsqueda de Prolog
Cuando el motor recibe un objetivo $G$:
1. Toma el objetivo $G$ y recorre la base de conocimientos **de arriba hacia abajo**.
2. Intenta hacer **unificar** el objetivo $G$ con la cabeza de la primera cláusula que encuentre.
3. Si unifica con una **regla**, reemplaza ese objetivo por la conjunción de subtérminos del cuerpo de la regla, evaluando **de izquierda a derecha** en una estrategia de **búsqueda en profundidad** (*Depth-First Search - DFS*).
4. Si unifica con un **hecho**, el objetivo actual queda resuelto con éxito y el motor avanza al siguiente subtérmino pendiente.

---

### Unificación y Sustitución
La **unificación** es el proceso algorítmico mediante el cual dos términos se hacen estructuralmente idénticos encontrando una **sustitución** $\sigma$ de variables por otros términos.

Dado un conjunto de pares de términos $S = \{s_1 = t_1, s_2 = t_2, \dots, s_n = t_n\}$, una sustitución $\sigma = \{X_1 \leftarrow v_1, X_2 \leftarrow v_2, \dots\}$ es un **unificador** si al aplicar la sustitución a ambos lados, $s_i\sigma$ es idéntico a $t_i\sigma$ para todo $i$.

#### Reglas de Unificación según el Tipo de Componente:
1. **Variable y Variable ($X$ e $Y$)**: Unifican siempre. Ambas variables quedan ligadas (*aliased*) entre sí; lo que instancie a una instanciará automáticamente a la otra.
2. **Variable y Constante/Término ($X$ y $c$)**: Unifican siempre. La variable $X$ queda ligada al valor $c$ ($\sigma = \{X \leftarrow c\}$). Una vez ligada en una rama, $X$ no puede cambiar su valor (es inmutable).
3. **Constantes ($c$ y $c'$)**: Unifican únicamente si son idénticas ($c = c'$). Dos átomos distintos (ej. `juan` y `pedro`) **no unifican**.
4. **Términos Compuestos ($f(s_1, \dots, s_n)$ y $g(t_1, \dots, t_m)$)**: Unifican únicamente si:
   * Tienen el mismo functor ($f = g$).
   * Tienen la misma aridad ($n = m$).
   * Todos los argumentos correspondientes unifican entre sí ($s_i$ unifica con $t_i$ para todo $i \in \{1 \dots n\}$).

---

### Inversibilidad (Relación Multidireccional)
Gracias al mecanismo de unificación, los predicados bien diseñados en Prolog poseen la propiedad de **inversibilidad**: no existe una distinción rígida en tiempo de compilación entre parámetros de "entrada" (*input*) y parámetros de "salida" (*output*). Un mismo predicado puede ser utilizado para:
* Verificar una relación completamente instanciada (`?- suma(2, 3, 5).`).
* Calcular el valor de salida a partir de las entradas (`?- suma(2, 3, Resultado).`).
* Generar las entradas que producen una determinada salida (`?- suma(X, Y, 5).`).

---

### Metáfora Didáctica: Las Llaves Maestro y las Claves Combinadas
La unificación es como un cerrajero ajustando una **llave de combinación modular** para abrir un candado. Una variable es una ranura vacía en la llave que puede tomar la forma de cualquier diente. Si la ranura $X$ toca un pin fijo de altura 3 (una constante), la ranura se moldea a la altura 3. Si más adelante en el mismo mecanismo la misma llave vuelve a tocar otra sección, la ranura $X$ sigue midiendo 3. Si en algún punto esa ranura de altura 3 intenta ingresar en un pin fijo de altura 5, el mecanismo **se traba** (falla la unificación) y el cerrajero debe dar un paso atrás (*backtracking*) para probar otro molde de llave.

---

### Ejemplo Práctico / Técnico: Genealogía y Unificación

Dado el siguiente programa familiar, se analiza el proceso de unificación para determinar quién es el abuelo de Manuel:

```prolog
% ==============================================================================
% BASE DE CONOCIMIENTOS FAMILIAR
% ==============================================================================
padre(juan, carlos).
padre(juan, luis).
padre(carlos, daniel).
padre(luis, manuel).
padre(luis, ramiro).

% Regla: X es abuelo de Y si X es padre de Z e Y es hijo de Z.
abuelo(X, Y) :- 
    padre(X, Z), 
    padre(Z, Y).

% ==============================================================================
% CONSULTA
% ==============================================================================
% ?- abuelo(X, manuel).
```

#### Árbol de Búsqueda y Paso a Paso Lógico:
1. **Objetivo Inicial**: `abuelo(X, manuel)`.
2. **Unificación con la Cabeza**: Unifica con `abuelo(X_1, Y_1)` con la sustitución $\sigma_1 = \{Y_1 \leftarrow 	ext{manuel}, X \leftarrow X_1\}$.
3. **Cuerpo de la Regla**: Queda pendiente resolver `padre(X_1, Z_1), padre(Z_1, manuel)`.
4. **Primer Subobjetivo**: `padre(X_1, Z_1)`. Recorre la base de conocimientos desde arriba:
   * *Intento 1*: Unifica con `padre(juan, carlos)`. Sustitución $\sigma_2 = \{X_1 \leftarrow 	ext{juan}, Z_1 \leftarrow 	ext{carlos}\}$.
   * *Segundo Subobjetivo*: Reemplaza $Z_1$ y evalúa `padre(carlos, manuel)`.
   * Recorre la base: `padre(carlos, daniel)` no unifica con `padre(carlos, manuel)` porque `daniel` $
eq$ `manuel`. No hay más hechos de `carlos`. **Falla**.
5. **Backtracking al Primer Subobjetivo**: Deshace $\sigma_2$ ($Z_1$ vuelve a estar libre) y busca la siguiente cláusula para `padre(juan, Z_1)`:
   * *Intento 2*: Unifica con `padre(juan, luis)`. Sustitución $\sigma_3 = \{X_1 \leftarrow 	ext{juan}, Z_1 \leftarrow 	ext{luis}\}$.
   * *Segundo Subobjetivo*: Evalúa `padre(luis, manuel)`.
   * Recorre la base: Encuentra el hecho explícito `padre(luis, manuel)`. **Éxito**.
6. **Resultado Final**: Se retorna $X = 	ext{juan}$.

---

## 4. Motor de Búsqueda, Backtracking y el Modelo de Seguimiento (Trace)

### El Algoritmo de Backtracking
El **Backtracking** (retroceso sistemático) es la estrategia algorítmica central del motor de Prolog. Cuando el sistema está resolviendo una conjunción de metas y alcanza un punto en el que un subobjetivo **falla** (no puede unificar con ninguna cláusula de la base), el motor retrocede automáticamente al **último punto de elección** (*choice point*) donde existían alternativas no exploradas. Al retroceder, el motor **deshace (desliga)** las variables que se habían instanciado desde ese punto de elección y procede a probar la siguiente alternativa disponible.

---

### El Modelo de Seguimiento de Cuatro Puertas (Trace)
Para depurar y comprender la ejecución paso a paso en entornos de desarrollo como SWI-Prolog, el motor expone el comando `trace`, el cual modela el ciclo de vida de cada subobjetivo a través de **cuatro eventos o puertas principales**:

```
           +---------------------------------+
           |              CALL               |  (Intento de satisfacer un objetivo)
           +---------------------------------+
                           |
             +-------------+-------------+
             |                           |
             v                           v
     +---------------+           +---------------+
     |     EXIT      |           |     FAIL      |  (Imposibilidad de unificar)
     +---------------+           +---------------+
             |                           ^
             v                           |
     +---------------+                   |
     |     REDO      | ------------------+  (Backtracking solicita nueva opción)
     +---------------+
```

1. **`CALL`**: Se gatilla al inicio del intento de satisfacer un objetivo específico.
2. **`EXIT`**: Se gatilla cuando el objetivo se ha satisfecho con éxito, habiendo encontrado una sustitución válida para sus variables.
3. **`REDO`**: Se gatilla cuando un objetivo posterior en la regla falló, obligando al algoritmo a realizar backtracking y volver sobre este objetivo para buscar una solución alternativa.
4. **`FAIL`**: Se gatilla cuando se han agotado todas las cláusulas probables y no existe ninguna sustitución que permita satisfacer el objetivo.

---

### Prevención de Bucles Infinitos y Recursión de Dominio (`Stack Overflow`)
En programas recursivos sobre grafos o redes de transporte, un diseño ingenuo de reglas puede provocar que el motor caiga en ciclos recursivos infinitos, consumiendo toda la memoria de la pila de llamadas (*Stack Overflow*).

Por ejemplo, si se define la conectividad en un mapa de la siguiente forma:

```prolog
% Mal diseño: Bucle infinito en backtracking
viaje(X, Y) :- viaje(X, Z), viaje(Z, Y).
viaje(X, Y) :- directo(X, Y).
```

El motor ingresará eternamente en la primera rama sintáctica `viaje(X, Z)` antes de intentar los hechos directos.

#### Regla de Oro para Estructuras Recursivas:
1. **Colocar siempre los Casos Base (directos) primero**.
2. **Utilizar un predicado auxiliar directo** para descomponer la llamada recursiva, evitando que la regla principal se invoque a sí misma sin consumir un paso real del dominio.

---

### Ejemplo Práctico / Técnico: Sistema de Transporte de Ciudades

El siguiente código implementa una red de rutas entre ciudades usando transporte por auto, tren o avión, resolviendo la conectividad recursiva sin entrar en bucles infinitos:

```prolog
% ==============================================================================
% RUTA BASE: CONEXIONES DIRECTAS (HECHOS)
% ==============================================================================
en_auto(auckland, hamilton).
en_auto(hamilton, raglan).
en_tren(valmont, metz).
en_avion(singapur, auckland).

% ==============================================================================
% PREDICADO AUXILIAR DIRECTO (Caso Base Unificado)
% Existe viaje directo si se puede ir por auto, tren o avión.
% ==============================================================================
directo(X, Y) :- en_auto(X, Y).
directo(X, Y) :- en_tren(X, Y).
directo(X, Y) :- en_avion(X, Y).

% ==============================================================================
% PREDICADO RECURSIVO DE VIAJE MULTI-ESCALA
% ==============================================================================
% Caso Base: Hay un viaje directo entre X e Y.
travel(X, Y) :- directo(X, Y).

% Caso Recursivo: Se viaja de X a un punto intermedio Z (paso directo),
% y luego se resuelve recursivamente el trayecto desde Z hasta Y.
travel(X, Y) :- 
    directo(X, Z), 
    X \= Z, 
    travel(Z, Y).

% ==============================================================================
% CONSULTA DE EVALUACIÓN
% ==============================================================================
% ?- travel(singapur, raglan).
% Respuesta: true.
```

#### Seguimiento Simplificado (`trace`):
```
[1] 1 Call: travel(singapur, raglan) ? 
[2] 2 Call: directo(singapur, raglan) ? 
[2] 2 Fail: directo(singapur, raglan) ? 
[1] 1 REDO: travel(singapur, raglan) ? 
[3] 2 Call: directo(singapur, Z) ? 
[3] 2 Exit: directo(singapur, auckland) -> Z = auckland
[4] 2 Call: travel(auckland, raglan) ? 
[5] 3 Call: directo(auckland, Z2) ? 
[5] 3 Exit: directo(auckland, hamilton) -> Z2 = hamilton
[6] 3 Call: travel(hamilton, raglan) ? 
[7] 4 Call: directo(hamilton, raglan) ? 
[7] 4 Exit: directo(hamilton, raglan)
[6] 3 Exit: travel(hamilton, raglan)
[4] 2 Exit: travel(auckland, raglan)
[1] 1 Exit: travel(singapur, raglan)
```

---

## 5. Aritmética en Prolog: Inserción de Expresiones y el Operador `is`

Una de las particularidades conceptuales más marcadas de Prolog es que **las expresiones aritméticas no se evalúan automáticamente**. Para el motor gráfico y simbólico de Prolog, un término como `1 + 2` no es el número `3`, sino un término compuesto escrito en notación de infijo: `+(1, 2)`.

### Operadores de Comparación y Evaluación Aritmética

| Operador | Propósito y Comportamiento | Ejemplo | Resultado |
| :--- | :--- | :--- | :--- |
| `=` | **Unificación Estructural Puro**: No evalúa aritmética. Verifica si ambos lados son simbólicamente idénticos. | `?- 1 + 2 = 2 + 1.` | `false` (los functores `+(1,2)` y `+(2,1)` difieren) |
| `=:=` | **Igualdad Aritmética**: Evalúa obligatoriamente ambos lados numéricamente y compara sus valores finales. | `?- 1 + 2 =:= 2 + 1.` | `true` ($3 =:= 3$) |
| `=\=` | **Desigualdad Aritmética**: Evalúa ambos lados y retorna éxito si los resultados numéricos son distintos. | `?- 10 =\= 100.` | `true` |
| `is` | **Evaluación y Asignación Aritmética**: Evalúa el lado derecho numéricamente y **unifica** el resultado con la variable del lado izquierdo. | `?- X is 1 + 2.` | `X = 3` |

### Regla de Oro del Operador `is`
Para que una operación `X is Expresion` se ejecute correctamente, **todos los componentes y variables presentes en la `Expresion` del lado derecho deben estar completamente instanciados con valores numéricos en el momento exacto de la evaluación**. 

Si el lado derecho contiene una variable libre (no ligada), Prolog interrumpirá la ejecución lanzando un error de instanciación (`Instantiation error`).

```prolog
% Correcto: V y T están instanciados
distancia(X, D) :- velocidad(X, V), tiempo(X, T), D is V * T.

% Incorrecto: D e is intentan resolver algebraicamente sin valores en la derecha
D is V * T, velocidad(X, V), tiempo(X, T). % Instantiation Error!
```

---

### Metáfora Didáctica: La Pizarra de Fórmulas y la Calculadora
Pensar en Prolog procesando la suma `1 + 2` es como imaginar a un profesor de matemáticas escribiendo en una **pizarra de tiza** el texto `"1 + 2"`. El profesor no borra la pizarra mágicamente para convertirla en un `"3"`; el texto escrito es un dibujo simbólico de tres caracteres. 

El operador `is` equivale a tomar una **calculadora de mano**, ingresar manualmente los números que están escritos en la pizarra, presionar la tecla `=` y anotar el resultado numérico resultante en un casillero designado (la variable del lado izquierdo). Si la pizarra dice `"1 + X"`, la calculadora no puede presionar el botón de calcular e imprime un error.

---

### Ejemplo Práctico / Técnico: Cálculo de Física Móvil

```prolog
% ==============================================================================
% BASE DE DATOS DE FÍSICA
% ==============================================================================
velocidad(mati, 10).   % km/h
velocidad(fede, 5).
velocidad(nacho, 5).

tiempo(mati, 30).      % minutos
tiempo(fede, 40).
tiempo(nacho, 50).

% ==============================================================================
% REGLA DE CÁLCULO
% D is V * T exige que V y T vengan ligados desde las llamadas anteriores.
% ==============================================================================
distancia(Persona, D) :- 
    velocidad(Persona, V), 
    tiempo(Persona, T), 
    D is V * T.

% ==============================================================================
% CONSULTA
% ==============================================================================
% ?- distancia(mati, D).
% 1. velocidad(mati, V) -> V = 10.
% 2. tiempo(mati, T)    -> T = 30.
% 3. D is 10 * 30       -> D = 300.
% Resultado: D = 300.
```

---

## 6. Estructuras de Datos Avanzadas: Listas y Recursión

Las **listas** en Prolog son secuencias ordenadas de elementos de cualquier tipo (átomos, números, variables o incluso otras listas). Sintácticamente se delimitan mediante corchetes `[...]`.

### Representación Estructural Cabeza y Cola (`[Head | Tail]`)
Internamente, una lista no vacía se compone de dos partes fundamentales mediante el operador de separación barra vertical `|`:

```
               Lista: [1, 2, 3, 4]
              /                      Cabeza (Head): 1        Cola (Tail): [2, 3, 4]
  (El primer elemento)     (Una LISTA con el resto de los elementos)
```

* **`Head` (Cabeza)**: Es el primer elemento individual de la lista (puede ser cualquier término).
* **`Tail` (Cola)**: Es **siempre una lista** que contiene todos los elementos restantes de la secuencia. La cola de una lista con un único elemento (`[x]`) es la lista vacía `[]`.

#### Sintaxis de Descomposición Patrón:
```prolog
?- [1, 2, 3] = [H | T].
H = 1,
T = [2, 3].

?- [1] = [H | T].
H = 1,
T = [].

?- [1, 2, 3, 4] = [A, B | Resto].
A = 1,
B = 2,
Resto = [3, 4].
```

---

### Patrón General de Recursión sobre Listas
Para procesar una lista en Prolog, se aplica un esquema inductivo clásico:
1. **Caso Base**: Define el comportamiento para la lista vacía `[]` o para una lista con la cabeza buscada.
2. **Caso Recursivo**: Descompone la lista en `[Head | Tail]`, procesa la `Head` (o la ignora) y realiza la llamada recursiva pasando la `Tail`.

---

### Metáfora Didáctica: La Trenza de Vagones de Tren y las Matrioskas
Una lista en Prolog se comporta exactamente como una **matrioska (muñeca rusa)**. La muñeca exterior completa es el término de la lista. Cuando abres la primera capa, obtienes una figura sólida en la mano (la **Cabeza**) y otra muñeca entera más pequeña adentro (la **Cola**). Para contar cuántas muñecas hay dentro, tomas la muñeca en tu mano (+1) y le pides a un ayudante que abra la muñeca más pequeña. El proceso continúa abriendo muñecas sucesivamente hasta llegar a la última pieza diminuta y sólida que no se puede abrir más: la **lista vacía `[]`**, punto en el cual comienzas a cerrar las capas sumando los resultados.

---

### Ejemplo Práctico / Técnico: Colección de Predicados sobre Listas

A continuación se presentan tres algoritmos completos sobre listas desarrollados con buenas prácticas de unificación directa:

#### 1. Pertenencia de un Elemento (`contiene/2`)
Determina si un elemento $X$ forma parte de una lista dada:

```prolog
% Caso Base: El elemento X es directamente la Cabeza de la lista.
contiene(X, [X | _]).

% Caso Recursivo: El elemento X no era la cabeza, se busca recursivamente en la Cola.
contiene(X, [_ | Tail]) :- 
    contiene(X, Tail).

% Consulta:
% ?- contiene(1, [3, 1, 2]).
% 1. contiene(1, [3|[1, 2]]) -> Falla caso base (1 \= 3). Pasa al caso recursivo.
% 2. contiene(1, [1, 2])     -> Unifica caso base (1 = 1). Éxito.
% Resultado: true.
```

#### 2. Conteo de Elementos con Acumulador Desapilado (`contar/2`)
Calcula la longitud total de una lista de elementos:

```prolog
% Caso Base: La lista vacía tiene longitud 0.
contar([], 0).

% Caso Recursivo: La longitud de la lista [Head|Tail] es 1 + la longitud de Tail.
contar([_ | Tail], CantidadTotal) :- 
    contar(Tail, CantidadParcial), 
    CantidadTotal is CantidadParcial + 1.

% Consulta:
% ?- contar([a, b, c], N).
% N = 3.
```

#### 3. Verificación de Igualdad Estructural de Listas (`iguales/2`)

```prolog
% Caso Base: Dos listas vacías son iguales entre sí.
iguales([], []).

% Caso Recursivo: Dos listas son iguales si sus cabezas son idénticas 
% y sus colas son estructuralmente iguales.
iguales([H | T1], [H | T2]) :- 
    iguales(T1, T2).

% Consulta:
% ?- iguales([1, 2], [1, 2]).
% Resultado: true.
```

#### 4. Inserción al Inicio (`agregar_inicio/3`)
Demuestra cómo la unificación en la cabeza resuelve la construcción de listas sin necesidad de predicados de mutación:

```prolog
% Forma directa y elegante en una sola línea de unificación:
% La lista resultante L2 es simplemente el elemento X colocado como Cabeza de L1.
agregar_inicio(X, L1, [X | L1]).

% Consulta:
% ?- agregar_inicio(4, [1, 2, 3], L2).
% Resultado: L2 = [4, 1, 2, 3].
```

---

## 7. Tabla Resumen de Operadores y Buenas Prácticas

### Matriz Comparativa de Operadores Sintácticos

| Operador | Nombre / Función | Evalúa Aritmética | Permite Variables Libres a la Derecha |
| :---: | :--- | :---: | :---: |
| `:-` | Definición de Regla ($ \impliedby $) | No | N/A |
| `,` | Conjunción Lógica ($ \land $) | No | N/A |
| `=` | Unificación de Términos | **No** | **Sí** |
| `\=` | Imposibilidad de Unificar | **No** | **Sí** |
| `=:=` | Igualdad Aritmética | **Sí** | **No** (Lanza Error) |
| `=\=` | Desigualdad Aritmética | **Sí** | **No** (Lanza Error) |
| `is` | Asignación por Evaluación | **Sí** (solo lado derecho) | **No** (Lanza Error) |

---

### Guía de Buenas Prácticas para Exámenes y Proyectos de Diseño Lógico

1. **Priorizar Unificación en la Cabeza**: En lugar de asignar valores explícitos dentro del cuerpo de la regla (ej. `M = H`), unifica directamente las variables en la cabeza de la regla (ej. `mi_predicado([H | T], H)`). Esto acorta el código, mejora la legibilidad y optimiza el rendimiento del motor.
2. **Respetar la Invariante del Mundo Cerrado**: No intente definir reglas para casos falsos explicitando resultados negativos salvo que sea estrictamente necesario mediante negación por fracaso (`not` / `\+`). Si una relación no se cumple, deje que la ausencia de hechos o el fallo de la unificación retorne `false` naturalmente.
3. **Manejar la Recursión con Cuidado de Pila**: Al definir reglas sobre estructuras jerárquicas o grafos, garantice siempre que la llamada recursiva avance achicando el dominio del problema (consumiendo elementos de una lista o avanzando un paso en una red de ciudades).
4. **Distinguir Estrictamente Variables de Átomos**: Recuerde la convención de capitalización. Un error común de tipografía es escribir nombres de constantes en mayúscula (ej. `Juan` en lugar de `juan`), lo cual convierte la constante en una variable libre y desconfigura la unificación de la base de conocimientos.
