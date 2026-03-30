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
| `KVPI_mean` | Media de KVPI entre las 6 campañas (potencial estructural) |
| `KVPI_std` | Desvío estándar interanual |
| `KVPI_cv` | Coeficiente de variación (std/mean) |
| `KVPI_estabilidad` | Índice mean/std (mayor = más estable) |
| `años_alto/medio/bajo` | Número de campañas en cada clase de tercil |

## 5. Topografía

**DEM:** Copernicus DEM (30 m) procesado en GEE y reproyectado a UTM (EPSG:32721)  
**Pendiente:** calculada desde el DEM con `ee.Terrain.slope()`  
**Extracción:** intersección punto-a-píxel con `rasterio.sample`

## 6. GDD (Grados Día de Crecimiento)

```
GDD_diario = max(Tmedia - 10, 0)     donde Tmedia = (Tmax + Tmin) / 2
```

- **Tbase:** 10°C (valor estándar para *Actinidia deliciosa*)
- **Temporada:** se inicia en julio de cada año
- **Fuente de temperatura:** datos diarios de estación meteorológica INTA o equivalente

## 7. Detección de anomalías climáticas

**Algoritmo:** Isolation Forest (`contamination=0.1`, `random_state=42`)  
**Features:** NDVI mensual, temperatura media, precipitación mensual, GDD mensual  
**Preprocesamiento:** StandardScaler (media=0, std=1)  
**Interpretación:** puntaje de anomalía más negativo = mayor atipicidad

## 8. Clasificación de ambientes y recomendaciones

### Ambientes productivos
Clasificación en terciles del KVPI_mean multianual:
- **Alto Potencial:** KVPI_mean ≥ p66
- **Transicional:** p33 ≤ KVPI_mean < p66
- **Limitante:** KVPI_mean < p33

### Recomendaciones de manejo
Combinan potencial (KVPI_mean) con estabilidad interanual:

| Condición | Recomendación |
|---|---|
| KVPI_mean ≥ p66 **y** estabilidad ≥ p75 | Mantener |
| KVPI_mean ≤ p33 **y** estabilidad ≥ p75 | Intervenir |
| Resto | Observar |

Ver `justificacion_umbrales.md` para la fundamentación del uso de p75.
