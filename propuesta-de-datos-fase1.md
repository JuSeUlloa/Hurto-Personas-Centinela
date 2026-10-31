# Propuesta de Datos · Proyecto Centinela Metodologia Inteligencia Territorial · Fase 1

| Campo | Detalle |
| --- | --- |
| Título del proyecto | **Centinela de Hurtos** · Línea base tabular (MLP) de alerta temprana del riesgo de hurto a personas por municipio y mes |
| Integrantes | Yeison David Toro — Juan Sebastián Ulloa |
| Fecha | 2026-10-29 |
| Versión | v1 (Fase 1) |

Uso de  registro  de hurtos a personas de la Policía Nacional (por municipio y mes) para entrenar una red neuronal sencilla que avise con un mes de antelación cuáles municipios están entrando en un repunte de hurtos, distinguiendo cada territorio de su propia historia.

## 1. Problema, cliente y objetivo binario

- **Evento de riesgo y cliente:** anticipar el **repunte mensual del hurto a personas** en un municipio para las entidades encargadas de la seguridad ciudadana (gobernaciones, alcaldías, Policía) que deben decidir dónde enviar personal de vigilancia y programas de prevención con recursos escasos.
- **Impacto:** avisar a tiempo da la oportunidad de desplegar vigilancia antes de que ocurran las víctimas. Una alerta falsa mueve agentes hacia municipios donde no pasa nada y "seña" al municipio como peligroso. Por eso aquí el tamaño de cada tipo de error pesa distinto.
- **Objetivo binario:** la tasa de hurtos (hurtos por cada 100.000 habitantes, por municipio y mes) es un número continuo. Lo volvemos sí/no así:

  > **Alerta = 1** si la tasa del mes supera el **percentil 75 (P75) del propio municipio** en su historia reciente (con un ajuste suave para municipios con pocos meses de datos); **0** en el otro caso.

  **Por qué este umbral:** Con un umbral nacional único, un pueblo pequeño casi nunca superaría el promedio de Bogotá y quedaráías siempre bajo, aunque se le desborde delito. El P75 del propio municipio pregunta ¿este mes le fue inusualmente mal a este municipio?, que es la pregunta que el cliente puede responder con acciones. Con este criterio la mayoría de los meses no quedan en alerta , y ese es el caso realista. Lo que se pierde al convertir a binario: el detalle de **por cuánto** se superó el umbral — lo reportamos en el análisis de errores, pero no entra en la métrica.

## 2. Las dos modalidades del sistema completo

- **Rama de imagen:** imágenes **satelitales de luces nocturnas** (NASA Black Marble VIIRS, una foto-malla por mes a 500 metros) recortadas por el área de cada municipio. Es un instrumento físico completamente distinto al registro policial: mide luz, no denuncias. De cada imagen se extraen rasgos sencillos (cuánta área brillante, cómo se distribuye la luz, si el asentamiento es compacto o disperso).
- **Rama temporal:** la serie mensual por municipio del registro policial (modalidad **HURTO A PERSONAS**) más datos de contexto que cambian lento (población del DANE, si el municipio es PDET o ZOMAC, PIB departamental).
- **Pregunta de control — ¿La imagen contiene algo que un número no contenga?:** la respuesta es si, la forma en que se reparte la luz del asentamiento (compacto/disperso, densidad del perímetro urbano) no cabe en promedios por municipio; y como mide el territorio con otro instrumento, puede mostrar realidades que la denuncia no refleja ejemplo, municipios casi despoblados pero con iluminación industrial, o apagones.

> **Aclaración sobre el panel de visualización existente:** el proyecto ya tiene un **panel web público** donde el cliente ve los resultados en tableros y mapas: **https://intelligence-territorial-panel.vercel.app/dash**. Ese panel es la herramienta de **verificación humana de cada alerta** , pero **no** cuenta como segunda modalidad: muestra la misma serie dibijada como mapa o tablero; la imagen de este estudio es la foto satelital, que proviene de otro instrumento.

## 3. Fuentes (con enlaces y licencia)

