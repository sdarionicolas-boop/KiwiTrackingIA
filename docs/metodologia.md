# Metodología – KiwiTrackingIA

## 1. Fuente de datos satelitales

**Colección:** Sentinel-2 Surface Reflectance Harmonized (`COPERNICUS/S2_SR_HARMONIZED`)  
**Plataforma:** Google Earth Engine (GEE)  
**Resolución espacial:** 10 m  
**Período:** junio 2019 – junio 2025 (6 campañas agrícolas)  
**Filtro de nubes:** cobertura < 20% (atributo `CLOUDY_PIXEL_PERCENTAGE`) + máscara QA60 (bits 10 y 11)

## 2. Índices espectrales

| Índice | Fórmula | Interpretación |
|---|---|---|
| NDVI | (B8 - B4) / (B8 + B4) | Vigor general de la vegetación |
| NDRE | (B8 - B5) / (B8 + B5) | Actividad fotosintética (clorofila) |
| NDMI | (B8 - B11) / (B8 + B11) | Condición hídrica |
| EVI | 2.5 × (NIR - RED) / (NIR + 6·RED - 7.5·BLUE + 1) | Vigor con corrección atmosférica |

## 3. Cálculo del KVPI

```
KVPI = mean(NDVI, NDRE, EVI, NDMI)
```

- Todos los índices están normalizados en [-1, 1]
- La imagen representativa de cada campaña es la **mediana temporal** del conjunto de imágenes sin nubes en el período junio–junio
- El muestreo por planta usa `sampleRegions` de GEE a escala de 10 m

## 4. Indicadores por planta (serie 2019–2025)

| Indicador | Descripción |
|---|---|
| `KVPI_mean` | Media de KVPI entre las 6 campañas (potencial estructural), sin corregir |
| `KVPI_corregido` | `KVPI_mean` ajustado por el factor de corrección de malla (sección 5.1). Si el lote no tiene malla, es igual a `KVPI_mean` |
| `KVPI_std` | Desvío estándar interanual |
| `KVPI_cv` | Coeficiente de variación (std/mean) |
| `estabilidad` | Índice KVPI_corregido/std (mayor = más estable) |
| `bajo_malla` | 1 si la planta está dentro del polígono de malla de protección, 0 si no |

## 5. Topografía

**DEM:** Copernicus DEM (30 m) procesado en GEE y reproyectado a UTM (EPSG:32721)  
**Pendiente:** calculada desde el DEM con `ee.Terrain.slope()`  
**Extracción:** intersección punto-a-píxel con `rasterio.sample`

## 5.1 Corrección por sesgo de malla de protección *(opcional, según el lote)*

Las mallas monofilamento antigranizo/antiheladas atenúan la radiación fotosintéticamente activa entre un 15% y un 17% (David *et al.*, 2020), lo que introduce un **sesgo óptico sistemático** en los índices espectrales de las plantas cubiertas — no un problema de calidad de datos, sino un efecto físico real de la infraestructura.

**Detección:** se delimita la zona bajo malla en Google Earth (polígono `malla_zona.geojson`) y se marca `bajo_malla=1` para todo punto contenido en él.

**Cuantificación:** se compara `KVPI_mean` entre plantas bajo malla y a cielo abierto mediante una prueba t de Student, y se comparan elevación y pendiente entre ambos grupos para descartar la topografía como factor explicativo alternativo.

**Corrección:** si la diferencia es significativa (p < 0.05), se calcula un factor de corrección y se aplica solo a las plantas bajo malla:

```
FC = KVPI_mean(fuera de malla) / KVPI_mean(bajo malla)
KVPI_corregido = KVPI_mean × FC   (solo para bajo_malla == 1)
```

En el caso de referencia (lote "El Abrojito", 53 de 159 plantas bajo malla instalada en dic-2018): KVPI fuera de malla = 0.541, KVPI bajo malla = 0.441 (p < 0.001, diferencia no explicada por elevación ni pendiente) → **FC = 1.227**, que reduce a cero la diferencia entre poblaciones.

A partir de este punto, todos los análisis subsiguientes (clasificación de ambientes, estabilidad, recomendaciones, mapa) usan `KVPI_corregido` como variable principal. Si el lote no tiene malla, o el archivo `malla_zona.geojson` no existe, este paso se omite automáticamente y `KVPI_corregido = KVPI_mean`.

## 6. GDD (Grados Día de Crecimiento)

```
GDD_diario = max(Tmedia - 10, 0)     donde Tmedia = (Tmax + Tmin) / 2
```

- **Tbase:** 10°C (valor estándar para *Actinidia deliciosa*)
- **Temporada:** se inicia en julio de cada año
- **Fuente de temperatura:** datos diarios de estación meteorológica INTA o equivalente

## 6.1 Horas de Frío (modelo coseno)

El cultivar Hayward requiere una acumulación mínima de horas de frío (T ≤ 7°C) durante el reposo invernal para una brotación y floración sincronizadas (David *et al.*, 2020). Como los registros climáticos disponibles son diarios (Tmax/Tmin) y no horarios, las horas bajo el umbral se estiman con un **modelo de distribución coseno** de la temperatura a lo largo del día:

```
amp    = (Tmax - Tmin) / 2
Tmedia = (Tmax + Tmin) / 2
θ      = arccos( clip((umbral - Tmedia) / amp, -1, 1) )
horas_bajo_umbral = (24 / π) × (π - θ)
```

- **Umbral:** 7°C
- **Ventana:** mayo–agosto (período de dormancia)
- **Agregación:** suma por temporada agrícola
- **Clasificación (requerimiento Hayward):**

| Horas de frío | Estado |
|---|---|
| ≥ 950 h | Óptimo |
| 750–950 h | Adecuado |
| < 750 h | Deficiente |

## 7. Detección de anomalías climáticas

**Algoritmo:** Isolation Forest (`contamination=0.1`, `random_state=42`)  
**Features:** NDVI mensual, temperatura media, precipitación mensual, GDD mensual, horas de frío mensuales (0 fuera de mayo–agosto)  
**Preprocesamiento:** StandardScaler (media=0, std=1)  
**Interpretación:** puntaje de anomalía más negativo = mayor atipicidad

## 8. Clasificación de ambientes y recomendaciones

### Ambientes productivos
Clasificación en terciles del `KVPI_corregido` multianual:
- **Alto Potencial:** KVPI_corregido ≥ p66
- **Transicional:** p33 ≤ KVPI_corregido < p66
- **Limitante:** KVPI_corregido < p33

### Recomendaciones de manejo
Combinan potencial (`KVPI_corregido`) con estabilidad interanual:

| Condición | Recomendación |
|---|---|
| KVPI_corregido ≥ p66 **y** estabilidad ≥ p75 | Mantener |
| KVPI_corregido ≤ p33 **y** estabilidad ≥ p75 | Intervenir |
| Resto | Observar |

**Ajuste por déficit de frío:** si la última campaña disponible se clasificó como "Deficiente" en horas de frío, las plantas con recomendación "Mantener" se reclasifican a "Observar" — un potencial estructural alto no se expresa si el cultivo no acumuló el frío invernal necesario esa temporada.

Ver `justificacion_umbrales.md` para la fundamentación del uso de p75.

## Referencias

David, M.A.; Yommi, A. & Sánchez, E. (2020). *Elección del terreno y plantación del cultivo de kiwi*. INTA Ediciones. ISBN 978-987-8333-45-8.
