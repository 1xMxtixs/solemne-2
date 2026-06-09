# Solemne 2: Optimización de la Distribución de Combustibles
## Formulación Determinista, Metaheurística y Análisis de Resultados
### CINF105: Optimización — Facultad de Ingeniería, Universidad Andrés Bello

Este repositorio contiene la implementación y el informe correspondientes a la **Solemne 2** del curso de **Optimización (CINF105)**, sección **NRC 7924**, dictado por el profesor **Luis Fuentes**.

**Integrantes:**
*   Matías Santos
*   José Romero
*   David Cortez

---

## 📌 Resumen del Proyecto

El proyecto aborda un problema complejo de logística y distribución física de combustibles (Regular y Diésel) desde un depósito central hacia cuatro estaciones de servicio. La distribución se realiza mediante una flota de dos camiones cisterna con tecnología de doble compartimento. 

El objetivo principal es diseñar un plan operativo de costo mínimo que determine:
1.  Las rutas óptimas para cada camión.
2.  La asignación de productos (Regular/Diésel) a cada compartimento.
3.  Las cantidades de combustible a cargar y entregar.
4.  La secuencia detallada de carga en el depósito central (Scheduling de carga).
5.  El cumplimiento de múltiples restricciones operativas críticas (capacidad, compatibilidad, ventanas de tiempo y estabilidad de la carga).

---

## 📂 Estructura del Proyecto

*   **`simulated_annealing_combustibles.ipynb`**: Notebook de Jupyter que contiene la implementación en Python del algoritmo de **Simulated Annealing (Recocido Simulado)** para la resolución metaheurística del ruteo, además del análisis de convergencia y visualizaciones de los resultados.
*   **`optimización___Solemne_2.pdf`**: Documento completo en formato PDF con el informe académico, el cual detalla la formulación matemática determinista, el scheduling en el depósito, la validación de la metaheurística, el análisis de la propuesta del operador y la propuesta de extensión estocástica.

---

## ⚙️ Descripción del Problema

### 1. Demanda y Ventanas de Tiempo por Estación
La empresa abastece a 4 estaciones con las siguientes demandas y restricciones horarias:

| Estación | Demanda Regular (L) | Demanda Diésel (L) | Ventana de Tiempo |
| :---: | :---: | :---: | :---: |
| **1** | 3.000 | 2.000 | `[06:00 - 10:00]` |
| **2** | 4.000 | 1.500 | `[07:00 - 12:00]` |
| **3** | 2.500 | 3.000 | `[08:00 - 14:00]` |
| **4** | 1.000 | 2.500 | `[06:00 - 11:00]` |

### 2. Características de la Flota de Camiones
Se cuenta con 2 camiones de doble compartimento con distintas capacidades y costos asociados:

| Camión | Capacidad C0 (L) | Capacidad C1 (L) | Costo Fijo ($) | Velocidad Promedio | Hora de Salida Programada |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **T1** | 8.000 | 7.000 | 500 | 60 km/h | 05:00 |
| **T2** | 6.000 | 9.000 | 400 | 60 km/h | 05:30 |

### 3. Parámetros Económicos y Logísticos
*   **Costo por kilómetro recorrido ($c_d$):** \$2 / km.
*   **Penalización por demanda no satisfecha / Shortage ($c_s$):** \$10 / litro no entregado.
*   **Tiempo de servicio en estación ($T_j$):** 30 minutos (fijo en todas las estaciones).
*   **Imbalance de Carga Máximo ($\Delta$):** Diferencia máxima permitida entre los *fill ratios* (porcentaje de llenado) de los compartimentos del camión en cualquier instante de la ruta: $\Delta = 0.30$ ($30\%$).

---

## 🛠️ Formulación Matemática del Modelo Determinista

El modelo optimiza simultáneamente las decisiones de ruteo y asignación de productos a compartimentos. 

### Función Objetivo
Minimizar el costo total de operación:
$$\min Z = c_d \sum_{k \in K} \sum_{i \in V} \sum_{\substack{j \in V \\ j \neq i}} d_{ij} x_{ijk} + \sum_{k \in K} F_k y_k + c_s \sum_{j \in N} \sum_{p \in P} s_{jp}$$
*   **Término 1:** Costo total por distancia recorrida.
*   **Término 2:** Costo fijo por activar y usar los camiones.
*   **Término 3:** Costo por penalización de demanda insatisfecha (shortage).