| Modalidad | Fuente (productor primario) | Enlace o DOI | Licencia y versión | Dónde se verificó la licencia | ¿Datos de personas? |
| --- | --- | --- | --- | --- | --- |
| Temporal (principal) | **Policía Nacional — DIJIN/SIEDCO**: *Reporte Hurto por Modalidades*. El proyecto usa **solo la modalidad «HURTO A PERSONAS»** del conjunto | Portal oficial: https://portalsiedco.policia.gov.co:4443/extensions/PortalPublico/index.html#/Indicadores/Delito · copia oficial en datos abiertos: https://www.datos.gov.co/resource/d4fr-sbn2.json (ID `d4fr-sbn2`) | **CC BY-SA 4.0** (libre con atribución a la Policía) | Licencia visible en los metadatos del conjunto en datos.gov.co (API `api/views/d4fr-sbn2.json`) y en las notas del portal SIEDCO | **No** (hechos anonimizados, sin nombres ni documentos) |
| Temporal (contexto) | **DANE** — proyecciones de población por municipio (es el denominator de la tasa) | https://www.dane.gov.co/index.php/estadisticas-por-tema/demografia-y-poblacion | Dato oficial público (uso estadístico) | Sitio DANE | **No** (cifras por municipio) |
| Temporal (contexto) | **DNP** — municipios PDET y ZOMAC · **DANE** — PIB departamental | PDET: https://www.datos.gov.co/resource/idrk-ba8y · ZOMAC y PIB: portales DNP y DANE | Datos abiertos oficiales (Ley 1712 de 2014) | Ficha de cada conjunto + portal del productor | **No** (cifras por municipio) |
| Imagen | **NASA Black Marble (VIIRS)** — compuestos **mensuales** de luces nocturnas (VNP46A3) | https://blackmarble.gsfc.nasa.gov/ · descarga: https://ladsweb.modaps.eosdis.nasa.gov/ | Datos NASA de libre uso (registro gratuito en Earthdata) | Página del producto en LADS | **No** (fotos satelitales de luz) |

> Nota: el trabajo de grado del que hereda este proyecto ya trabajó con una copia (parquet) del reporte de la Policía; el cuaderno de la Fase 1 **no depende de** esa copia: descarga directo desde fuente oficial de portalsiedco.policia.gov.co con lo cual cualquier persona puede reproducirlo. Las huellas de esos archivos quedan en el anexo B.

## 4. Por qué NO es un reto resuelto

- **Filtro del estudio:** el reporte oficial incluye **todas** las modalidades de crimen (amenazas, homicidios, lesiones personales, ...). Aquí se procesa **exclusivamente «HURTO A PERSONAS»**, que es donde una persona es la víctima directa. Tras el filtro quedan **83.778 de 2026 registros**.
- La fuente es **cruda**: hay que normalizar los códigos de municipio, unificar categorías, calcular la tasa por municipio-mes, definir el umbral con su ajuste para municipios con pocos meses y armar la partición temporal. **Ningún archivo público (ni el portal SIEDCO, ni datos.gov.co, ni Kaggle) entrega esa tabla mensual ya lista con la etiqueta de alerta por P75.**
- No hay copia de competencia de Kaggle a la que atribuirlo: el productor primario es la Policía Nacional. Las búsquedas de espejos quedan documentadas con fecha en el anexo B.
- El trabajo de grado (XGBoost para estimar la tasa)  **no resuelve** esta tarea: aquí el problema es **binario** (alerta sí/no) con una red neuronal de adherencia simple y valores por propia referencias locales — un diseño diferente.

## 5. Plan de la Fase 1, clave de alineación y prueba de acoplamiento

- **Modalidad que se modela en la Fase 1 (aplanada a tabla):** la **serie**. Cada fila es un (municipio, año-mes) con su tasa, los valores de los meses anteriores y los datos estructurales del municipio. **Paso a la Fase 2:** esa tabla se reemplaza por la forma nativa de cada fuente — la foto satelital entra a una CNN y la serie a una red recurrente (LSTM/GRU) —, incluyendo los dos instrumentos juntos.
- **Clave de alineación (se marca una):**
  
  - Por muestra un **(municipio, año-mes)** es una muestra: cada uno tiene su conteo policial (de la fuente SIEDGO, codificada por código DANE) y su imagen satelital mensual (recortada por el área del municipio, con las geometrías oficiales IGAC que ya tiene el proyecto). Son dos instrumentos distintos midiendo el mismo par territorio-tiempo.
