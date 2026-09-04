# Principios de Algoritmos Greedy

## 1. Fundamentos Filosóficos y Psicológicos del Enfoque Greedy

El diseño de algoritmos voraces (o _greedy_) representa una de las manifestaciones más puras de la eficiencia, tanto en el plano computacional como en el cognitivo. No es meramente una técnica de programación; es una estrategia heurística que busca resolver problemas complejos de optimización mediante una sucesión de decisiones simplificadas. En un entorno donde los recursos de procesamiento son finitos, el enfoque voraz emerge como una herramienta estratégica indispensable, permitiendo obtener soluciones rápidas y, bajo ciertas condiciones estructurales, matemáticamente óptimas.

### Análisis del Marco de Diseño

El paradigma voraz se articula sobre tres componentes fundamentales:

1. **Estado Actual:** La representación de la información y los recursos disponibles en un momento determinado del proceso.
2. **Regla Sencilla de Elección:** Un criterio de selección que no requiere mirar hacia el futuro ni reconsiderar el pasado.
3. **Óptimo Local:** La mejor decisión posible basada exclusivamente en la regla aplicada al estado actual.

La tensión fundamental reside en la relación entre este **Óptimo Local** y el **Óptimo Global**. Mientras que el algoritmo confía en que la acumulación de mejores decisiones inmediatas conducirá a la mejor solución general, esta premisa no siempre se cumple, lo que exige un rigor analítico superior para validar su éxito.

### La Dualidad del "Cerebro Greedy"

Desde una perspectiva neuropsicológica, el algoritmo voraz emula el funcionamiento del **Sistema Límbico**, responsable de la gratificación inmediata (el instinto "unga-unga"). Bajo este prisma, el sesgo evolutivo de los arbustos de frambuesas ilustra esta condición: ante dos arbustos, el cerebro primitivo ordena recolectar del más cargado en el instante presente, ignorando si el otro arbusto permitiría una recolección más eficiente a largo plazo. En contraste, la **Corteza Prefrontal** representa la planeación y la capacidad de suprimir estos sesgos inmediatos en favor de una estrategia global.

### Intersección con la Inteligencia Artificial Moderna

Incluso en la vanguardia tecnológica, los Modelos de Lenguaje Extensos (LLMs) como ChatGPT y los procesos de **Descenso por Gradiente** operan bajo una naturaleza voraz. El gradiente busca la mejora inmediata en la dirección de la pendiente máxima, un comportamiento puramente _greedy_ que puede quedar atrapado en mínimos locales. Estos modelos heredan los sesgos humanos debido al entrenamiento sobre datos generados por sujetos que priorizan la coherencia local sobre la planificación estructural profunda.

### Metáfora Arquitectónica

Podemos visualizar esta dicotomía como el enfrentamiento entre el **Cerebro Primitivo** y el **Maestro de Ajedrez**. El primero reacciona instintivamente ante la recompensa inmediata, mientras que el segundo es capaz de sacrificar piezas en el presente para asegurar el jaque mate estratégico.

### Modelado de Abstracción

A continuación, se presenta un esquema genérico de un ciclo de optimización voraz:

```python
def optimizacion_greedy(elementos):
    # El estado inicial es el conjunto de elementos a procesar
    estado_actual = inicializar_estado(elementos)
    solucion = []
    
    # El ordenamiento es la optimización clave para la regla
    elementos_ordenados = aplicar_ordenamiento_estrategico(elementos)
    
    for item in elementos_ordenados:
        # La REGLA SENCILLA determina el Óptimo Local
        if es_compatible(item, estado_actual):
            # Se actualiza el estado con la elección local
            solucion.append(item)
            estado_actual = actualizar_estado(estado_actual, item)
            
    return solucion
```

**Complejidad:** La complejidad teórica de este esquema se define como \mathcal{O}(f(n)) = \mathcal{O}(\text{sort}(n) + n \cdot \text{regla}(n)). En la vasta mayoría de las aplicaciones, el ordenamiento inicial domina la función, resultando en una complejidad asintótica de \mathcal{O}(n \log n).

Esta comprensión de la psicología de la elección local nos permite abordar con rigor su primera aplicación práctica: el manejo de recursos temporales.

## 2. Planificación de Intervalos (Interval Scheduling)

El problema de la planificación de intervalos con un solo recurso plantea el desafío de maximizar la utilidad (cantidad de actividades) en escenarios de exclusión mutua.

### Análisis de Heurísticas Fallidas

Para entender la optimalidad, debemos desmontar analíticamente criterios intuitivos que resultan subóptimos:

1. **Menor duración:** Una tarea breve situada entre dos tareas ligeramente más largas puede bloquear a ambas, reduciendo el total de 2 a 1.
2. **Inicio temprano:** Priorizar lo que empieza primero puede llevar a seleccionar una tarea de gran extensión que bloquee el resto del horizonte temporal.
3. **Menor cantidad de colisiones:** Esta heurística es computacionalmente costosa, pues requiere el recálculo dinámico del estado tras cada selección, y no garantiza la optimalidad global.

### El Criterio de Optimalidad

