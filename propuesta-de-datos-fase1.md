# Propuesta de Datos · Proyecto Hurto Personas Centinela· Fase 1

| Campo | Detalle |
| --- | --- |
| Título del proyecto | **Centinela de Hurtos** · Línea base tabular (MLP) de alerta temprana del riesgo de hurto a personas por municipio y mes |
| Integrantes | Yeison David Toro — Juan Sebastian Ulloa |
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
| Temporal (principal) | **Policía Nacional — DIJIN/SIEDCO**: *Reporte Hurto por Modalidades*. El proyecto usa **solo la modalidad «HURTO A PERSONAS»** del conjunto | Portal SIEDCO: https://portalsiedco.policia.gov.co:4443/extensions/PortalPublico/index.html#/Indicadores/Delito | **CC BY-SA 4.0** (libre con atribución a la Policía) | Notas del portal SIEDCO y de su ficha pública (2026-11-04) | **No** (hechos anonimizados, sin nombres ni documentos) |
| Temporal (contexto) | **DANE** — proyecciones de población por municipio (es el denominator de la tasa) | https://www.dane.gov.co/index.php/estadisticas-por-tema/demografia-y-poblacion | Dato oficial público (uso estadístico) | Sitio DANE | **No** (cifras por municipio) |
| Temporal (contexto) | **ART** — municipios PDET · **DNP** — ZOMAC · **DANE** — PIB departamental | PDET: https://www.datos.gov.co/resource/idrk-ba8y · ZOMAC y PIB: portales DNP y DANE | PDET: **CC BY-SA 4.0** · ZOMAC/PIB: licencia por verificar en la ficha del productor | Ficha `api/views/idrk-ba8y.json` (PDET —productor: Agencia de Renovación del Territorio—, 2026-11-04); fichas DNP/DANE por anotar | **No** (cifras por municipio) |
| Origen del cuaderno de la Fase 1 | **API pública del propio panel del proyecto** — `bronze` (eventos/municipio por mes) y `gold` (tasas, P25/P50/P75, población DANE) | https://intelligence-territorial-panel.vercel.app/api/dbx/bronze · `…/gold` | **Derivada de la fuente principal**: los datos que publica esta API se obtuvieron del **portal SIEDCO** (productor primario). Por el *share-alike* de **CC BY-SA 4.0**, las salidas publicadas del panel comparten esa licencia | Fuente principal (fila Temporal de esta tabla) + huellas de descarga en el anexo B |
| Imagen | **NASA Black Marble (VIIRS)** — compuestos **mensuales** de luces nocturnas (**VNP46A3, Collection 2.0**) | https://blackmarble.gsfc.nasa.gov/ · descarga: https://ladsweb.modaps.eosdis.nasa.gov/ · datos: https://doi.org/10.5067/VIIRS/VNP46A3.002 | Datos NASA de libre uso (registro gratuito en Earthdata) | Página del producto en LADS | **No** (fotos satelitales de luz) |

> Nota: el trabajo de grado del que hereda este proyecto ya trabajó con una copia (parquet) del reporte de la Policía; el cuaderno de la Fase 1 **no depende de** esa copia: descarga de la **API del panel**, cuya fuente es la oficial (portal SIEDCO). Cadena de provenance: **portal SIEDCO (productor primario) → API del panel (bronze/gold) → cuaderno de la Fase 1**; cualquier persona puede reproducirlo y las huellas quedan en el anexo B.

## 4. Por qué NO es un reto resuelto

- **Filtro del estudio:** el reporte oficial incluye **todas** las modalidades de crimen (amenazas, homicidios, lesiones personales, ...). Aquí se procesa **exclusivamente «HURTO A PERSONAS»**, que es donde una persona es la víctima directa. En la ventana publicada por la API (2021 + ene–feb 2022) quedan **330.374 eventos** (179.594 filas agregadas por municipio/dimensiones).
- La fuente es **cruda**: hay que normalizar los códigos de municipio, unificar categorías, calcular la tasa por municipio-mes, definir el umbral con su ajuste para municipios con pocos meses y armar la partición temporal. **Ningún archivo público (ni el portal SIEDCO, ni datos.gov.co, ni Kaggle) entrega esa tabla mensual ya lista con la etiqueta de alerta por P75.**
- No hay copia de competencia de Kaggle a la que atribuirlo: el productor primario es la Policía Nacional. Las búsquedas de espejos quedan documentadas con fecha en el anexo B.
- El trabajo de grado (XGBoost para estimar la tasa)  **no resuelve** esta tarea: aquí el problema es **binario** (alerta sí/no) con una red neuronal de arquitectura sencilla y umbrales por referencias locales — un diseño diferente.

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
- **Partición:** la serie se obliga a corte cronológico — **entrenamiento 2021 · validación ene-2022 · prueba feb-2022** (la ventana publicada por la API en la fecha de corrida) — para que el modelo se evalúe en un "futuro" que nunca vio. Rezagos y transformaciones usan solo información de meses anteriores. La ventana está en constantes del cuaderno: cuando el panel publique los años restantes, todo el flujo se re-ejecuta sin cambios de código.

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