### Restricciones Principales
1.  **Ruteo y Flujo:** Aseguran que si un camión se activa, sale del depósito ($0$), visita estaciones y regresa al depósito, respetando la conservación de flujo y visitando cada estación como máximo una vez.
2.  **Satisfacción de Demanda y Capacidad:** La entrega total de un compartimento a las estaciones de la ruta no debe superar la capacidad del compartimento. Las cantidades no entregadas se registran como shortage.
3.  **Compatibilidad de Compartimento-Producto:** Un compartimento solo puede transportar un tipo de producto en cada viaje (Regular o Diésel) para evitar contaminación cruzada.
4.  **Estabilidad de Carga en Ruta:** Después de salir del depósito y tras realizar descargas en cada estación, la diferencia del porcentaje de llenado entre los compartimentos $C_0$ y $C_1$ de un mismo camión no puede superar el límite $\Delta = 0.30$.
5.  **Ventanas de Tiempo y Arribo:** Controlan los tiempos de viaje basándose en la velocidad constante (60 km/h) y penalizan el retraso si la llegada es posterior al cierre de la ventana de la estación.
6.  **Eliminación de Subtours:** Restricciones de tipo Miller-Tucker-Zemlin (MTZ) para evitar ciclos cerrados desconectados del depósito central.

---

## 🕒 Scheduling de Carga en el Depósito

Antes de iniciar la distribución, los camiones deben cargarse bajo las siguientes reglas operativas:
*   **Secuencialidad Interna:** Cada camión se carga de forma secuencial compartimento por compartimento (primero $C_0$, luego $C_1$).
*   **Infraestructura de Carga:** El depósito tiene **una bahía para combustible Regular** y **una bahía para Diésel**. No se pueden cargar simultáneamente dos compartimentos con el mismo combustible (conflicto de bahía), pero sí se pueden cargar combustibles distintos en camiones diferentes a la vez.
*   **Tasas de Carga y Tiempos de Limpieza:**
    *   Tasa Regular: 500 L/min.
    *   Tasa Diésel: 400 L/min.
    *   Limpieza: 5 minutos de tiempo muerto al terminar cada operación de carga.

### Cronograma de Carga Propuesto (Makespan Óptimo)
Para los volúmenes requeridos en la solución óptima, se calcula la duración de cada operación mediante la fórmula:
$$p_{kc} = \frac{\text{Vol}_{kc}}{r_p} + \mu$$

| Operación | Producto | Volumen (L) | Tasa (L/min) | Limpieza (min) | Duración Total (min) |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **T1-C0** | Regular | 5.000 | 500 | 5 | 15.0 |
| **T1-C1** | Diésel | 5.000 | 400 | 5 | 17.5 |
| **T2-C0** | Diésel | 4.000 | 400 | 5 | 15.0 |
| **T2-C1** | Regular | 3.500 | 500 | 5 | 12.0 |

**Secuencia de Carga y Makespan:**
*   **Bahía Regular:** Carga **T1-C0** del minuto 0 al 15. Luego carga **T2-C1** del minuto 15 al 27.
*   **Bahía Diésel:** Carga **T2-C0** del minuto 0 al 15. Luego carga **T1-C1** del minuto 15 al 32.5.
*   **Makespan Total ($C_{\text{máx}}$):** **32.5 minutos**. 
*   **Verificación de Horarios:** Si el proceso inicia a las **04:27:30**, el camión T1 finaliza su carga y queda listo para salir a las **05:00**, y el camión T2 finaliza a las **04:57:00** (listo antes de su salida programada de las **05:30**).

---

## 🧠 Diseño de la Metaheurística: Simulated Annealing

Debido a la complejidad combinatoria del problema (Ruteo VRP con Ventanas de Tiempo, Compartimentos y Restricciones No Lineales de Estabilidad), se diseñó un algoritmo de **Recocido Simulado** en Python para encontrar soluciones óptimas o cuasi-óptimas.

### 1. Representación de la Solución
Una solución $S$ se define como una colección de estructuras por camión:
$$S = \{ R_k, A_k, Q_k \}_{k \in K}$$
Donde $R_k$ es la ruta ordenada, $A_k$ es la asignación de producto a compartimentos y $Q_k$ representa las cargas asignadas.