- **Salvedades que declaramos honestamente:**
  1. Los datos de contexto son **anuales** y se repiten en los 12 meses del año — por eso se usan versiones derivadas (log de población, cruce PDET×ZOMAC) para que no degeneren en un mismo número.
  2. La foto satelital mensual tiene **días con nubes**: se cuenta el faltante por (municipio, mes) y se decide imputar con el mes previo.
  3. **No adelantamos información**: la imagen del mes **x** se usa con un desfase de un mes para no usar el mismo evento que estamos prediciendo.
  4. **La sabana tiene huecos**: no todos los municipio-mes están reportados. En **2024** (validación) hay 7.403 pares registrados de un teórico 13.464 (≈ 55 %; apenas ≈ 19 % de municipios tienen los 12 meses). En **2026** (año de prueba, con datos hasta septiembre) la cobertura es aún menor: **4.855 pares de 10.098 posibles (≈ 48 %)**, con entre **539 y 606 municipios reportando por mes** (≈ la mitad de los 1.122 oficiales) y solo **17 % con los 9 meses completos**; septiembre es parcial (280 municipios, mes en proceso de carga). El análisis exploratorio mostrará ese mapa de cobertura por año; **un hueco no significa "cero hurtos"** y por eso no imputamos ceros.
- **Prueba de acoplamiento (obligatoria):** **anexo A — 10 pares (municipio, mes) con datos reales de ambas fuentes.** El dato policial ya está en el documento; la columna de imagen se completa con un script de descarga satelital (cuenta gratuita Earthdata) antes de presentar.
- **Partición:** la serie se obliga a corte cronológico — **entrenamiento 2021–2023 · validación 2024 · prueba 2025–2026** — para que el modelo se evalúe en un "futuro" que nunca vio. Rezagos y transformaciones usan solo información de meses anteriores.

## 6. Línea base comprometida

- **Clase mayoritaria:** exactitud y *recall* de la clase de alerta en el conjunto de prueba; calculadas **solo sobre los pares que sí tienen datos** y reportadas junto al mapa de cobertura del panel.
- **Cursor crudo:** la **tasa del mes inmediatamente anterior** usada sola (si ese valor supera el P75 del municipio, alerta) — umbral elegido en validación; desempeño reportado en prueba. Ideal para saber si la red aporta algo que la intuición del mes pasado no entrega.
- La partición, la semilla (42) y estas cifras quedan guardadas: son el punto de comparación de la Fase 2.

## 7. Riesgos éticos y protección de datos

| Riesgo | Daño para el cliente | Mitigación |
| --- | --- | --- |
| **La fuente mide denuncias, no el delito real** — subregistro según la confianza en la Policía, especialmente rural. | Reforzar vigilancia en municipios que reporta más por cómo se denuncia, no por dónde se delinque más. | Trabajo por **tasa relativa al propio municipio** (P75 local), métricas detalladas por clase, y una segunda mirada en el informe con la Encuesta de Convivencia y Seguridad (ECSC). |
| **Sabana con huecos** — ni todos los meses ni todos los municipios tienen registros (≈ 55 % del panel en 2024). | Silencio sistemático hacia municipios con menor capacidad administrativa de reporte; "sin registro" ≠ "sin hurtos". | Mapa de cobertura en el EDA; métricas calculadas solo sobre pares reales; **nunca imputar ceros**; umbral con ajuste que exige un mínimo de meses; cifras de cobertura junto a resultados por departamento. |
| **Falsa alarma (FP)** | Movilización de recursos a municipios sin repunte; estigma del "municipio criminal" (turismo, inversión). | Umbral decidido en **validación** con la matriz de costos del cliente; **verificación humana** de cada alerta en el panel del cliente antes de actuar; revisión de si los FP se están concentrando en unos departamentos. |
| **Alarma omitida (FN)** | Víctimas sin aviso: el daño ya ocurrió cuando alguien lo detectó a mano. | Priorizar el **recall de la clase de alerta** dentro del costo de FP aceptado; reportar la curva de costo al elegir el umbral. |
| **Privacidad (Ley 1581)** | Reidentificación por combinación de variables. | Checklist del anexo C por cada fuente; sin personas identificables (todo está agrupado por municipio); **no se pegan registros reales con datos personales a asistentes de IA**. |