- Policía Nacional de Colombia, Dirección de Investigación Criminal e Interpol (DIJIN). (2025). *Reporte hurto por modalidades* [Conjunto de datos]. Portal SIEDCO: https://portalsiedco.policia.gov.co:4443/extensions/PortalPublico/index.html#/Indicadores/Delito

- Departamento Administrativo Nacional de Estadística. (2025). *Proyecciones de población por municipio* [Conjunto de datos]. https://www.dane.gov.co/index.php/estadisticas-por-tema/demografia-y-poblacion
- Agencia de Renovación del Territorio. (s. f.). *Municipios PDET* [Conjunto de datos]. Datos Abiertos Colombia. https://www.datos.gov.co/resource/idrk-ba8y
- Departamento Nacional de Planeación. (s. f.). *Zonas más Afectadas por el Conflicto Armado (ZOMAC) — Municipios ZOMAC (Ves 42)*. Portal DNP.
- Departamento Administrativo Nacional de Estadística. (2025). *Producto interno bruto departamental* [Conjunto de datos]. https://www.dane.gov.co/index.php/estadisticas-por-tema/cuentas-nacionales
- Román, M. O., Yin, J., et al. (2018). NASA's Black Marble nighttime lights product suite. *Remote Sensing of Environment*, *210*, 113–143. https://doi.org/10.1016/j.rse.2018.03.017
- Congreso de la República de Colombia. (2012). *Ley Estatutaria 1581 de 2012* — protección de datos personales. https://www.funcionpublica.gov.co/eva/gestornormativo/norma.php?i=49981


| Integrante | Firma | Fecha |
| --- | --- | --- |
| Yeison David Toro | Yeison Toro| 2026-10-29 |
| Juan Sebastián Ulloa | Juan Ulloa| 2026-10-29 |

## Anexo A · Prueba de acoplamiento

Clave: **(departamento, municipio, año-mes)** — el departamento entra en la clave porque 257 municipios comparten nombre en distintos departamentos. Los pares están dentro de la ventana publicada por la API (2021 + ene–feb 2022; huella `e46ba559…`, anexo B).

Las filas policales vienen del reporte oficial a través del API del proyecto. El dato de imagen es la **luz promedio del municipio en ese mes** (VNP46A3 Collection 2.0, DOI 10.5067/VIIRS/VNP46A3.002), que se produce con el script de satélites sobre el polígono MGN del IGAC (requiere cuenta gratuita NASA Earthdata; obligatoriamente llenado antes de presentar al docente).

| N.º | Departamento · municipio · código DANE | Año-mes | Dato de imagen (OBLIGATORIO por completar) | Conteo de hurtos a personas (API, verificado 2026-11-04) |
| --- | --- | --- | --- | --- |
| 1 | CUNDINAMARCA · BOGOTÁ D.C. (CT) · 11001000 | 2021-05 | luz promedio VIIRS 05/2021 — *por completar* | 8.839 |
| 2 | CUNDINAMARCA · BOGOTÁ D.C. (CT) · 11001000 | 2021-11 | luz promedio VIIRS 11/2021 — *por completar* | 11.013 |
| 3 | ANTIOQUIA · MEDELLÍN (CT) · 05001000 | 2021-03 | luz promedio VIIRS 03/2021 — *por completar* | 1.897 |
| 4 | ANTIOQUIA · MEDELLÍN (CT) · 05001000 | 2021-08 | luz promedio VIIRS 08/2021 — *por completar* | 2.320 |
| 5 | ANTIOQUIA · MEDELLÍN (CT) · 05001000 | 2022-02 | luz promedio VIIRS 02/2022 — *por completar* | 2.345 |
| 6 | VALLE · BUENAVENTURA · 76109 (y 76109098) | 2021-01 | luz promedio VIIRS 01/2021 — *por completar* | 51 |
| 7 | VALLE · BUENAVENTURA · 76109 (y 76109098) | 2021-07 | luz promedio VIIRS 07/2021 — *por completar* | 67 |
| 8 | VALLE · BUENAVENTURA · 76109 (y 76109098) | 2022-01 | luz promedio VIIRS 01/2022 — *por completar* | 75 |
| 9 | BOLÍVAR · ACHÍ · 13006000 | 2021-02 | luz promedio VIIRS 02/2021 — *por completar* | 1 |
| 10 | BOLÍVAR · ACHÍ · 13006000 | 2021-08 | luz promedio VIIRS 08/2021 — *por completar* | 1 |

