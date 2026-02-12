# SEED Algorithm: Socio-Economic and Environmental Distribution

> [!IMPORTANT]
> [cite_start]**🏆 Algoritmo Ganador:** Este proyecto ha sido galardonado con el **Excellence Award** de **Akademia Future Builders**, destacando por su precisión técnica y su impacto en la resolución de desafíos sociales reales[cite: 5].

## Descripción General

[cite_start]**SEED** (*Socio-Economic and Environmental Distribution*) es un modelo de optimización espacial diseñado para seleccionar y distribuir **1,000 ubicaciones óptimas** para residencias de mayores en España[cite: 1, 8]. [cite_start]Desarrollado por el **Equipo Sandía**, el algoritmo transforma la expansión de infraestructuras de cuidado en una decisión basada en datos, equilibrando la viabilidad económica con la responsabilidad social[cite: 3, 4, 19].

## El Desafío Social

España enfrenta uno de los mayores desafíos socioeconómicos del siglo XXI:
* [cite_start]**Proyección 2050**: Se estima que más del **30% de la población** superará los 65 años[cite: 7, 12].
* [cite_start]**Déficit de Infraestructura**: Actualmente existe un déficit habitacional crítico, con más de **13,800 plazas faltantes** solo en la región de Galicia[cite: 7, 90].
* [cite_start]**Visión Estratégica**: El algoritmo entiende que el cliente no es solo el residente, sino el sistema familiar que busca alivio y tranquilidad ante la saturación de cuidados[cite: 17, 18].

---

## Arquitectura del Algoritmo

[cite_start]SEED procesa más de **36,000 secciones censales** del INE a través de un modelo multicriterio compuesto por cuatro capas jerárquicas[cite: 9, 21, 22].

### 1. Base Territorial (Capa 1)
[cite_start]Define el espacio de decisión utilizando coordenadas geográficas (latitud, longitud) y códigos administrativos a nivel de sección censal para garantizar la máxima precisión[cite: 22, 23].

### 2. Dimensiones y Pesos de Decisión

| Capa | Variable Clave | Peso | Descripción |
| :--- | :--- | :--- | :--- |
| **Demanda Residencial** | F-of-M, Dependencia, Densidad | **45%** | [cite_start]Evalúa el potencial de clientes locales y logística urbana[cite: 24]. |
| **Viabilidad Económica**| Renta Media del Hogar | **40%** | [cite_start]Proxy de capacidad de pago para asegurar morosidad nula[cite: 33]. |
| **Saturación Territorial**| Oferta/Demanda Provincial | **15%** | [cite_start]Factor de corrección para evitar mercados sobrepoblados[cite: 45]. |

---

## Lógica Matemática de las Capas

### Capa de Demanda Residencial (Peso: 0.45)
[cite_start]Calculada mediante tres indicadores normalizados[cite: 24]:
* [cite_start]**Figure of Merit (F-of-M)** (65%): Métrica propietaria sobre la idoneidad de la pirámide poblacional[cite: 25].
* [cite_start]**Índice de Dependencia** (10%): Proporción de población con Grado de Dependencia III[cite: 27].
* [cite_start]**Densidad Poblacional** (25%): Prioriza zonas con mayor concentración de recursos y accesibilidad[cite: 30, 31].

### Capa de Viabilidad Económica (Peso: 0.40)
[cite_start]A diferencia de otros modelos, SEED aplica una **función asimétrica** basada en el coste medio de una plaza (aprox. 2,100€/mes) para optimizar la selección de rentas[cite: 34, 44]:

[cite_start]$$Score(Renta) = \begin{cases} 0 & \text{si } Renta < 32,000 \\ \frac{Renta - 32,000}{40,000} \cdot 0.7 & \text{si } 32,000 \leq Renta \leq 72,000 \\ 1 + 0.5 \cdot \frac{Renta - 72,000}{32,000} & \text{si } Renta > 72,000 \end{cases}$$ [cite: 35-43]

### Capa de Saturación (Peso: 0.15)
[cite_start]Calcula el cociente entre las plazas existentes y la población mayor de 80 años por provincia[cite: 46]. [cite_start]A menor saturación, mayor es el atractivo de la ubicación (puntuación invertida)[cite: 47, 48].

---

## Restricción Espacial: Clustering Adaptativo

[cite_start]Para evitar la **canibalización** entre centros, SEED implementa un algoritmo *greedy* iterativo que garantiza distancias mínimas según la densidad de la zona[cite: 53, 54, 68]:

* [cite_start]**Zonas de Alta Densidad** (>5,000 hab/km²): $d_{min} = 1.5\text{ km}$[cite: 69].
* [cite_start]**Zonas de Densidad Media** (1,000-5,000 hab/km²): $d_{min} = 2.5\text{ km}$[cite: 70].
* [cite_start]**Zonas Rurales** (<1,000 hab/km²): $d_{min} = 5.0\text{ km}$[cite: 71].

[cite_start]Se utiliza la **fórmula de Haversine** para calcular distancias geodésicas precisas sobre la superficie terrestre[cite: 55, 67].

---

## Validación y Resultados

El modelo ha sido validado con métricas de alta fidelidad:
* [cite_start]**Correlación Sectorial**: Presenta un ajuste de $r = 0.882$ respecto a la distribución real del mercado residencial[cite: 10, 91].
* [cite_start]**Ubicación Top**: La sección censal `1503003001` (A Coruña) obtuvo el score más alto (0.838) debido a su equilibrio perfecto entre renta (74,388€) y baja saturación[cite: 94, 95].
* [cite_start]**Concentración Estratégica**: Las regiones con mayor potencial identificado son **Galicia, C. Valenciana, Andalucía, Cataluña y Madrid**[cite: 87, 88].
* [cite_start]**Perfil del Éxito**: El Top 50 de ubicaciones promedia una renta de **72,769.89€** y un F-of-M de **0.1194**, alineándose con el "punto ideal" del modelo[cite: 92].

---

## Stack Tecnológico

* **Lenguaje**: Python 3.x
* **Análisis de Datos**: `pandas`, `numpy`, `openpyxl`.
* **Cálculo Espacial**: `scipy` (Haversine), `scikit-learn` (MinMaxScaler).
* **Visualización**: `folium` (Mapas de calor e interactivos), `matplotlib`.

## Ejecución

1. **Preparación**: Cargar el archivo `VARIABLES SEED.xlsx` con los datos del INE.
2. **Scoring**: Ejecutar el cálculo de las 4 capas normalizadas.
3. **Clustering**: Aplicar el filtrado de distancia adaptativa para obtener el ranking de las 1,000 mejores ubicaciones.
4. **Visualización**: Generar mapas HTML interactivos para análisis de microlocalización.

---
© 2026 Equipo Sandía - Proyecto Ganador del Excellence Award en Akademia Future Builders.