### 2. Función de Evaluación y Penalizaciones
Se utiliza una función de evaluación que suma al costo real penalizaciones por infactibilidad mediante factores de escala ($\lambda$):
$$f(S) = C_{\text{dist}}(S) + C_{\text{fijo}}(S) + C_{\text{shortage}}(S) + P_{\text{inf}}(S)$$
$$P_{\text{inf}}(S) = \lambda_{\text{cap}} P_{\text{cap}}(S) + \lambda_{\text{est}} P_{\text{est}}(S) + \lambda_{\text{time}} P_{\text{time}}(S) + \lambda_{\text{estaciones}} P_{\text{estaciones}}(S)$$

Los factores de penalización definidos en el código son:
*   $\lambda_{\text{capacidad}} = 10.000$
*   $\lambda_{\text{estabilidad}} = 50.000$
*   $\lambda_{\text{tiempo}} = 1.000$
*   $\lambda_{\text{estaciones no asignadas / duplicadas}} = 100.000$

### 3. Operadores de Vecindad
En cada iteración se selecciona aleatoriamente uno de los siguientes movimientos vecinos:
*   **`swap`:** Intercambiar de forma aleatoria una estación de la ruta de T1 con otra de la ruta de T2.
*   **`move`:** Quitar una estación de la ruta de un camión e insertarla en una posición aleatoria de la ruta del otro camión.
*   **`reverse`:** Invertir el orden de visita de las estaciones en la ruta de uno de los camiones.
*   **`asignacion`:** Intercambiar la asignación de combustibles de los compartimentos $C_0$ y $C_1$ de un camión (ej. cambiar Regular a C1 y Diésel a C0).

### 4. Parámetros de Enfriamiento
*   **Temperatura Inicial ($T_0$):** 1.000
*   **Temperatura Final ($T_f$):** 0.01
*   **Factor de Enfriamiento Geométrico ($\alpha$):** 0.95 ($T \leftarrow \alpha T$)
*   **Iteraciones por Temperatura:** 100
*   **Semilla Aleatoria:** 42 (para garantizar reproducibilidad)

---

## 📈 Resultados Obtenidos

La metaheurística encuentra la **solución óptima global** con costo total de **$1.130** (cero penalizaciones de factibilidad y cero shortage).

### Desglose de Costos de la Mejor Solución:
*   **Costo de Distancia:** $230 (115 km totales recorridos).
*   **Costo Fijo de la Flota:** $900 (ambos camiones utilizados: T1 y T2).
*   **Penalizaciones / Shortage:** $0.00 (todos los clientes atendidos dentro de su horario y sin exceder capacidades).

### Rutas y Asignaciones Óptimas:
*   **Camión T1 (Costo Fijo: $500, Capacidad: C0=8.000, C1=7.000$):**
    *   **Ruta:** Depósito $\rightarrow$ Estación 1 $\rightarrow$ Estación 2 $\rightarrow$ Estación 4 $\rightarrow$ Depósito.
    *   **Distancia Recorrida:** 85 km.
    *   **Asignación de Compartimentos:** C0 = Regular | C1 = Diésel.
    *   **Cargas iniciales al salir del depósito:** C0 = 8.000 L | C1 = 6.000 L.
    *   **Llegadas estimadas:**
        *   Estación 1: 05:20 (espera a apertura de ventana [06:00 - 10:00]).
        *   Estación 2: 06:40 (espera a apertura de ventana [07:00 - 12:00]).
        *   Estación 4: 07:45 (dentro de ventana [06:00 - 11:00]).

*   **Camión T2 (Costo Fijo: $400, Capacidad: C0=6.000, C1=9.000$):**
    *   **Ruta:** Depósito $\rightarrow$ Estación 3 $\rightarrow$ Depósito.
    *   **Distancia Recorrida:** 30 km.
    *   **Asignación de Compartimentos:** C0 = Diésel | C1 = Regular.
    *   **Cargas iniciales al salir del depósito:** C0 = 3.000 L | C1 = 2.500 L.
    *   **Llegadas estimadas:**
        *   Estación 3: 05:45 (espera a apertura de ventana [08:00 - 14:00]).

---

## 🔍 Análisis de la Solución del Operador