> Notas: los conteos pertenecen a la **unidad principal del municipio** (los códigos con sufijo `.000`); otros códigos del mismo nombre son subdivisiones y se unifican en el análisis. Los meses sin registro (p. ej., Achí-Enero) no produce fila: ese hueco es parte del dato real. La luz VIIRS es el promedio de brillantez dentro del polígono del municipio (no se necesitan coordenadas de los hechos).

Código que produce las filas (celda del cuaderno — mismo origen que el modelo: la API del panel):

```python
import json, urllib.request
BRONZE = "https://intelligence-territorial-panel.vercel.app/api/dbx/bronze"   # huella en anexo B
eventos = json.load(urllib.request.urlopen(BRONZE, timeout=60))["response"]

MUESTRAS = {("CUNDINAMARCA", "BOGOTÁ D.C. (CT)"), ("ANTIOQUIA", "MEDELLÍN (CT)"),
            ("VALLE", "BUENAVENTURA"), ("BOLÍVAR", "ACHÍ")}
r10 = {}
for e in eventos:
    k = (e["departamento"], e["municipio"])
    if k in MUESTRAS:
        kk = k + (int(e["anio"]), int(e["mes"]))
        r10[kk] = r10.get(kk, 0) + int(e["cantidad"])
print(r10)   # 45 pares (todos los meses de la ventana); la tabla del anexo A selecciona 10 de ellos
```
> Nota: es una **prueba de verificación de la clave** (no el entrenamiento): el cuaderno que produce la tabla y el que entrena el modelo leen la misma fuente con la misma huella.

## Anexo B · Evidencias del test de procedencia

### B.1 Cadena de procedencia (un archivo por fila)

| Archivo / servicio (nombre exacto) | URL del productor / servidor | Fecha de descarga/verificación | Huella SHA-256 |
| --- | --- | --- | --- |
| **API del panel** — `bronze` (eventos por municipio-mes; la fuente del cuaderno de la Fase 1) | https://intelligence-territorial-panel.vercel.app/api/dbx/bronze | 2026-11-04 | `e46ba559f2204e78…` (huella de la descarga completa en la celda de carga del cuaderno) |
| **API del panel** — `gold` (tasas, percentiles P25/P50/P75 y población DANE por municipio) | https://intelligence-territorial-panel.vercel.app/api/dbx/gold | 2026-11-04 | `912b0add433d9390…` |
| `VNP46A3_*.hdf` (archivos satelitales mensuales, Collection 2.0; DOI 10.5067/VIIRS/VNP46A3.002) | https://ladsweb.modaps.eosdis.nasa.gov/ | *(al ejecutar el script con cuenta Earthdata)* | *(se calcula con hashlib en celda)* |

### B.2 Descarga trazable

El cuaderno de la Fase 1 **no** usa copias intermedias como fuente: descarga la tabla del **API pública del panel** (bronze/gold, señalada en §3) — un derivado del conjunto principal, cuyas salidas publicadas comparten su licencia CC BY-SA 4.0. Si ya existe un archivo previo, solo se reutiliza si su **huella SHA-256** (la "huella digital" del archivo, calculada en el cuaderno con hashlib) coincide con la de la descarga verificada. Verificación anti-copia del paquete entregado: `grep -inE "kaggle|kagglehub|/kaggle/input" *.ipynb` → **cero coincidencias**.

### B.3 Búsqueda declarada de espejos (2026-10-29)

Búsquedas realizadas en `kaggle.com/datasets` y con `site:kaggle.com`: «colombia hurto personas», «reporte hurto por modalidades». Resultado: **sin copias de este estudio** (no hay paquete con la tabla mensual por municipio y su etiqueta de alerta por P75). Las capturas se adjuntan con la entrega.

### B.4 Huella de crudeza (antes de limpiar)

