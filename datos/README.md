# Datos de entrada – KiwiTrackingIA

Esta carpeta contiene todos los archivos de datos necesarios para ejecutar los notebooks del proyecto.

## Archivos requeridos

| Archivo | Generado por | Descripción |
|---|---|---|
| `muestreo_sistematico.csv` | Manual / campo | Coordenadas (X=lon, Y=lat) e ID de cada planta muestreada |
| `KVPI_serie_2019_2025.csv` | `01_KVPI_Calculo.ipynb` (GEE export) | Serie histórica de KVPI por planta y campaña |
| `datos_climaticos_diarios_2019_2026.csv` | INTA / fuente climática | Tmax, Tmin, Precip diarios para el período |
| `Dataset_Kiwi_DL_Final_2019_2025.csv` | Procesamiento previo | NDVI mensual del lote para Isolation Forest |
| `DEM_Miramar_UTM.tif` | Copernicus DEM (GEE) | Modelo digital de elevación, reproyectado a UTM |
| `Slope_Recalc_UTM.tif` | Derivado del DEM | Raster de pendiente (%) en UTM |

## Estructura de columnas clave

### `muestreo_sistematico.csv`
```
id, X, Y, [row_index, col_index]
```
- `id`: identificador único de planta
- `X`: longitud (WGS84)
- `Y`: latitud (WGS84)

### `KVPI_serie_2019_2025.csv`
```
id, campaña, KVPI, .geo, [row, col]
```
- `.geo`: geometría GeoJSON exportada por GEE (columna estándar de export)
- `campaña`: formato `YYYY_YYYY+1` (ej: `2019_2020`)

### `datos_climaticos_diarios_2019_2026.csv`
```
date, Tmax, Tmin, Precip
```
- `date`: formato `YYYY-MM-DD`
- Temperaturas en °C, precipitación en mm

## Nota sobre archivos .tif

Los rasters DEM y Slope pueden superar los 100 MB (límite de GitHub).  
Si es el caso, se recomienda usar **Git LFS**:

```bash
git lfs install
git lfs track "*.tif"
git add .gitattributes
```

Alternativamente, los archivos `.tif` pueden descargarse directamente desde Google Drive / Copernicus DEM y ubicarse aquí antes de ejecutar los notebooks.
