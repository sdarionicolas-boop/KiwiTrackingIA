# KiwiTrackingIA
### Análisis de Variabilidad Productiva para Kiwi · Lote "El Abrojito" · 2019–2025

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue)](https://python.org)
[![GEE](https://img.shields.io/badge/Google%20Earth%20Engine-API-green)](https://earthengine.google.com)
[![Licencia](https://img.shields.io/badge/Licencia-MIT-yellow)](LICENSE)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.19352553.svg)](https://doi.org/10.5281/zenodo.19352553)
---

## Descripción

KiwiTrackingIA es un sistema de análisis de variabilidad productiva para el cultivo de kiwi (*Actinidia deliciosa*) que integra teledetección satelital, topografía real y análisis climático con técnicas de machine learning.

El proyecto construye el **KVPI (Kiwi Vegetation Productivity Index)**, un índice compuesto a nivel planta derivado de Sentinel-2, y lo integra con:

- Topografía real (DEM Copernicus)
- Corrección por sesgo de malla de protección (opcional, según el lote)
- GDD (Grados Día de Crecimiento, Tbase=10°C)
- Horas de Frío (modelo coseno, requerimiento cultivar Hayward)
- Alertas climáticas mediante Isolation Forest
- Recomendaciones de manejo por planta (percentil p75 de estabilidad)
- Dashboard interactivo (Folium)

El análisis se desarrolló sobre el lote **"El Abrojito"** (Miramar, Buenos Aires, Argentina) para el período **junio 2019 – junio 2025** (6 campañas agrícolas).

---

## Estructura del repositorio

```
KiwiTrackingIA/
│
├── notebooks/
│   ├── 01_KVPI_Calculo.ipynb             # Pipeline GEE: índices, cálculo KVPI, análisis multianual
│   ├── 02_KVPI_vs_DEM.ipynb              # Validación topográfica: DEM Copernicus + pendiente
│   └── 03_IVP_Kiwi_Producto_Final.ipynb  # Integración total: KVPI + DEM + GDD + IA + mapa
│
├── datos/                                # Datos de entrada (ver datos/README.md)
│   ├── README.md
│   ├── KVPI_serie_2019_2025.csv          # Serie histórica KVPI por planta (generada por NB01)
│   ├── muestreo_sistematico.csv          # Coordenadas y IDs de plantas
│   ├── datos_climaticos_diarios_2019_2026.csv
│   ├── Dataset_Kiwi_DL_Final_2019_2025.csv
│   ├── DEM_Miramar_UTM.tif               # (Git LFS si > 100 MB)
│   ├── Slope_Recalc_UTM.tif             # (Git LFS si > 100 MB)
│   └── malla_zona.geojson                # Opcional: zona bajo malla de protección
│
├── resultados/                           # Outputs generados por NB03
│   ├── IVP_Kiwi_Producto_Final.html      # Mapa interactivo unificado
│   ├── datos_plantas_completos.csv       # Incluye KVPI original y corregido
│   ├── gdd_por_campana.csv
│   ├── horas_frio_por_campana.csv
│   ├── meses_anomalos.csv
│   ├── gdd_por_campana.png
│   ├── horas_frio_por_campana.png
│   ├── kvpi_vs_topografia.png
│   ├── correccion_mallas.png             # Solo si el lote tiene malla
│   ├── anomalias_climaticas.png
│   └── metadata_ejecucion.txt
│
├── mapas_interactivos/                   # Mapas HTML por campaña (generados por NB01)
│   ├── Mapa_KVPI_2019_2020.html
│   └── ...
│
├── docs/
│   ├── metodologia.md
│   ├── justificacion_umbrales.md
│   ├── interpretacion_resultados.md
│   └── guia_uso.md
│
├── README.md
├── LICENSE
├── requirements.txt
└── .gitignore
```

---

## Metodología

**KVPI** se calcula como la media de cuatro índices espectrales Sentinel-2 (NDVI, NDRE, NDMI, EVI), todos normalizados en [-1, 1], sobre imágenes con cobertura de nubes < 20% por campaña. El índice representativo de cada campaña es la mediana temporal del conjunto. Los indicadores por planta (KVPI_mean, CV, estabilidad) se calculan sobre la serie de 6 campañas (2019–2025).

Si el lote tiene sectores bajo malla de protección, el KVPI se **corrige por el sesgo óptico que introduce el sombreo de la malla** (ver `docs/metodologia.md`, sección 5.1) antes de continuar el análisis — `KVPI_corregido` es la variable usada de ahí en más.

La clasificación de ambientes usa terciles de KVPI_corregido (p33/p66). Las recomendaciones de manejo combinan potencial estructural con estabilidad interanual usando el percentil 75 como umbral conservador, y se ajustan por déficit de horas de frío en la última campaña (ver `docs/justificacion_umbrales.md`).

---

## Notebooks: flujo de ejecución

Los notebooks están diseñados para ejecutarse en orden, ya que el output de cada uno es input del siguiente:

```
01_KVPI_Calculo.ipynb
  └─► genera: KVPI_serie_2019_2025.csv

02_KVPI_vs_DEM.ipynb
  └─► requiere: KVPI_serie_2019_2025.csv + DEM/Slope .tif

03_IVP_Kiwi_Producto_Final.ipynb
  └─► requiere: todos los datos anteriores
  └─► genera:  todos los resultados finales
```

---

## Requisitos

- Python 3.8+
- Google Earth Engine (cuenta autenticada, proyecto GEE propio)
- Google Colab (recomendado) o entorno local con GPU/CPU

```bash
pip install -r requirements.txt
```

Para autenticar GEE:
```python
import ee
ee.Authenticate()
ee.Initialize(project='TU_PROYECTO_GEE')
```

---

## Instalación y uso rápido

```bash
git clone https://github.com/sdarionicolas-boop/KiwiTrackingIA.git
cd KiwiTrackingIA
pip install -r requirements.txt
```

Subir la carpeta `datos/` a Google Drive en `MyDrive/KiwiTrackingIA/` y ejecutar los notebooks en orden desde Google Colab.

---

## Hallazgos principales

- **Las mallas de protección introducen un sesgo espectral sistemático:** en el lote de referencia, las plantas bajo malla monofilamento mostraron un KVPI 18.5% menor que las plantas a cielo abierto (p < 0.001), no explicado por diferencias de elevación ni pendiente. El sesgo es consistente con el sombreo del 15-17% que producen estas mallas y se corrige con un factor multiplicativo (FC=1.227) antes de cualquier análisis posterior — un paso necesario en cualquier lote con infraestructura de protección parcial.
- El KVPI corregido mostró estabilidad espacial alta entre campañas: las zonas de mayor potencial se mantuvieron estructuralmente consistentes durante los 6 años analizados.
- La topografía (elevación y pendiente) no resultó determinante de la variabilidad espacial del KVPI en el lote de referencia, a diferencia de lo reportado en otros estudios de variabilidad intralote.
- El análisis de GDD evidenció variabilidad interanual en el acúmulo térmico, con impacto detectable en el KVPI medio por campaña.
- Las horas de frío (mayo-agosto) resultaron deficientes respecto del requerimiento del cultivar Hayward en 2 de 8 temporadas analizadas, coincidiendo con los meses climáticamente anómalos detectados por Isolation Forest — evidencia de que la vulnerabilidad térmica del sitio no es un evento aislado.
- Isolation Forest identificó meses con comportamiento anómalo en la combinación NDVI–GDD–precipitación–horas de frío, coincidentes con eventos climáticos extremos del período.

---

## Archivos grandes (Git LFS)

Los rasters DEM y Slope superan el límite de GitHub (100 MB). Si es necesario, usar Git LFS:

```bash
git lfs install
git lfs track "*.tif"
git add .gitattributes
```

---

## Licencia

MIT License. Ver [LICENSE](LICENSE).

---

## Autor

**Darío Nicolás Sánchez Leguizamón**  
Técnico en Producción Vegetal Intensiva (UNAJ) · Licenciatura en Ciencias Agrarias (en curso)  
Becario BIEI 2025 – Banco de Germoplasma de Especies Nativas (BGEN), UNAJ  
GEF/PNUD ARG/19/G24 · Red ARGENA

📧 sdarionicolas@gmail.com  
🔗 [LinkedIn](https://www.linkedin.com/in/darionicolas)  
🐙 [GitHub](https://github.com/sdarionicolas-boop)



**🌱 Filosofía de uso**

"Este proyecto se distribuye como código abierto bajo licencia MIT. La intención es que cada usuario lo adapte a sus propios cultivos, regiones y necesidades. No se incluyen archivos de datos pesados; todos los insumos se obtienen de fuentes abiertas (Sentinel-2, INTA, Copernicus) o se generan al ejecutar los notebooks. ¡Sentite libre de explorar, modificar y compartir!"

## 📚 Cómo citar este trabajo

Si utilizás KiwiTrackingIA en tu investigación, por favor citá la tesis asociada:

> Sánchez Leguizamón, D. N. (2025). *Análisis de variabilidad productiva para kiwi integrando teledetección, topografía y machine learning* [Tesis de licenciatura]. Universidad Nacional Arturo Jauretche. https://github.com/sdarionicolas-boop/KiwiTrackingIA

**Versión citada**: v1.0-tesis  