## 8. Cita APA 7 de cada fuente

- Policía Nacional de Colombia, Dirección de Investigación Criminal e Interpol (DIJIN). (2025). *Reporte hurto por modalidades (modalidad: hurto a personas)* [Conjunto de datos]. Portal SIEDCO: https://portalsiedco.policia.gov.co:4443/extensions/PortalPublico/index.html#/Indicadores/Delito · copia oficial en Datos Abiertos: https://www.datos.gov.co/resource/d4fr-sbn2.json
- Departamento Administrativo Nacional de Estadística. (2025). *Proyecciones de población por municipio* [Conjunto de datos]. https://www.dane.gov.co/index.php/estadisticas-por-tema/demografia-y-poblacion
- Departamento Nacional de Planeación. (s. f.). *Municipios PDET* [Conjunto de datos]. Datos Abiertos Colombia. https://www.datos.gov.co/resource/idrk-ba8y
- Departamento Nacional de Planeación. (s. f.). *Zonas más Afectadas por el Conflicto Armado (ZOMAC) — Municipios ZOMAC (Ves 42)*. Portal DNP.
- Departamento Administrativo Nacional de Estadística. (2025). *Producto interno bruto departamental* [Conjunto de datos]. https://www.dane.gov.co/index.php/estadisticas-por-tema/cuentas-nacionales
- Román, M. O., Yin, J., et al. (2018). NASA's Black Marble nighttime lights product suite. *Remote Sensing of Environment*, *210*, 113–143. https://doi.org/10.1016/j.rse.2018.03.005
- Henderson, J. V., Storeygard, A. y Weil, D. N. (2012). Measuring economic growth from outer space. *American Economic Review*, *102*(2), 994–1028. https://doi.org/10.1257/aer.102.2.994
- Congreso de la República de Colombia. (2012). *Ley Estatutaria 1581 de 2012* — protección de datos personales. https://www.funcionpublica.gov.co/eva/gestornormativo/norma.php?i=49981

---

## Anexo A · Prueba de acoplamiento

Clave: **(código DANE del municipio, año-mes)**

Las filas policales vienen del reporte oficial (huella en el anexo B). El dato de imagen es la **luz promedio del municipio en ese mes**, que se producirá con el script de satélites (requiere registrar cuenta gratuita NASA Earthdata; se llena antes de presentar al docente).

| N.º | Municipio · código DANE | Año-mes | Dato de imagen (OBLIGATORIO por completar) | Conteo de hurtos a personas |
| --- | --- | --- | --- | --- |
| 1 | BOGOTÁ D.C. (CT) · 11001000 | 2024-04 | luz promedio VIIRS 04/2024 — *por completar* | 11.970 |
| 2 | BOGOTÁ D.C. (CT) · 11001000 | 2024-08 | luz promedio VIIRS 08/2024 — *por completar* | 10.742 |
| 3 | MEDELLÍN (CT) · 5001000 | 2024-04 | luz promedio VIIRS 04/2024 — *por completar* | 2.358 |
| 4 | MEDELLÍN (CT) · 5001000 | 2024-05 | luz promedio VIIRS 05/2024 — *por completar* | 2.474 |
| 5 | MEDELLÍN (CT) · 5001000 | 2024-02 | luz promedio VIIRS 02/2024 — *por completar* | 2.266 |
| 6 | BUENAVENTURA · 76109000 | 2024-04 | luz promedio VIIRS 04/2024 — *por completar* | 46 |
| 7 | BUENAVENTURA · 76109000 | 2024-08 | luz promedio VIIRS 08/2024 — *por completar* | 49 |
| 8 | BUENAVENTURA · 76109000 | 2024-01 | luz promedio VIIRS 01/2024 — *por completar* | 54 |
| 9 | ACHÍ · 13006000 | 2024-06 | luz promedio VIIRS 06/2024 — *por completar* | 1 |
| 10 | ACHÍ · 13006000 | 2024-12 | luz promedio VIIRS 12/2024 — *por completar* | 1 |

