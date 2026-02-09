# SEED Algorithm: Socio-Economic and Environmental Distribution

## Descripción General

**SEED** (*Socio-Economic and Environmental Distribution*) es un algoritmo de optimización espacial diseñado para seleccionar y distribuir 1000 ubicaciones óptimas para residencias de mayores en España.

El algoritmo combina análisis territorial, demanda residencial, viabilidad económica y restricciones espaciales para garantizar una distribución equilibrada que maximice la cobertura territorial y la rentabilidad del proyecto.

---

## Objetivo

Seleccionar 1000 ubicaciones óptimas de entre ~36,000 secciones censales en España, optimizando:

- **Cobertura territorial**: Evitar saturación y canibalización entre residencias
- **Demanda social**: Priorizar zonas con mayor población dependiente
- **Viabilidad económica**: Seleccionar áreas con poder adquisitivo adecuado
- **Distribución adaptativa**: Ajustar densidad de residencias según características urbanas/rurales

---

## Arquitectura del Algoritmo

El algoritmo SEED (residence-mvp/notebooks/03_seed_algorithm.ipynb) está compuesto por **4 capas** que evalúan diferentes dimensiones:

### **Capa 1: Base Territorial**
- **Espacio de decisión**: 36,000 secciones censales
- **Datos geográficos**: Coordenadas (latitud, longitud) de cada sección
- **Granularidad**: Nivel de sección censal (máxima precisión administrativa)

### **Capa 2: Demanda Residencial** (Peso: 0.45)
Evalúa el potencial de demanda en cada ubicación mediante:

- **Número de hogares** (65%): Mayor base poblacional = mayor demanda potencial
- **Tasa de dependencia** (10%): % población >65 años y <16 años
- **Densidad poblacional** (25%): Habitantes por km²

**Fórmula de la capa:**
```
Demanda = 0.65 × hogares_norm + 0.10 × dependencia_norm + 0.25 × densidad_norm
```

### **Capa 3: Viabilidad Económica** (Peso: 0.40)
- **Renta media del hogar**: Indicador de capacidad de pago
- **Normalización**: Escala 0-1 usando MinMaxScaler
- **Impacto**: Segunda variable más importante del modelo

### **Capa 4: Saturación Territorial** (Peso: 0.15)
- **Factor de corrección**: Penaliza zonas con alta competencia existente
- **Inversión**: Menor saturación = mayor puntuación (1 - saturación_norm)
- **Función**: Evitar concentración excesiva en áreas ya saturadas

---

## Fórmula SEED

La puntuación final de cada sección censal se calcula mediante:

```
SEED_score = 0.45 × (0.65 × hogares + 0.10 × dependencia + 0.25 × densidad)
           + 0.40 × renta
           + 0.15 × (1 - saturación)
```

**Pesos de las capas:**
- 45% → Demanda residencial
- 40% → Viabilidad económica  
- 15% → Corrección por saturación

Todas las variables están normalizadas en el rango [0, 1] usando `MinMaxScaler` de scikit-learn.

---

## Restricción Espacial: Clustering Adaptativo

### Concepto

En lugar de usar clustering tradicional (K-means, DBSCAN), SEED implementa una **restricción de distancia mínima adaptativa** que evita la saturación territorial mientras selecciona iterativamente las mejores ubicaciones.

### Distancias Mínimas por Tipo de Zona

```python
DISTANCIA_MIN_CIUDAD_GRANDE = 1.5 km   # Densidad > 5000 hab/km²
DISTANCIA_MIN_CIUDAD_MEDIA   = 2.5 km   # Densidad > 1000 hab/km²
DISTANCIA_MIN_RURAL          = 5.0 km   # Densidad < 1000 hab/km²
```

**Rationale:**
- **Ciudades grandes**: Mayor demanda permite residencias más cercanas
- **Ciudades medias**: Balance entre demanda y territorialidad
- **Zonas rurales**: Mayor dispersión para garantizar cobertura

### Algoritmo de Selección Iterativa

