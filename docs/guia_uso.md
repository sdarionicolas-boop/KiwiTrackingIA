# Guía de Uso – KiwiTrackingIA

## Para técnicos y productores: ¿cómo leer los resultados?

### El mapa interactivo (`IVP_Kiwi_Producto_Final.html`)

Abrir el archivo en cualquier navegador (Chrome, Firefox, Edge). No requiere conexión a internet.

**Cada punto = una planta de kiwi.**

| Color | Ambiente | Significado |
|---|---|---|
| 🟢 Verde | Alto Potencial | Plantas con mayor KVPI promedio 2019–2025 y alta estabilidad |
| 🟠 Naranja | Transicional | Plantas en rango intermedio, respuesta variable según el año |
| 🔴 Rojo | Limitante | Plantas con menor potencial relativo; pueden tener limitantes de suelo, drenaje o posición topográfica |

**Al pasar el mouse sobre una planta** aparece su ID, ambiente y recomendación.  
**Al hacer click** se despliega el detalle completo: KVPI_mean, estabilidad, elevación, pendiente, número de campañas, recomendación y GDD promedio.

### Recomendaciones de manejo

| Recomendación | Acción sugerida |
|---|---|
| **Mantener** | Plantas con alto potencial estructural y alta estabilidad. Continuar con el manejo actual. |
| **Observar** | Plantas con comportamiento intermedio o variable. Monitorear de cerca; no requieren intervención inmediata. |
| **Intervenir** | Plantas con bajo potencial pero alta estabilidad (bajo consistente). Evaluar causas locales: suelo, porta-injerto, posición en la hilera, competencia. |

### El heatmap de KVPI

La capa de calor (activable desde el control de capas) muestra la densidad espacial del potencial KVPI. Las zonas más oscuras/calientes corresponden a sectores de mayor potencial relativo.

---

## Para desarrolladores: flujo de ejecución

Ver `metodologia.md` para el detalle técnico de cada etapa.

### Paso a paso

1. **Preparar datos de entrada** según la estructura descripta en `datos/README.md`
2. **Ejecutar `01_KVPI_Calculo.ipynb`** en Google Colab con GEE autenticado → genera `KVPI_serie_2019_2025.csv`
3. **Ejecutar `02_KVPI_vs_DEM.ipynb`** para validación topográfica → genera mapas de ambientes
4. **Ejecutar `03_IVP_Kiwi_Producto_Final.ipynb`** → genera todos los resultados finales en `resultados/`

### Adaptar a otro lote

Los parámetros que deben ajustarse para un nuevo lote son:

- `muestreo_sistematico.csv`: reemplazar con los puntos del nuevo lote
- `campanias` en NB01: ajustar fechas si el cultivo o la región tienen otro calendario
- `DEM` y `Slope` correspondientes al nuevo sitio
- Datos climáticos del sitio de interés

Los umbrales (p33/p66/p75) se recalculan automáticamente sobre la nueva distribución.
