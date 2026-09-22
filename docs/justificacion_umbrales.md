# Justificación de Umbrales – KiwiTrackingIA

## Umbral de estabilidad: percentil 75 (p75)

El umbral de estabilidad productiva utilizado en las recomendaciones de manejo es el **percentil 75** del índice `estabilidad` (= KVPI_corregido / KVPI_std) calculado sobre el conjunto de plantas del lote. Se usa el KVPI ya corregido por sesgo de malla (si corresponde; ver `metodologia.md`, sección 5.1), de modo que la estabilidad no quede distorsionada por el sombreo de la infraestructura de protección.

### Fundamentación

**1. Enfoque conservador**  
El p75 implica que sólo el 25% superior de las plantas del lote es considerado de "alta estabilidad". Esto evita sobreestimar la cantidad de plantas candidatas a intervención o mantenimiento diferenciado.

**2. Minimización de falsos positivos**  
En agricultura de precisión, una recomendación de "Intervenir" incorrecta tiene mayor costo (económico y agronómico) que omitir una planta. El p75 reduce la tasa de falsos positivos priorizando precision sobre recall.

**3. Focalización de recursos**  
Al restringir las categorías "Mantener" e "Intervenir" a plantas con alta estabilidad interanual, se asegura que las recomendaciones apliquen a plantas con un comportamiento estructural consistente y no a respuestas coyunturales de una sola campaña.

**4. Consistencia con literatura**  
El uso de percentiles altos (p75–p80) como umbrales de corte en índices de estabilidad es una práctica documentada en estudios de variabilidad intraespecífica en frutales de hoja caduca (ver literatura de estabilidad genotipo×ambiente).

**5. Validez empírica del resultado**  
En el lote de referencia (El Abrojito, sobre KVPI corregido), la distribución resultante fue ~88% Observar y ~12% Intervenir — sin plantas en "Mantener" esa campaña, por el ajuste adicional según el estado de horas de frío (ver más abajo). Es coherente con la estructura esperada en un lote productivo típico de kiwi, donde la mayoría de las plantas se encuentra en condición intermedia y sólo una fracción pequeña presenta comportamiento productivo extremo y estable.

**6. Ajuste por déficit de frío**  
Las recomendaciones de manejo se calculan sobre el potencial y la estabilidad estructural del KVPI, pero un potencial alto no se expresa si el cultivo no acumuló las horas de frío necesarias esa temporada. Por eso, si la última campaña disponible se clasifica como "Deficiente" en horas de frío (< 750 h; ver `metodologia.md`, sección 6.1), las plantas con recomendación "Mantener" se reclasifican a "Observar": el ajuste evita recomendar continuidad de manejo sobre una base climática que no se cumplió ese año.

## Clasificación de ambientes: terciles de KVPI (p33/p66)

La clasificación en **tres categorías iguales** (p33/p66) fue elegida por:

- **Parsimonia:** tres clases son interpretables agronómicamente sin requerir mayor resolución.
- **Balance:** los terciles garantizan que cada clase tenga aproximadamente la misma cantidad de plantas, evitando clases muy pequeñas.
- **Convención en PA:** la zonificación en tres ambientes (alto/medio/bajo potencial) es el estándar en agricultura de precisión para cultivos perennes.

## Nota sobre adaptación a otros lotes

Estos umbrales son **relativos al lote analizado** (El Abrojito). Para aplicar el sistema a otro lote o cultivo, los umbrales deben recalibrarse sobre la distribución de ese nuevo conjunto de datos. No deben usarse como valores absolutos entre lotes distintos.