> Notas: los conteos pertenecen a la **unidad principal del municipio** (los códigos con sufijo `.000`); otros códigos del mismo nombre son subdivisiones y se unifican en el análisis. Los meses sin registro (p. ej., Achí-Enero) no produce fila: ese hueco es parte del dato real. La luz VIIRS es el promedio de brillantez dentro del polígono del municipio (no se necesitan coordenadas de los hechos).

Código que produce las filas (celda del cuaderno):

```python
import pyarrow.parquet as pq
df = pq.read_table('data/hurtos_personas.parquet').to_pandas()   # huella en anexo B
muni = df.groupby(['anio','mes','codigo_dane','municipio','departamento'], as_index=False)['cantidad'].sum()
codes = [11001000, 5001000, 76109000, 13006000]                  # municipios de la muestra
r10 = muni[(muni.anio==2024) & muni.codigo_dane.isin(codes)]
```

## Anexo B · Evidencias del test de procedencia

### B.1 Cadena de procedencia (un archivo por fila)

| Archivo (nombre exacto) | URL del productor primario | Fecha de descarga | Huella SHA-256 |
| --- | --- | --- | --- |
| `hurtos_personas.parquet` (copia intermedia 2021–2025, modalidad *HURTO A PERSONAS*) | Portal SIEDCO (Policía Nacional): https://portalsiedco.policia.gov.co:4443/extensions/PortalPublico/index.html#/Indicadores/Delito · copia oficial: https://www.datos.gov.co/resource/d4fr-sbn2.json | 2025 (carga del trabajo de grado) | `6665d5d84d1703be7246849f541a1dbcf515b4214f63e86d4e2527939a06894b` |
| `hurtos_personas_2026.parquet` (actualización 2026) | ídem | 2026 (carga del trabajo de grado) | `2a8c6086d7963c07cd9c7bc8449060a8a94e12e9529cd63e7aa8931875917740` |
| `consolidado_municipal.parquet` (población/pdet/zomac/PIB por municipio) | DANE y DNP (ver §3) | *(se calcula al exportar en el cuaderno)* | *(se calcula con hashlib en celda)* |
| `VNP46A3_*.hdf` (archivos satelitales mensuales) | https://ladsweb.modaps.eosdis.nasa.gov/ | *(al ejecutar el script con cuenta Earthdata)* | *(se calcula con hashlib en celda)* |

### B.2 Descarga trazable

El cuaderno de la Fase 1 **no** usa copias intermedias como fuente: descarga la tabla oficial desde la API pública `https://www.datos.gov.co/resource/d4fr-sbn2.json`. Si ya existe un archivo previo, solo se reutiliza si su **huella SHA-256** (la "huella digital" del archivo, calculada en el cuaderno con hashlib) coincide con la de la descarga verificada. Verificación anti-copia del paquete entregado: `grep -inE "kaggle|kagglehub|/kaggle/input" *.ipynb` → **cero coincidencias**.

### B.3 Búsqueda declarada de espejos (2026-10-29)

Búsquedas realizadas en `kaggle.com/datasets` y con `site:kaggle.com`: «colombia hurto personas», «reporte hurto por modalidades». Resultado: **sin copias de este estudio** (no hay paquete con la tabla mensual por municipio y su etiqueta de alerta por P75). Las capturas se adjuntan con la entrega.

### B.4 Huella de crudeza (antes de limpiar)