| Servicio | Filas × columnas | % faltantes | Crudeza observada |
| --- | --- | --- | --- |
| API `bronze` (descarga 2026-11-04) | 179.594 × 13 | 0,00 % | Escalas muy distintas (Achí con 1 evento/mes vs. Bogotá con más de 10.000); nombres de municipios repetidos entre departamentos (clave `(departamento, municipio)`) y códigos DANE de 7–8 dígitos con variantes y subdivisiones (`…098`) que hay que unificar; dimensiones crudas (genero/zona/armas/edad sin diccionario verificado — no se usan como features); desbalance de la clase alerta (~22–25 %); **ventana limitada**: 2021 + ene–feb 2022 (marzo 2022 parcial); municipios con meses incompletos (solo 24 % con los 12 de 2021) |
| API `gold` (descarga 2026-11-04) | 9.909 × 19 | 60 filas sin `tasa_real` (mes parcial) | Solo año 2026 (enero–septiembre): se usa **únicamente** la columna de población (`pob_dane_2025`) como denominador; las salidas del panel (`tasa_predicha`, niveles de riesgo) se ignoran para no introducir sus resultados como entrada |

> El 0 % de nulos **no** significa "demasiado limpio": es natural en un reporte oficial consolidado. La crudeza exigible está en normalizar códigos, unificar categorías, armar la etiqueta (P75 por municipio con ajuste), el desbalance de clases y los **huecos de la sabana**.

### B.5 Contraste con el espejo (si existe)

No hay republicación conocida a este nivel de agregado. El paralelo más cercano es el mismo reporte DIJIN sin transformar (las mismas columnas); con la huella de B.1 se demuestra que la copia usada no coincide con ningún paquete «listo para modelar».

## Anexo C · *Checklist* de protección de datos personales (Ley 1581 de 2012)

Se entrega el checklist una vez por fuente. **Regla general: la Fase 1 exige que no haya personas identificables.**

### Fuente 1 — Reporte Hurto por Modalidades (Policía Nacional / DIJIN)

**Resultado:** ☑ **Sin datos de personas** (hechos anonimizados por la fuente oficial).

- [x] **A. ¿Datos personales?** No: sin nombres, documentos, teléfonos ni direcciones; tampoco coordenadas por evento (solo código DANE del municipio). Son conteos de hechos.
- [x] **B. ¿Datos sensibles?** No viene de personas como salud o biometría.
- [x] **C. ¿Menores?** El conjunto principal en uso (API del panel, agregado por municipio-mes) **no publica atributos de víctimas** (sin edad, zona ni medio). La copia intermedia del trabajo de grado sí trae **grupo etario** — junto con fecha y municipio, identificadores indirectos —: esos datos no se publican y sus columnas se ignoran al cargar.
- [x] **D. ¿Base legal/terceros?** Fuente oficial pública con términos citables.
- [x] **E. Finalidad:** "uso académico para entrenar y evaluar un modelo en el curso; no se toman decisiones sobre personas identificadas".
- [x] **F. ¿Riesgo de reidentificar?** Bajo: todo agrupado por municipio; no se publican tablas con conteos de contados de puestos muy pequeños a nivel de vereda o barrio ni fechas exactas de submunicipios.
- [x] **G. Licencia:** CC BY-SA 4.0 — obligar de citar (Policía Nacional); cualquier derivado redistribuido comparte la misma licencia.
- [x] **H. Transferencia internacional:** no se suben registros con datos personales a asistentes de IA (incluida esta conversación); solo se comparten esquemas o filas simuladas si fuese útil.
- [x] **I. Minimización y conservación:** solo las columnas necesarias; los archivos crudos se guardan localmente y se **borran al cerrar el curso**.
- [x] **J. Repositorio:** será **privado compartido con el docente** (o público sin los datos, con script de descarga y `data/` en `.gitignore`).

### Fuente 2 — DANE (población, PIB)

**Resultado:** ☑ **Sin datos de personas** — cifras por municipio o departamento. La finalidad, licencia y todos los ítems A–J valen los mismos principios anteriores sin moneda: no vienen de una persona identificable en el archivo.

### Fuente 3 — ART (PDET) y DNP (ZOMAC)

**Resultado:** ☑ **Sin datos de personas** — es una clasificación por municipio. Sin riesgo de reidentificar; citable con atribución.

### Fuente 4 — NASA Black Marble VIIRS (**VNP46A3 Collection 2.0**; DOI 10.5067/VIIRS/VNP46A3.002)

**Resultado:** ☑ **Sin datos de personas** — son composiciones satelitales de luz nocturna. El recorte se hace a nivel del polígono municipal (nunca de vereda). "Earthdata" es un registro gratuito; se citan los términos de la fuente.

**Declaración.** Lo consignado en este checklist es verdadero y se revisó para cada fuente.

| Integrante | Firma | Fecha |
| --- | --- | --- |
| Yeison David Toro | | 2026-10-29 |
| Juan Sebastián Ulloa | | 2026-10-29 |