La regla que garantiza el éxito es el ordenamiento por **tiempo de finalización (****f_i****)**. Al elegir la tarea que termina más temprano, liberamos el recurso lo antes posible.

### Demostración Rigurosa (Greedy Stays Ahead)

Postulamos que la solución voraz G = \{g_1, \dots, g_k\} "va por delante" de cualquier solución óptima O = \{o_1, \dots, o_m\}. Mediante inducción, demostramos que para todo índice r \leq k, se cumple que f(g_r) \leq f(o_r).

- **Base:** g_1 es la tarea que termina antes por definición, ergo f(g_1) \leq f(o_1).
- **Paso:** Si f(g_{r-1}) \leq f(o_{r-1}), dado que o_r es compatible con o_{r-1}, también lo es con g_{r-1}. Como el algoritmo voraz elige la tarea de menor tiempo de fin entre las compatibles, necesariamente f(g_r) \leq f(o_r). Consiguientemente, el algoritmo voraz nunca termina una tarea después que el óptimo para el mismo índice, maximizando el espacio para tareas futuras.

### Metáfora Cultural

Imagine el **Teatro de Ópera de Alta Gama**. Para maximizar las galas, el administrador acepta a aquel elenco que desocupa el escenario lo más rápido posible, permitiendo la entrada inmediata del siguiente.

### Implementación Técnica

```python
def scheduling(horarios):
    # Ordenar por tiempo de finalización: O(n log n)
    horarios_ordenados = sorted(horarios, key=lambda x: x[1])
    seleccionadas = []
    ultimo_fin = 0
    
    for inicio, fin in horarios_ordenados:
        if inicio >= ultimo_fin: # O(1) la regla de compatibilidad
            seleccionadas.append((inicio, fin))
            ultimo_fin = fin
            
    return seleccionadas
```

La complejidad total es \mathcal{O}(n \log n), dominada por el proceso de ordenamiento. Este manejo de un solo recurso sienta las bases para problemas de infraestructura multi-sala.

## 3. Partición de Intervalos para Múltiples Salas (Interval Partitioning)

Aquí el objetivo es minimizar la infraestructura necesaria para cubrir la demanda total de actividades, asegurando que ninguna se solape en la misma sala.

### Refutación de la Aplicación Secuencial

Aplicar el algoritmo de _scheduling_ simple k veces no produce el mínimo de salas. Consideremos las charlas: [A,0,1], [B,0,2], [C,2,3], [D,3,4], [E,1,5]. Si aplicamos _scheduling_ secuencial, la primera sala tomaría \{A, C, D\}, dejando \{B, E\} para una segunda sala. Sin embargo, si un error de elección voraz inicial (como priorizar por duración en lugar de inicio) forzara una asignación ineficiente, podríamos terminar requiriendo 3 salas innecesariamente. El número mínimo de salas es igual a la **profundidad** (máximo solapamiento) del conjunto de intervalos.

### Algoritmo de Asignación Óptima

La estrategia consiste en ordenar por **tiempo de inicio (****s_i****)**. Al procesar cada tarea, se asigna a una sala que ya esté libre. Para optimizar el proceso, utilizamos un **Min-Heap** que almacena los tiempos de finalización de las salas ocupadas.

### Metáfora de Logística

Funciona como el **Despachador de Cintas de Equipaje en un Aeropuerto**: revisa qué cinta se libera primero para asignar la siguiente carga sin demora.

### Implementación de Alto Rendimiento

```python
import heapq

def interval_partitioning(charlas):
    charlas.sort(key=lambda x: x[0]) # Ordenar por inicio
    # Heap para gestionar los tiempos de fin de las salas
    salas_heap = [] 
    
    for inicio, fin in charlas:
        if salas_heap and salas_heap[0] <= inicio:
            # Reutilización eficiente de la sala libre
            heapq.heapreplace(salas_heap, fin)
        else:
            # Apertura de nueva infraestructura
            heapq.heappush(salas_heap, fin)
            
    return len(salas_heap)
```

**Análisis Asintótico:** La complejidad temporal es \mathcal{O}(n \log n) y la espacial es \mathcal{O}(k), donde k es el número de salas. El uso del Heap evita la degradación a \mathcal{O}(n \cdot k) al permitir consultas del mínimo en tiempo logarítmico.

## 4. Minimización de la Latencia Máxima (Scheduling con Plazos Límite)

Contextualizamos tareas con duración t_i y plazos límite (_deadlines_) d_i. El objetivo es minimizar la latencia máxima L = \max(L_i), donde L_i = \max(0, f_i - d_i).

### Evaluación de Propuestas

1. **Menor** **t_i****:** Ignora la urgencia, causando latencias masivas en tareas con plazos próximos.
2. **Menor holgura (****d_i - t_i****):** No garantiza la optimalidad global, como se demuestra con el contraejemplo [(2, 1), (10, 10)], donde priorizar la holgura mínima puede resultar en una latencia mayor.

### Estrategia Óptima

Regla **Earliest Deadline First (EDF)**: ordenar ascendente por d_i.

