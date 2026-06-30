# Proyecto QUBO-QAOA: Matching Bipartito 4×4 para Reconversión Laboral en Baja California

## Descripción del proyecto

Este proyecto aborda un problema de reconversión laboral en Baja California mediante técnicas de optimización combinatoria y computación cuántica. El escenario considerado consiste en identificar una asignación óptima entre ocupaciones que han experimentado una disminución significativa en el número de empleos y ocupaciones que presentan crecimiento y capacidad de absorción laboral.

El problema se modela como un matching bipartito 4×4, donde cada ocupación del conjunto de origen debe asignarse exactamente a una ocupación del conjunto de destino. El objetivo es maximizar la afinidad entre los perfiles ocupacionales y favorecer transiciones laborales que requieran un menor esfuerzo de adaptación y, al mismo tiempo, representen una mejora o una pérdida mínima en el ingreso económico de los trabajadores.

Para construir la compatibilidad entre ocupaciones se utilizaron datos públicos de Data México correspondientes al cuarto trimestre de 2022, así como atributos heurísticos relacionados con el esfuerzo físico, la interacción social y la destreza técnica requerida por cada actividad. Estos atributos permiten estimar qué tan viable resulta una transición laboral entre distintos sectores económicos.

La formulación matemática se implementó como un problema QUBO (Quadratic Unconstrained Binary Optimization), lo que permite resolverlo tanto mediante métodos clásicos exactos como mediante el algoritmo QAOA (Quantum Approximate Optimization Algorithm). El proyecto tiene fines exclusivamente educativos y busca ilustrar el potencial de las técnicas de optimización cuántica para abordar problemas reales de asignación.

---

## Formulación del problema

El problema se plantea como un matching bipartito entre dos conjuntos disjuntos de ocupaciones:

* **Conjunto A (emisores):** ocupaciones que presentan una pérdida significativa de empleos y cuyos trabajadores requieren alternativas de reconversión laboral.
* **Conjunto B (receptores):** ocupaciones con crecimiento en la demanda y capacidad potencial para absorber trabajadores provenientes de otros sectores.

La variable binaria de decisión se define como:

```math
x_{ij} =
\begin{cases}
1, & \text{si la ocupación } A_i \text{ se asigna a } B_j,\\
0, & \text{en otro caso}.
\end{cases}
```

El objetivo consiste en maximizar:

```math
\max \sum_{i=1}^{4}\sum_{j=1}^{4} S_{ij}x_{ij}
```

donde ($S_{ij}$) representa el score de compatibilidad entre las ocupaciones.

Las restricciones del problema garantizan un matching uno-a-uno:

```math
\sum_j x_{ij}=1
```

```math
\sum_i x_{ij}=1
```

La formulación final se expresa como un problema QUBO, incorporando las restricciones mediante penalizaciones cuadráticas.

---

# Dataset

**Nombre del dataset:**

Reconversión laboral 4×4 construida a partir de datos ocupacionales de Baja California (2022-T4).

**Fuente oficial o confiable:**

Data México.

**Institución responsable:**

Secretaría de Economía del Gobierno de México y Datawheel.

**URL de la fuente:**

https://datamexico.org/

**URL RAW del CSV usado en `data/`:**

```text
https://github.com/JAGFraga/proyecto-qubo-qaoa-empleo-bc/tree/main/data/dataset_real_4x4.csv

```

**Licencia o condiciones de uso:**

Los datos utilizados provienen de fuentes públicas agregadas disponibles en Data México y se emplean exclusivamente con fines educativos y de investigación.

**Fecha de consulta:**

Junio de 2026.

**Dominio del problema:**

Reconversión laboral y asignación óptima de trabajadores provenientes de sectores en crisis hacia sectores con crecimiento y capacidad de absorción de empleo.

---

# Modelado

## Conjunto A

Sectores con pérdida significativa de empleos:

| ID   | Ocupación                                     | Pérdida de empleos | Salario mensual (MXN) |
| ---- | --------------------------------------------- | -----------------: | --------------------: |
| 7121 | Albañiles, Mamposteros y Afines               |            -12,457 |              7,435.90 |
| 5313 | Vigilantes y Guardias en Establecimientos     |             -8,373 |              5,949.27 |
| 5112 | Fonderos, Vendedores y Comerciantes de Comida |             -8,143 |              4,135.52 |
| 4224 | Vendedores por Catálogo                       |             -5,560 |              1,924.62 |

### Criterio para elegir exactamente 4 elementos de A

Se seleccionaron las cuatro ocupaciones operativas con mayor destrucción neta de empleo según la variable **Workforce Growth Value** para Baja California en el cuarto trimestre de 2022.

---

## Conjunto B

Sectores con crecimiento y capacidad de absorción laboral:

| ID   | Ocupación                                                      | Nuevos empleos | Salario mensual (MXN) |
| ---- | -------------------------------------------------------------- | -------------: | --------------------: |
| 4211 | Empleados de Ventas, Despachadores y Dependientes en Comercios |        +17,177 |              4,310.62 |
| 4111 | Comerciantes en Establecimientos                               |         +8,395 |              4,666.57 |
| 9411 | Ayudantes en la Preparación de Alimentos                       |         +7,069 |              4,600.36 |
| 8101 | Supervisores de Operadores de Maquinaria Industrial            |         +4,204 |              6,618.55 |

### Criterio para elegir exactamente 4 elementos de B

Se seleccionaron las cuatro ocupaciones con mayor creación neta de empleos y perfiles potencialmente alcanzables mediante procesos de capacitación o reconversión laboral.

---

### Definición de $x_{ij} = 1$

($x_{ij}=1$) significa que la ocupación emisora ($A_i$) se asigna a la ocupación receptora ($B_j$).

