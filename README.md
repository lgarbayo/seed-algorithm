# SEED Algorithm: Socio-Economic and Environmental Distribution

> [!IMPORTANT]
> 🏆 **Algoritmo Ganador**: Este proyecto ha sido galardonado con el **Excellence Award de Akademia Future Builders**, destacando por su precisión técnica y su impacto en la resolución de desafíos sociales reales.
> > [!IMPORTANT]
> El notebook del algoritmo se encuentra en /seed-algorithm/notebooks/03_seed_algorithm.ipynb
> > [!IMPORTANT]
> El informe técnico relacionado con el desarrollo del algoritmo se encuentra en /seed-algorithm/SEED_tech_report.pdf

## 1. Descripción General

**SEED** (Socio-Economic and Environmental Distribution) es un algoritmo de optimización espacial diseñado para seleccionar y distribuir 1,000 ubicaciones óptimas para residencias de mayores en España. El modelo transforma la expansión de infraestructuras de cuidado en una decisión basada en datos, equilibrando la viabilidad económica con la responsabilidad social.

## 2. El Desafío Social y Demográfico

- **Envejecimiento proyectado**: Para 2050, se estima que más del 30% de la población española superará los 65 años.
- **Déficit de plazas**: Existe una carencia crítica; solo en Galicia se identifica un déficit superior a las 13,800 plazas.
- **Público Objetivo**: El algoritmo entiende que el cliente no es solo el residente, sino el sistema familiar completo que busca alivio y tranquilidad.

## 3. Arquitectura del Algoritmo

El algoritmo SEED se estructura en **cuatro capas jerárquicas** que procesan más de 36,000 secciones censales del INE:

### Capa 1: Base Territorial

Define el espacio de decisión utilizando la sección censal como unidad geográfica básica, incluyendo coordenadas (latitud, longitud) y códigos administrativos.

### Capa 2: Demanda Residencial (Peso: 45%)

Evalúa el potencial de demanda local mediante tres indicadores:

- **Figure of Merit (F-of-M) (65%)**: Métrica propietaria que evalúa la idoneidad demográfica según la pirámide poblacional ideal.
- **Índice de Dependencia (10%)**: Proporción de población con Grado de Dependencia III.
- **Densidad Poblacional (25%)**: Factor clave para la logística y accesibilidad urbana.

**Fórmula de la capa:**
```
Demanda = 0.65 × hogares_norm + 0.10 × dependencia_norm + 0.25 × densidad_norm
```

### Capa 3: Viabilidad Económica (Peso: 40%)

Utiliza la renta media del hogar como proxy de la capacidad de pago para asegurar un objetivo de morosidad nula. Aplica una función asimétrica basada en el coste medio de una plaza (~2,100€/mes):

```
Score(Renta) = {
  0,                                    si Renta < 32,000
  (Renta - 32,000) / 40,000 × 0.7,     si 32,000 ≤ Renta ≤ 72,000
  1 + 0.5 × (Renta - 72,000) / 32,000, si Renta > 72,000
}
```

### Capa 4: Saturación Territorial (Peso: 15%)

Factor de corrección que mide la relación entre la oferta de plazas existentes y la población de 80 o más años por provincia. A menor saturación, mayor es la puntuación final.

## 4. Fórmula SEED Final

La puntuación global de cada ubicación se obtiene mediante la suma ponderada de las dimensiones principales:

```
SEED_score = 0.45 · Score_Demanda + 0.40 · Score_Renta + 0.15 · Score_Saturación
```

## 5. Restricción Espacial: Clustering Adaptativo

Para evitar la canibalización entre centros, SEED implementa una restricción de distancia mínima que varía según la densidad del entorno:

- **Zonas de Alta Densidad** (>5,000 hab/km²): d_min = 1.5 km
- **Zonas de Densidad Media** (1,000-5,000 hab/km²): d_min = 2.5 km
- **Zonas Rurales** (<1,000 hab/km²): d_min = 5.0 km

Se utiliza la **fórmula de Haversine** para calcular distancias geodésicas precisas sobre la superficie terrestre.

## 6. Validación y Resultados Estratégicos

- **Fiabilidad**: El modelo presenta una correlación de **r = 0.882** respecto a la distribución real del sector.
- **Ubicación #1**: La sección censal 1503003001 en A Coruña obtuvo el score más alto (0.838) con una renta de 74,388€.
- **Mercados Clave**: Galicia, Comunidad Valenciana, Andalucía, Cataluña y Madrid concentran el mayor potencial estratégico.
- **Perfil de Éxito**: El Top 50 de ubicaciones promedia una renta de 72,769.89 € y un F-of-M de 0.1194, alineándose con el "punto ideal" del modelo.

## 7. Comparativa de Enfoques

| Característica | K-means Tradicional | SEED Algorithm |
|---------------|---------------------|----------------|
| **Objetivo** | Agrupar puntos similares | Dispersar ubicaciones óptimas |
| **Distancias** | Ignora mínimos geográficos | Garantiza separación mínima adaptativa |
| **Selección** | Centroides estadísticos | Greedy iterativo por score descendente |
| **Adaptabilidad** | Estática / Uniforme | Ajusta según densidad urbana o rural |

## 8. Stack Tecnológico

- **Análisis y Datos**: pandas, numpy, openpyxl
- **Cálculo Espacial**: scipy (Haversine), scikit-learn (MinMaxScaler)
- **Visualización**: folium (Mapas de calor interactivos), matplotlib

## 9. Estructura de Datos de Entrada (VARIABLES SEED.xlsx)

El algoritmo requiere las siguientes variables por sección censal:

- `id_seccion`: Código INE de la sección
- `latitude` / `length`: Coordenadas del centroide
- `f_of_m`: Número de hogares / Idoneidad demográfica
- `density`: Densidad poblacional (hab/km²)
- `dependence`: Tasa de dependencia (%)
- `rent`: Renta media del hogar (€)
- `saturation`: Factor de saturación competitiva

---

© 2026 Proyecto Ganador del Excellence Award en Akademia Future Builders.



