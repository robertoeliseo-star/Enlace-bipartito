# Enlace-bipartito-de-trenes
Se analiza la posible relación de rutas de trenes con zona de acuerdo a paramétros como la demanda, la cantidad de estaciones, la distancia de la ruta y el un factor socioeconómico obtenido apartir del INEGI. Se hace la comparción entre un módelo clásico y uno cuántico y se obtienen la preponderancia de la ruta según los paramétros antes dichos.
# Proyecto QUBO-QAOA: matching bipartito 4x4

## Dataset

* **Nombre del dataset:** Matriz de Priorización de Infraestructura Ferroviaria (ZVM-Bajío-Hidalgo)
* **Fuente oficial o confiable:** Agencia Reguladora del Transporte Ferroviario (ARTF), INEGI (Censos Económicos) y datos operativos históricos de la extensión del Tren Suburbano.
* **Institución responsable:** Secretaría de Infraestructura, Comunicaciones y Transportes (SICT) / Instituto Nacional de Estadística y Geografía (INEGI).
* **URL de la fuente:** https://www.gob.mx/artf | https://www.inegi.org.mx
* **URL raw del CSV** usado en data/: Se obtiene en la carpeta rutas_ferroviarias.xlsx
* **Licencia o condiciones de uso:** Datos de uso público abierto con fines de investigación académica.
* **Fecha de consulta:** Junio 2026
* **Dominio del problema:** Optimización de transporte, planeación de infraestructura ferroviaria y asignación de recursos socioeconómicos.



## Metodología de Estimación de Datos Socioeconómicos

Dado que el INEGI no calcula el Producto Interno Bruto (PIB) directamente por municipio de manera anual ordinaria (sino que lo hace centralizado para las 32 entidades federativas), la aproximación económica de los corredores ferroviarios se obtuvo mediante un **método de desagregación y distribución del valor añadido espacial** empleando datos censales locales.

### 📐 Algoritmo de Distribución en 4 Pasos

#### Paso 1: Mapeo y delimitación geográfica de las rutas
Se identificaron detalladamente todos los municipios que la infraestructura ferroviaria o carretera de la ruta atraviesa físicamente, delimitando así el **corredor de influencia económica directa** para cada una de las 4 opciones de matching.

#### Paso 2: Obtención del Peso Municipal ($Factor_{m}$)
Utilizando el Sistema Automatizado de Información Censal (SAIC), se extrajo y sumó el **Valor Agregado Censal Bruto (VACB)** de los municipios mapeados en la ruta, dividiéndolo entre el VACB total de su respectiva entidad federativa. Esto define la concentración económica exacta de la ruta frente a su estado .

$$Factor_{m} = \frac{\sum \text{VACB}_{\text{municipios de la ruta}}}{\text{VACB}_{\text{Total del Estado}}}$$

#### Paso 3: Estimación del PIB Municipal Equivalente
El factor porcentual o ponderador espacial obtenido en el paso anterior se multiplicó por el **Producto Interno Bruto por Entidad Federativa (PIBE)** oficial del estado a precios corrientes para mapear el valor en unidades monetarias corrientes .

$$\text{PIB}_{\text{Ruta Fragmento}} = Factor_{m} \times \text{PIBE}_{\text{Oficial}}$$

#### Paso 4: Consolidación interestatal y agregación final
Para los corredores que atraviesan más de un estado (como los ejes hacia Querétaro y Guanajuato), se ejecutó el cálculo de forma aislada para cada entidad federativa y finalmente se sumaron todos los subtotales para arrojar la estimación del **PIB global anual de la ruta** ($\sum \text{PIB}_{\text{Ruta Fragmento}}$).

---

### 📊 Desglose Analítico y Resultados por Ruta

La siguiente tabla resume los criterios y los valores de PIB consolidado anual resultantes de la metodología de desagregación:

| ID | Ruta Ferroviaria | Municipios / Alcaldías Mapeadas | Criterio de Análisis Económico | PIB Estimado Consolidado |
| :---: | :--- | :--- | :--- | :---: |
| **A1** | **Buenavista - AIFA** | Cuauhtémoc, Azcapotzalco (CDMX); Tlalnepantla, Tultitlán, Tecámac (Edomex). | Absorbe la alcaldía corporativa más importante del país (Cuautémoc genera el $8.5\%$ del VACB nacional) y el nodo industrial del norte del Valle de México $[1, 2]$. | **$1.12 Billones de MXN**<br>($1,120,000 MDP) |
| **A2** | **Querétaro - Irapuato** | Santiago de Querétaro (Qro); Celaya, Salamanca, Irapuato (Gto). | Concentración masiva del sector automotriz y el complejo energético de la Refinería de Salamanca. La capital queretana aporta cerca del $60\%$ de su PIB estatal $[1]$. | **$890,000 MDP** |
| **A3** | **Huehuetoca - Querétaro** | Huehuetoca (Edomex); Tula de Allende (Hgo); San Juan del Río, Santiago de Querétaro (Qro). | Eje de alta densidad logística ferroviaria. Suma el valor industrial/cementero de Tula con parques industriales clave dedicados a la distribución hacia el norte del país. | **$780,000 MDP** |
| **A4** | **Xaltocan II - Pachuca** | Nextlalpan, Tizayuca, Pachuca de Soto (Hgo). | Zona de desarrollo periférico reciente vinculada al AIFA. Presenta menor densidad corporativa en comparación con el Bajío, impulsada por comercio y servicios urbanos $[1]$. | **$210,000 MDP** |