### Interpretación de $x_{ij} = 0$

($x_{ij}=0$) significa que la ocupación ($A_i$) no se asigna a la ocupación ($B_j$).

---

# Matriz de score

## Columnas usadas

* Workforce
* Monthly Wage
* Esfuerzo físico (heurístico)
* Interacción social (heurístico)
* Destreza técnica (heurístico)

La variable **Monthly Wage Growth** fue eliminada para simplificar el modelo final.

---

## Fórmula exacta de $S_ij$

```math
S_{ij}=0.8(1-D_{ij})+0.2G_{ij}
```

donde:

* ($D_{ij}$): distancia entre perfiles ocupacionales.
* ($G_{ij}$): ganancia salarial normalizada.

---

## Normalización aplicada

Los atributos físico, social y técnico fueron normalizados en el intervalo ([0,1]).

La ganancia salarial se normalizó mediante escalamiento min-max:

```math
G_{ij}=
\frac{\Delta salario_{ij}-\min(\Delta salario)}
{\max(\Delta salario)-\min(\Delta salario)}
```

La distancia entre perfiles se calculó mediante distancia euclidiana sobre los atributos normalizados.

---

## Matriz S 4×4

```text
         B1      B2      B3      B4
A1      7.5	   -3.0	   -0.5	    3.0
A2     -3.0	    7.5	   -0.5	    3.0
A3     -0.5	   -0.5	    3.5	   -1.0
A4      3.0	    3.0	   -1.0	    3.5

```

---

# Restricciones

## Restricción por filas

Cada ocupación emisora debe asignarse exactamente a una ocupación receptora:

```math
\sum_j x_{ij}=1
```

---

## Restricción por columnas

Cada ocupación receptora debe recibir exactamente una ocupación emisora:

```math
\sum_i x_{ij}=1
```

---

## Otras restricciones

No se consideraron restricciones adicionales relacionadas con escolaridad, certificaciones, movilidad geográfica o capacidad máxima de absorción laboral.

---

## Justificación de por qué el problema es matching bipartito

El problema involucra dos conjuntos disjuntos de ocupaciones: sectores en crisis y sectores en expansión. Cada elemento de un conjunto puede relacionarse con exactamente un elemento del otro conjunto, lo que corresponde naturalmente a un matching bipartito uno-a-uno.

---

## Justificación de por qué es razonable modelarlo como QUBO

El problema utiliza variables binarias y restricciones cuadráticas de igualdad que pueden incorporarse mediante penalizaciones dentro de una función objetivo única. Esto permite resolverlo tanto por algoritmos clásicos exactos como por métodos cuánticos variacionales como QAOA.

---

# Resultados

## Solución clásica exacta

| Ocupación emisora                     | Ocupación receptora                       | Score |
| ------------------------------------- | ----------------------------------------- | ----: |
| A1: Albañiles y Mamposteros           | B4: Supervisores de Maquinaria Industrial | 0.419 |
| A2: Vigilantes y Guardias             | B3: Ayudantes en Preparación de Alimentos | 0.663 |
| A3: Fonderos y Comerciantes de Comida | B2: Comerciantes en Establecimientos      | 0.716 |
| A4: Vendedores por Catálogo           | B1: Empleados de Ventas                   | 0.822 |

Score óptimo:

```text
2.620
```

La solución cumple completamente todas las restricciones del matching bipartito.

---

## Resultado QAOA local

```text
Mejor energía clásica: -2.620
Mejor energía observada por QAOA: -2.620
```

Probabilidad total de soluciones factibles:

```text
0.02059 (2.06%)
```

Probabilidad de la solución óptima:

```text
0.00087 (0.087%)
```

---

## Comparación clásico vs QAOA local

QAOA local recuperó exactamente la misma solución óptima obtenida mediante búsqueda exhaustiva clásica, validando la formulación QUBO implementada para este caso de estudio.

---

## Hardware real o pipeline híbrido

No se utilizaron dispositivos cuánticos reales ni mecanismos adicionales de reparación clásica. Todas las pruebas se realizaron mediante simulación local utilizando Qiskit.

---

# Ética y limitaciones

## Riesgos éticos

* Interpretar los resultados como recomendaciones reales de política pública.
* Introducir sesgos derivados de la selección de variables y perfiles ocupacionales.
* Ignorar factores humanos y sociales relevantes.
* Asumir que la similitud matemática implica viabilidad laboral real.

---

## Medidas de mitigación

* Uso de datos públicos y agregados.
* Documentación explícita de los atributos heurísticos utilizados.
* Fines exclusivamente educativos y de investigación.
* Ausencia de decisiones automatizadas sobre personas reales.

---

## Limitaciones del modelo

* El problema se limita a un esquema 4×4.
* Los atributos físico, social y técnico fueron construidos mediante juicio experto y no provienen de fuentes oficiales.
* No se consideran certificaciones, escolaridad, experiencia laboral ni movilidad geográfica.
* Las restricciones se simplificaron a un matching uno-a-uno.

Si el tamaño del problema creciera, el número de variables binarias y el espacio de búsqueda aumentarían exponencialmente, haciendo necesario el uso de estrategias híbridas y hardware cuántico más avanzado.

---

# Ejecución

## Abrir el notebook en Google Colab

1. Abrir el repositorio en GitHub.
2. Seleccionar el archivo `proyecto_qubo_qaoa.ipynb`.
3. Hacer clic en **Open in Colab**.

---

## Ejecutar todas las celdas sin errores

1. Verificar que exista el archivo:

```text
data/dataset_real_4x4.csv
```

2. Ejecutar:

```text
Runtime → Run all
```

3. Esperar la instalación automática de dependencias y la ejecución completa del notebook.

4. Verificar que los resultados obtenidos coincidan con los reportados en este README.