### Prueba de Optimalidad mediante Argumento de Intercambio (Exchange Argument)

1. **Inversión:** Tareas i, j tales que i se programa antes que j, pero d_i > d_j.
2. **Postulado:** Un cronograma sin inversiones y sin tiempo ocioso es óptimo.
3. **Swapping formal:** Consideremos dos tareas contiguas invertidas i y j. Sea start el tiempo antes de i.
    - Original: f_i = start + t_i; f_j = start + t_i + t_j. Latencia L_j = f_j - d_j.
    - Tras intercambio (i' y j'): f'_j = start + t_j; f'_i = start + t_j + t_i.
    - Nótese que f'_i = f_j. Ergo, L'_i = f_j - d_i.
    - Como d_i > d_j, entonces L'_i = f_j - d_i < f_j - d_j = L_j.
    - La latencia de la tarea intercambiada es menor que la latencia original de j. Consecuientemente, \max(L'_i, L'_j) \leq \max(L_i, L_j). El intercambio no empeora la solución.

### Metáfora Operativa

La lógica del **Cartero de Correspondencia Urgente**: entrega siempre primero lo que vence antes para minimizar el impacto del retraso más grave.

### Implementación en Python

La complejidad es \mathcal{O}(n \log n) por el ordenamiento inicial EDF.

## 5. Codificación Óptima de Prefijos (Algoritmo de Huffman)

El desafío es la compresión de datos mediante códigos de longitud variable con la **propiedad de prefijo** (ningún código es prefijo de otro), representados como hojas en un árbol binario. Huffman supera el enfoque _Top-Down_ de Shannon-Fano mediante una construcción _Bottom-Up_.

### Resolución Caso "INTERESTELAR"

1. **Frecuencias:** E:3, T:2, R:2, I:1, N:1, S:1, L:1, A:1.
2. **Construcción del Árbol (Min-Heap):**
    - Fusionar (I, N) \to Nodo1(2).
    - Fusionar (S, L) \to Nodo2(2).
    - Fusionar (A, T) \to Nodo3(3).
    - Fusionar (R, Nodo1) \to Nodo4(4).
    - Fusionar (Nodo2, Nodo3) \to Nodo5(5).
    - Fusionar (E, Nodo4) \to Nodo6(7).
    - Fusionar (Nodo5, Nodo6) \to Raíz(12).
3. **Codificaciones Resultantes:**
    - `E`: `10`, `R`: `11`, `T`: `001`, `S`: `010`, `I`: `0000`, `N`: `0001`, `L`: `0110`, `A`: `0111`.
4. **Metáfora:** El **Idioma de Silbidos en una Tribu Selvática**, donde conceptos comunes son silbidos breves.

### Implementación en Python

Complejidad: \mathcal{O}(n + k \log k), donde k es el número de símbolos.

## 6. El Problema del Cambio de Monedas

### Demostración de No-Optimalidad

En el sistema [1, 5, 6, 9] para el valor 11:

- **Greedy:** 9+1+1 (3 monedas).
- **Óptimo:** 6+5 (2 monedas). El enfoque voraz solo es óptimo en **sistemas canónicos**, donde la estructura de denominaciones garantiza que el óptimo local sea global.

### Metáfora Mecánica

El **Dispensador de Dulces**: suelta lo más voluminoso primero para vaciar el depósito, sin considerar si piezas menores ajustarían mejor el espacio.

## 7. Carga de Combustible con Autonomía K

La regla óptima es **avanzar hasta la última estación alcanzable**. Justificación: detenerse en una estación i antes de la última posible j (i < j) nunca es superior, pues el horizonte de cobertura desde un tanque lleno en j siempre será mayor o igual que desde i.

### Metáfora de Supervivencia

El **Camello en los Oasis**: no bebe en cada punto, sino en el último posible antes del agotamiento. Complejidad \mathcal{O}(n).

## 8. Caso Práctico de Análisis: Submarinos y Faros

En una matriz con radio de iluminación R=2, el algoritmo de **Máxima Cobertura** (colocar faro donde más submarinos cubra) no es óptimo.

### Refutación de Optimalidad

En configuraciones simétricas, el algoritmo voraz elige el centro por su alta densidad inmediata. Esto "rompe" la posibilidad de que configuraciones descentradas cubran los extremos de forma compartida con menos faros. El algoritmo puede requerir 5 faros donde un diseño analítico manual lograría 4.

### Análisis de Costo Computacional

La complejidad es \mathcal{O}(n \cdot m \cdot S) debido a que, tras cada elección, se debe iterar sobre la matriz (n \times m) y verificar cuántos de los S submarinos permanecen descubiertos.

### Metáfora Industrial

El **Sistema de Focos en un Astillero**: un foco potente en el centro deja sombras críticas que obligan a instalar focos periféricos adicionales, incrementando el coste final.

**Lección Final:** La intuición voraz es una herramienta potente, pero su aplicación exige una validación rigurosa. Solo cuando la estructura del problema asegura que la "codicia" local no compromete el horizonte futuro, podemos certificar la elegancia de la optimalidad _greedy_.