El informe evalúa la factibilidad y el costo real de la ruta propuesta empíricamente por el operador de transporte, la cual consistía en:
*   **T1:** Depósito $\rightarrow$ Estación 1 $\rightarrow$ Estación 3 $\rightarrow$ Depósito (C0 = Regular, C1 = Diésel).
*   **T2:** Depósito $\rightarrow$ Estación 2 $\rightarrow$ Estación 4 $\rightarrow$ Depósito (C0 = Diésel, C1 = Regular).

### Errores y Faltas de Factibilidad Identificados:
1.  **Costo de Distancia Incorrecto:** El operador declaró un costo por distancia de \$260 (130 km). Sin embargo, el cálculo de las distancias reales de estas rutas da como resultado **150 km** (60 km para T1 y 90 km para T2). A \$2/km, el costo real de la distancia es **\$300**. El costo total real de su propuesta asciende a **\$1.200** (en lugar de los \$1.160 declarados).
2.  **Infracción de Estabilidad de Carga:** En el camión T2, al salir del depósito los *fill ratios* son de 0.6667 (C0) y 0.5556 (C1). Luego de realizar la entrega en la Estación 2, los *fill ratios* restantes son de **0.4167 (C0)** y **0.1111 (C1)**. Esto resulta en una diferencia de **0.3056**, lo cual **supera el límite de imbalance tolerado de $\Delta = 0.30$**, provocando inestabilidad física en el camión durante el trayecto hacia la Estación 4.

**Conclusión:** La solución del operador es **parcialmente infactible** y económicamente subóptima comparada con la solución calculada por la metaheurística.

---

## 🎲 Extensión Estocástica del Modelo

Para dotar al modelo de resiliencia ante la incertidumbre, se propuso una extensión estocástica de **Programación en Dos Etapas con Recurso** (*Two-Stage Stochastic Programming*). Se consideran tres escenarios probables de demanda: **Normal (s1)**, **Baja (s2)** y **Alta (s3)** con sus respectivas probabilidades de ocurrencia $p_s$.

### Clasificación de Decisiones por Etapa
*   **Primera Etapa (Antes de conocer la demanda real):**
    *   Activación de camiones ($y_k$).
    *   Rutas y arcos de viaje ($x_{ijk}$).
    *   Asignación de producto a compartimentos ($z_{kcp}$).
    *   Carga inicial en compartimentos ($L_{kc0}$).
    *   Tiempos programados de llegada y makespan en depósito.
*   **Segunda Etapa (Recurso - Una vez revelada la demanda del escenario $s$):**
    *   Carga remanente en ruta por escenario ($L_{kcjs}$).
    *   Cantidad física de producto entregada en cada estación ($q_{kcpjs}$).
    *   Demanda no satisfecha en cada estación (shortage) por escenario ($s_{jps}$).

### Función Objetivo Estocástica
Minimiza el costo determinista de la primera etapa sumado al valor esperado de las penalizaciones por demanda insatisfecha en la segunda etapa:
$$\min Z = \sum_{k \in K} F_k y_k + c_d \sum_{k \in K} \sum_{i \in V} \sum_{\substack{j \in V \\ j \neq i}} d_{ij} x_{ijk} + \sum_{s \in S} p_s \left( c_s \sum_{j \in N} \sum_{p \in P} s_{jps} \right)$$

Esta formulación permite que el depósito cargue volúmenes iniciales robustos que equilibren el costo de transportar combustible extra versus las penalizaciones esperadas por no satisfacer la demanda bajo escenarios de consumo alto.

---

## 🚀 Cómo Ejecutar la Simulación

### Requisitos Previos
Es necesario disponer de un entorno con Python 3 e instalar las siguientes bibliotecas básicas de ciencia de datos:

```bash
pip install jupyter pandas matplotlib
```

### Ejecución
1.  Clona o descarga este repositorio en tu máquina local.
2.  Abre una terminal en el directorio del proyecto y arranca Jupyter Notebook:
    ```bash
    jupyter notebook
    ```
3.  Abre el archivo `simulated_annealing_combustibles.ipynb`.
4.  Ejecuta todas las celdas de forma secuencial (`Run All`) para:
    *   Cargar los datos del problema y declarar las funciones de evaluación y penalización.
    *   Correr la metaheurística de Simulated Annealing con la semilla por defecto.
    *   Imprimir la mejor solución en formato texto y tabla de costos.
    *   Generar los gráficos de convergencia de temperatura y costo a lo largo de las iteraciones.