| Archivo | Filas × columnas | % faltantes | Crudeza observada |
| --- | --- | --- | --- |
| `hurtos_personas.parquet` | 735.802 × 13 | 0,00 % | Escalas muy distintas (Achí con 1 evento/mes vs. Bogotá con más de 10.000); nombres de municipios con variantes (`(CT)`) y códigos DANE de 7–8 dígitos que hay que normalizar a 5; categorías de zona/armas/edad que hay que unificar; desbalance extremo de la clase de alerta; **huecos de cobertura** (≈ 55 % de los pares municipio-mes de 2024 reportados; solo ≈ 19 % de municipios con 12 meses) |
| `hurtos_personas_2026.parquet` | 83.778 × 13 | 0,00 % | igual que el anterior; sirve como "futuro" de la partición. **Año parcial**: meses enero–septiembre; cobertura aún menor (≈ 48 % de los pares municipio-mes posibles; ≈ la mitad de municipios por mes; septiembre parcial con 280 municipios) |
| `poblacion_municipio.parquet` | 83 × 7 | 0,00 % | cobertura solo barrial (Buenaventura) — **no** se usa en la Fase 1 nacional |

> El 0 % de nulos **no** significa "demasiado limpio": es natural en un reporte oficial consolidado. La crudeza exigible está en normalizar códigos, unificar categorías, armar la etiqueta (P75 por municipio con ajuste), el desbalance de clases y los **huecos de la sabana**.

### B.5 Contraste con el espejo (si existe)

No hay republicación conocida a este nivel de agregado. El paralelo más cercano es el mismo reporte DIJIN sin transformar (las mismas columnas); con la huella de B.1 se demuestra que la copia usada no coincide con ningún paquete «listo para modelar».

## Anexo C · *Checklist* de protección de datos personales (Ley 1581 de 2012)

Se entrega el checklist una vez por fuente. **Regla general: la Fase 1 exige que no haya personas identificables.**

### Fuente 1 — Reporte Hurto por Modalidades (Policía Nacional / DIJIN)

**Resultado:** ☑ **Sin datos de personas** (hechos anonimizados por la fuente oficial).

- [x] **A. ¿Datos personales?** No: sin nombres, documentos, teléfonos ni direcciones; tampoco coordenadas por evento (solo código DANE del municipio). Son conteos de hechos.
- [x] **B. ¿Datos sensibles?** No viene de personas como salud o biometría.
- [x] **C. ¿Menores?** No se identifican; el grupoetario es genérico por hecho.
- [x] **D. ¿Base legal/terceros?** Fuente oficial pública con términos citables.
- [x] **E. Finalidad:** "uso académico para entrenar y evaluar un modelo en el curso; no se toman decisiones sobre personas identificadas".
- [x] **F. ¿Riesgo de reidentificar?** Bajo: todo agrupado por municipio; no se publican tablas con conteos de contados de puestos muy pequeños a nivel de vereda o barrio ni fechas exactas de submunicipios.
- [x] **G. Licencia:** CC BY-SA 4.0 — obligar de citar (Policía Nacional); cualquier derivado redistribuido comparte la misma licencia.
- [x] **H. Transferencia internacional:** no se suben registros con datos personales a asistentes de IA (incluida esta conversación); solo se comparten esquemas o filas simuladas si fuese útil.
- [x] **I. Minimización y conservación:** solo las columnas necesarias; los archivos crudos se guardan localmente y se **borran al cerrar el curso**.
- [x] **J. Repositorio:** será **privado compartido con el docente** (o público sin los datos, con script de descarga y `data/` en `.gitignore`).

### Fuente 2 — DANE (población, PIB)

**Resultado:** ☑ **Sin datos de personas** — cifras por municipio o departamento. La finalidad, licencia y todos los ítems A–J valen los mismos principios anteriores sin moneda: no vienen de una persona identificable en el archivo.

### Fuente 3 — DNP (PDET, ZOMAC)

**Resultado:** ☑ **Sin datos de personas** — es una clasificación por municipio. Sin riesgo de reidentificar; citable con atribución.

### Fuente 4 — NASA Black Marble VIIRS

**Resultado:** ☑ **Sin datos de personas** — son fotografías satelitales de grupo lumínico. El recorte se hace a nivel del polígono municipal (nunca de vereda). "Earthdata" es un registro gratuito; se citan los términos de la fuente.

**Declaración.** Lo consignado en este checklist es verdadero y se revisó para cada fuente.

| Integrante | Firma | Fecha |
| --- | --- | --- |
| Yeison David Toro | | 2026-10-29 |
| Juan Sebastián Ulloa | | 2026-10-29 |