```
ENTRADA: DataFrame con 36,000 secciones censales y SEED_score calculado
SALIDA: 1000 ubicaciones seleccionadas

1. Ordenar todas las secciones por SEED_score (descendente)
2. MIENTRAS longitud(seleccionadas) < 1000:
   
   a. Seleccionar la sección con mayor SEED_score disponible
   
   b. Añadir a lista de seleccionadas
   
   c. Determinar distancia_min según densidad de la sección:
      - Si densidad > 5000 → distancia_min = 1.5 km
      - Si densidad > 1000 → distancia_min = 2.5 km
      - Si no → distancia_min = 5.0 km
   
   d. Calcular distancias Haversine desde nueva ubicación a todas las restantes
   
   e. Eliminar del pool todas las secciones dentro de distancia_min
   
   f. Continuar con secciones restantes

3. RETORNAR lista de 1000 ubicaciones seleccionadas
```

### Cálculo de Distancia Haversine

Se utiliza la fórmula de Haversine para calcular distancias geodésicas precisas:

```python
from scipy.spatial.distance import cdist

# Convertir coordenadas a radianes
coords_rad = np.radians(coordenadas)

# Calcular matriz de distancias
distancias_km = cdist(punto_seleccionado, coords_restantes, 
                      metric='haversine') * 6371  # Radio terrestre
```

---

## Stack Tecnológico

### Dependencias principales

```python
pandas              # Manipulación de datos
numpy               # Cálculos numéricos
scikit-learn        # Normalización (MinMaxScaler)
scipy               # Cálculo de distancias (Haversine)
folium              # Visualización de mapas interactivos
matplotlib          # Gráficos estáticos
openpyxl            # Lectura de archivos Excel
```

### Instalación

```bash
pip install pandas numpy folium scipy scikit-learn openpyxl matplotlib
```

---

## Proceso de Normalización

### MinMaxScaler

Todas las variables se normalizan usando `sklearn.preprocessing.MinMaxScaler`:

```
X_normalizado = (X - X_min) / (X_max - X_min)
```

Resultado: Todas las variables en rango [0, 1]

### Variables invertidas

Algunas variables tienen interpretación inversa (menor = mejor):

```python
# Inversión para factor de masculinidad (equilibrio = 1)
f_of_m_norm = 1 - MinMaxScaler(f_of_m)

# Inversión para saturación (menos competencia = mejor)
saturation_norm_inv = 1 - MinMaxScaler(saturation)
```

---

## Outputs Generados

### 1. Rankings CSV
- `SEED_ranking.csv`: Ranking completo de las 1000 ubicaciones seleccionadas
- Columnas: `seccion_censal`, `latitud`, `longitud`, `SEED_score`, `ranking`, `provincia`

### 2. Mapas de Calor (HTML interactivos)
- **Gradiente de demanda residencial**: Top 5% secciones por demanda
- **Gradiente de viabilidad económica**: Top 5% secciones por renta
- **Gradiente SEED completo**: Top 5% secciones por score final

### 3. Mapa de Ubicaciones Seleccionadas
- Visualización de las 1000 residencias seleccionadas
- Markers interactivos con información de cada ubicación
- Color coding por provincia

---

## Ventajas del Enfoque SEED

### Frente a clustering tradicional (K-means)

| Característica | K-means | SEED |
|----------------|---------|------|
| **Objetivo** | Agrupar puntos similares | Dispersar ubicaciones óptimas |
| **Distancias** | No respeta mínimos geográficos | Garantiza distancia mínima |
| **Selección** | Centroides de clusters | Greedy por score descendente |
| **Adaptabilidad** | Uniforme | Ajusta según densidad urbana/rural |

### Optimizaciones específicas

1. **Score multidimensional**: Combina 3 dimensiones clave (demanda, economía, competencia)
2. **Greedy optimizado**: Siempre selecciona la mejor ubicación disponible
3. **Prevención de canibalización**: Distancias mínimas evitan solapamiento de áreas de influencia
4. **Escalabilidad**: Maneja 36,000 puntos de decisión eficientemente
5. **Cobertura territorial**: Garantiza representación en zonas rurales y urbanas