---

### 📚 Fuentes de Referencia y Citación

* **$[1]$ Para los Datos Municipales (VACB):** *Instituto Nacional de Estadística y Geografía (INEGI).* (2025). Resultados Definitivos de los Censos Económicos, Sistema Automatizado de Información Censal (SAIC). Aguascalientes, México: INEGI. Disponible en: [inegi.org.mx](https://www.inegi.org.mx)
  
* **$[2]$ Para el PIB Estatal (PIBE):** *Instituto Nacional de Estadística y Geografía (INEGI).* (2025). Producto Interno Bruto por Entidad Federativa (PIBE), Cuentas Nacionales de México. Aguascalientes, México: INEGI. Disponible en: [inegi.org.mx](https://www.inegi.org.mx)



---

## Modelado

* **Conjunto A:** Rutas de tren disjuntas con origen/conexión en el Valle de México y Bajío.
  * $a_1$: Buenavista - AIFA
  * $a_2$: Querétaro - Irapuato
  * $a_3$: Huehuetoca - Querétaro
  * $a_4$: Xaltocan II - Pachuca
* **Criterio para elegir exactamente 4 elementos de A:** Se seleccionaron 4 proyectos ferroviarios clave del centro del país con trazados geográficamente independientes para evaluar su viabilidad y peso en el sistema de transporte nacional.
* **Conjunto B:** Corredores socioeconómicos de impacto (Destinos).
  * $b_1$: Corredor Conurbado Norte (Zumpango-Cuautitlán)
  * $b_2$: Macrorregión Industrial del Bajío (Querétaro)
  * $b_3$: Nodo Logístico y Aeroportuario Extranjero (AIFA)
  * $b_4$: Corredor Metropolitano de la Frontera Hidalgo (Pachuca)
* **Criterio para elegir exactamente 4 elementos de B:** Se seleccionaron las zonas de influencia geoeconómica directa e inmediata de cada una de las terminales de las rutas asignadas en el Conjunto A.
* **Definición de $x_{ij} = 1$:** Indica que la ruta $a_i$ se asigna de manera prioritaria para el desarrollo y cobertura del corredor socioeconómico $b_j$.
* **Interpretación de $x_{ij} = 0$:** La ruta $a_i$ no se vincula al desarrollo del corredor $b_j$.

---

## Matriz de score

* **Columnas usadas:** Demanda anual base, N° de estaciones, Distancia de la ruta (km), Factor Socioeconómico.
* **Fórmula exacta de $S_{ij}$:** Para elementos de la diagonal principal ($i = j$):

  $$S_{ii} = \text{Demanda}_i \times \left( \alpha \cdot \frac{\text{Estaciones}_i}{\text{Distancia}_i} + \beta \cdot \text{Factor Socioeconómico}_i \right)$$
  
  Para elementos fuera de la diagonal ($i \neq j$):
  $$S_{ij} = 0.0$$
  
  (Parámetros del cálculo actual: $\alpha = 0.6$, $\beta = 0.4$)
  
* **Normalización aplicada:** Escalamiento lineal multi-criterio combinando la densidad física de estaciones ($\text{estaciones}/\text{km}$) con un índice ponderado del impacto del PIB regional (Factor Socioeconómico).
* **Matriz S 4x4 (Valores de simulación):**
  
$$
\begin{pmatrix} 
52515.78 & 0.00 & 0.00 & 0.00 \\
0.00 & 5460.74 & 0.00 & 0.00 \\ 
0.00 & 0.00 & 131.46 & 0.00 \\ 
0.00 & 0.00 & 0.00 & 6019.25 
\end{pmatrix}
$$

---

## Restricciones

* **Restricción por filas:** Cada ruta $a_i$ puede ser asignada a lo más a un corredor socioeconómico $b_j$ ($\sum_j x_{ij} = 1$).
* **Restricción por columnas:** Cada corredor $b_j$ debe recibir exactamente una ruta de conectividad prioritaria ($\sum_i x_{ij} = 1$).
* **Otras restricciones, si existen:** Ninguna adicional (Matching biunívoco perfecto).
* **Justificación de por qué el problema es matching bipartito:** Cumple la estructura matemática clásica de asignación lineal de costo/beneficio, donde dos conjuntos disjuntos (Rutas y Corredores) deben emparejarse de forma uno a uno minimizando o maximizando un score total, bajo penalizaciones de exclusión mutua.
* **Justificación de por qué es razonable modelarlo como QUBO:** Porque las restricciones estrictas de igualdad del matching se pueden incorporar al Hamiltoniano de costo como penalizaciones cuadráticas de la forma $P \cdot (\sum x - 1)^2$. Esto mapea el problema lineal restringido en un espacio binario no restringido apto para optimizadores cuánticos (Quantum Annealing) o algoritmos variacionales (QAOA).

---

## Resultados

* **Solución clásica exacta:**
  * **Mejor energía clásica:** `-64127.23418093729` (Formulación de minimización de la energía, equivalente a $- \sum S_{ii}x_{ii}$).
  * **Mejor score clásico:** `64127.23418093729`
  * **Mejor asignación clásica:**
    * $x_{00} = 1$ (Buenavista-AIFA $\rightarrow$ Corredor Conurbado Norte) | Score: `52515.78`
    * $x_{11} = 1$ (Querétaro-Irapuato $\rightarrow$ Macrorregión Industrial) | Score: `5460.74`
    * $x_{22} = 1$ (Huehuetoca-Querétaro $\rightarrow$ Nodo Logístico) | Score: `131.46`
    * $x_{33} = 1$ (Xaltocan II-Pachuca $\rightarrow$ Corredor Metropolitano) | Score: `6019.25`
* **Resultado QAOA local:**
  * **Energía esperada:** `783544.1231946987`
  * **Mejor energía observada:** `-64127.23418093729`
  * **Probabilidad ideal de factibilidad:** `2.63%` (`0.02636`)
  * **Probabilidad ideal del óptimo clásico:** `0.11%` (`0.00115`)
* **Comparación clásico vs QAOA local:** El algoritmo QAOA ejecutado localmente logró encontrar el estado fundamental óptimo en su mejor observación (Energía `-64127.23`), igualando con precisión el resultado del solver clásico exacto. No obstante, debido a la naturaleza estocástica del algoritmo y la penalización de las restricciones en el Hamiltoniano, la probabilidad ideal de colapsar directamente en el óptimo global es del 0.11%, lo que subraya la complejidad del paisaje energético cuántico para problemas combinatorios acotados.
---

## Ética y limitaciones

* **Riesgos éticos:** Un uso irresponsable o mal calibrado de este algoritmo podría desplazar la inversión de infraestructura pública de transporte hacia zonas de alta demanda inmediata, marginando comunidades en desarrollo o de bajos recursos basándose puramente en un criterio financiero de rentabilidad o PIB.
* **Medidas de mitigación:** Se incorporó un factor socioeconómico balanceado con pesos dinámicos ($\alpha$ y $\beta$). Esto permite a los tomadores de decisiones ajustar el modelo para dar prioridad al impacto social o accesibilidad local sobre los puros volúmenes masivos de pasajeros.
* **Limitaciones del modelo:** El modelo asume rutas estáticas y zonas completamente disjuntas. En un entorno ferroviario real, existen intersecciones de vías, cuellos de botella por tráfico compartido, horarios y dependencias dinámicas que un matching bipartito estricto de $4 \times 4$ no puede capturar en su totalidad.

---

## Ejecución

### Instrucciones para abrir el archivo .ipynb en Google Colab:
1. Dirígete a [Google Colab](https://colab.research.google.com).
2. Selecciona la pestaña **GitHub**.
3. Pega la URL de este repositorio y selecciona el archivo `.ipynb` del proyecto.

### Instrucciones para ejecutar todas las celdas sin errores:
1. Una vez abierto el notebook, ve al menú superior y selecciona **Entorno de ejecución > Cambiar tipo de entorno de ejecución**, asegurándote de contar con los recursos estándar de Python 3.
2. Sube el archivo `rutas_ferroviarias.xlsx` que se encuentra en la carpeta `data/` al menú lateral de archivos de Colab (o clona el repositorio directamente usando `!git clone`).
3. Ejecuta la primera celda dedicada a la instalación de dependencias requeridas (ej. `!pip install qiskit pandas numpy`).
4. Selecciona **Entorno de ejecución > Ejecutar todas las celdas** (`Ctrl + F9`). Las pruebas automáticas (`assert`) validarán las dimensiones de los DataFrames y la matriz de pesos sin detener la ejecución.