---

## Estructura de Datos de Entrada

### Archivo: `VARIABLES SEED.xlsx`

**Columnas requeridas:**

```
- id_seccion      : Código INE de sección censal (ej: 2801101001)
- latitude        : Latitud del centroide de la sección
- length          : Longitud del centroide de la sección
- f_of_m          : Número de hogares
- density         : Densidad poblacional (hab/km²)
- dependence      : Tasa de dependencia (%)
- rent            : Renta media del hogar (€)
- saturation      : Factor de saturación competitiva
```

**Preprocesamiento:**
- Filtrado de valores nulos en coordenadas
- Extracción de código de provincia (primeros 2 dígitos)
- Mapeo a nombres de provincias

---

## Ejecución del Algoritmo

### 1. Configuración

```python
ARCHIVO_DATOS = Path("data/VARIABLES SEED.xlsx")
NUM_RESIDENCIAS = 1000

DISTANCIA_MIN_CIUDAD_GRANDE = 1.5  # km
DISTANCIA_MIN_CIUDAD_MEDIA = 2.5   # km
DISTANCIA_MIN_RURAL = 5.0          # km
```

### 2. Carga y preparación de datos

```python
df = pd.read_excel(ARCHIVO_DATOS)
df = df.rename(columns={
    'id_seccion': 'seccion_censal',
    'length': 'longitud',
    'latitude': 'latitud'
})
```

### 3. Cálculo de capas y SEED score

```python
# Normalización
scaler = MinMaxScaler()
df['hogares_norm'] = scaler.fit_transform(df[['f_of_m']])
df['densidad_norm'] = scaler.fit_transform(df[['density']])
df['dependencia_norm'] = scaler.fit_transform(df[['dependence']])
df['renta_norm'] = scaler.fit_transform(df[['rent']])
df['saturacion_norm'] = 1 - scaler.fit_transform(df[['saturation']])

# Capa 2: Demanda
df['demanda_score'] = (0.65 * df['hogares_norm'] + 
                       0.10 * df['dependencia_norm'] + 
                       0.25 * df['densidad_norm'])

# Score final SEED
df['SEED_score'] = (0.45 * df['demanda_score'] + 
                    0.40 * df['renta_norm'] + 
                    0.15 * df['saturacion_norm'])
```

### 4. Selección con restricción espacial

```python
df_sorted = df.sort_values('SEED_score', ascending=False)
seleccionadas = []

while len(seleccionadas) < NUM_RESIDENCIAS:
    mejor = df_sorted.iloc[0]
    seleccionadas.append(mejor)
    
    # Determinar distancia mínima
    dist_min = calcular_distancia_adaptativa(mejor['density'])
    
    # Calcular distancias y filtrar
    distancias = calcular_haversine(mejor, df_sorted)
    df_sorted = df_sorted[distancias > dist_min]
```

### 5. Exportación de resultados

```python
# CSV ranking
df_seleccionadas.to_csv('outputs/SEED_ranking.csv')

# Mapas interactivos
crear_mapa_gradiente(df, 'SEED_score', 
                     'SEED Algorithm - Distribución Final',
                     'outputs/SEED_mapa_completo.html')
```

## Flujo del Algoritmo (Diagrama)

```
┌─────────────────────────┐
│  ENTRADA: 36,000        │
│  secciones censales     │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│  NORMALIZACIÓN          │
│  MinMaxScaler [0,1]     │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│  CÁLCULO SEED SCORE     │
│  45% Demanda            │
│  40% Economía           │
│  15% Saturación         │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│  ORDENAR POR SCORE      │
│  (Descendente)          │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│  LOOP: seleccionadas    │
│        < 1000           │
├─────────────────────────┤
│  1. Tomar mejor score   │
│  2. Añadir a lista      │
│  3. Calcular distancias │
│  4. Filtrar cercanos    │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│  SALIDA: 1000           │
│  ubicaciones óptimas    │
└─────────────────────────┘
```

