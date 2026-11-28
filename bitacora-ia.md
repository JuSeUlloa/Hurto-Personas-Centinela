# Bitácora de IA · Proyecto Centinela

La bitácora se diligencia a lo largo del proyecto, no al final. No se registra cada indicación (*prompt*): se documentan las interacciones que cambiaron una decisión técnica (arquitectura, datos, métricas, un error resuelto, un enfoque descartado). Cada entrada lleva la fecha y la hora de la consulta, anotadas cuando ocurre.

A un asistente de IA no se le pegan ni se le suben registros con datos personales ni datos de terceros sin su permiso: se le muestran el esquema y filas sintéticas, y la bitácora registra qué datos se compartieron.

## 1. Datos de la entrega

| Campo | Detalle |
| --- | --- |
| **Integrantes** | Yeison David Toro — Juan Sebastián Ulloa |
| **Proyecto** | **Centinela de Hurtos** — alerta temprana del repunte mensual de hurto a personas por municipio (tasa > P75 del propio municipio) |
| **Fase** | Fase 1 · Línea base con MLP |
| **Primer identificador de esta fase** | E-1 |
| **Fecha de entrega** | 2026-11-05 |

## 2. Declaración de uso de IA

### Declaración de uso (la firma cada integrante)

Quienes firman declaran que durante esta fase usaron las siguientes herramientas de IA generativa:

- **OpenCode · MiMo-V2.6-Flash, sesiones del 2026-11-02 al 2026-11-03** → para co-desarrollar el cuaderno de la Fase 1 (arquitectura del MLP, función de pérdida y bucle de entrenamiento), ejecutar y validar los experimentos (barrido de tasa de aprendizaje, sobreajuste/Dropout, ablación de features), auditar los errores del propio desarrollo y preparar la documentación (relacionada con la propuesta de datos).
- **OpenCode · MiMo-V2.6-Flash, sesión del 2026-11-04** → para revisión de fuentes y licencias de la Propuesta de Datos (licencias nominales y DOI de NASA VNP46A3).

Partes del entregable redactadas o programadas sin IA: la verificación manual de los conteos del Anexo A contra la fuente oficial por consulta directa (SODA), el texto de la matriz de costos del cliente (FN/FP) y la propuesta, checklist Ley 1581 y la decisión final de calibración del umbral de decisión.

Quienes firman declaran también que **comprenden, verificaron y pueden defender** como propio cada artefacto entregado, que cada integrante responde en la sustentación por una parte distinta del trabajo y que esta bitácora refleja con honestidad cómo se usaron estas herramientas.

| Integrante | Firma | Fecha |
| --- | --- | --- |
| Yeison David Toro | ______________________ | 2026-11-05 |
| Juan Sebastián Ulloa | ______________________ | 2026-11-05 |

## 3. Registro de interacciones

Cada interacción se marca como **Aceptada · Modificada · Rechazada**. La evidencia de origen (la salida literal de la IA, o una captura de pantalla con fecha y herramienta) es obligatoria en las entradas Modificada y Rechazada; en las Aceptadas es opcional. En este proyecto, la evidencia literal se conserva re-ejecutada en `bitacora/evidencia/*.txt` (salidas literales de las corridas de decisión, con su semilla).

**Cuántas entradas.** Para llegar a Básico, el Criterio 5 pide al menos tres entradas fechadas, con su decisión y una justificación en cada una. Este registro tiene **7** (E-1 a E-7).

> **Nota de comando**: el identificador de cada entrada aparece como comentario en la primera línea de la celda del cuaderno que materializa la decisión (`# E-n`).

### E-1 · Elección del optimizador Adam

| Campo | Registro |
| --- | --- |
| **ID** | E-1 |
| **Fecha y hora** | 2026-11-02 (sesión nocturna) |
| **Fase · paso** | F1 · Sección 4 (pérdida, optimizador y bucle) |
| **Integrante que hizo la consulta** | Juan Sebastián Ulloa |
| **Herramienta y modelo** | OpenCode · MiMo-V2.6-Flash |
| **Celda o *commit* afectado** | Cuaderno `02_scaffold_centinela_fase1.ipynb`, celda de la función `entrenar` · etiqueta `# E-1` |
| **Objetivo de la consulta** | Elegir el optimizador del MLP de la línea base (baseline §6 de la propuesta) |
| **Indicación (resumen)** | "Con el MLP de la Fase 1, elige entre SGD y Adam y justifícalo con la teoría del curso; la pérdida lleva pos_weight para el costo asimétrico FN/FP" |
| **Qué devolvió la IA** | Recomendación de **Adam** con lr≈0.01 como punto de partida: (i) pasos adaptativos por pesos que equilibran el gradiente en features de escala distinta (rezagos vs. medias móviles), (ii) robusto con lote completo (full-batch) en 2.989 muestras sin ajuste fino de schedule, (iii) la alternativa SGD+momentum exige una lr global más cuidadosa en presencia del pos_weight, sin ganancia esperable con solo 401 parámetros. Descarto de RMSprop/AdamW: AdamW (decaimiento desacoplado) queda anotado como opción de la Fase 2 si se añade regularización L2 (Loshchilov & Hutter, 2019) |
| **Datos compartidos con la IA** | Solo el esquema (conteos: 720 alertas / 2.269 no-alertas en train) y el diseño de features — ninguna fila individual |
| **Decisión** | **Aceptada** |
| **Justificación** | Teoría del curso (Módulo 1, Cap. 3; Goodfellow et al., 2016, cap. 8): Adam ajusta la tasa adaptativa para las características con diferencias de escala; con `pos_weight≈3.2` la trayectoria de optimizer es sensible a la escala y Adam la absorbe sin un schedule manual. La justificación final está en la evidencia del barrido (E-2) |
| **Cómo se verificó** | El barrido E-2 se ejecuta con Adam (misma arquitectura); el modelo estable convergió en 50 épocas (pérdida train 1,02 / val 1,19 — verificado en el cuaderno) |
| **Evidencia de origen** | (No obligatoria en Aceptadas; ver barrido en `bitacora/evidencia/E-2-barrido-lr.txt`) |

### E-2 · Elección de la tasa de aprendizaje (lr) — barrido

| Campo | Registro |
| --- | --- |
| **ID** | E-2 |
| **Fecha y hora** | 2026-11-03 (sesión vespertina) |
| **Fase · paso** | F1 · Sección 4 |
| **Integrante que hizo la consulta** | Juan Sebastián Ulloa |
| **Herramienta y modelo** | OpenCode · MiMo-V2.6-Flash |
| **Celda o *commit* afectado** | Celda `entrenar` — línea del lr (`lr=0.01`) · `# E-2` |
| **Objetivo de la consulta** | Poner a prueba la lr por defecto del material (0,05, usada en el andamio sintético del Módulo 1) sobre el panel nacional real |
| **Indicación (resumen)** | "Con los datos reales de la ventana (2.989/363/347), barre lr y selecciona con evidencia. Criterio del cliente: recall de la clase alerta sin sacrificar exactitud de forma injustificada" |
| **Qué devolvió la IA** | Barrido ejecutado con la misma semilla (42) y la misma arquitectura (6→16→16→1, pos_weight de train): **lr=0.005 → TEST acc 0,61 · rec 0,37**; **lr=0.01 → acc 0,62 · rec 0,42**; **lr=0.05 → acc 0,58 · rec 0,52** (evidencia literal en `bitacora/evidencia/E-2-barrido-lr.txt`) |
| **Datos compartidos con la IA** | Esquema + conteos por partición (720/111/81 alertas); ninguna fila individual |
| **Decisión** | **Modificada** (lr 0,05 → 0,01) |
| **Justificación** | El resultado real mostró cambio, con los datos sintéticos de la Fase 1 (andamio, lr=0,05), la lr funcionaba; en el panel nacional, la lr=0.05 sube el recall a expensa de un margen de aciertos (precisión 0,28 → acc 0,58) — un modelo con demasiadas falsas alarmas frustra la verificación (encontramos que el modelo coherente debe balancear la exactitud y el recall). La lr=0.01 ganó mejor equilibrio. Se registró como compromiso y anotó la alternativa (lr=0.05 si el cliente ajustara la matriz de costos) |
| **Cómo se verificó** | La corrida con barrido (misma semilla en los 3 gastos) y verificación de las mismas métricas del cuaderno entregado (pérdida train 1,02 · val 1,19 · TEST acc 0,62 / recall 0,42) |
| **Evidencia de origen** | **`bitacora/evidencia/E-2-barrido-lr.txt`** (salida literal del barrido; semilla 42) |

### E-3 · Arquitectura del MLP: capas ocultas ReLU y salida logit

| Campo | Registro |
| --- | --- |
| **ID** | E-3 |
| **Fecha y hora** | 2026-11-03 |
| **Fase · paso** | F1 · Sección 3 (TODO: arquitectura del MLP) |
| **Integrante que hizo la consulta** | Juan Sebastián Ulloa |
| **Herramienta y modelo** | OpenCode · MiMo-V2.6-Flash |
| **Celda o *commit* afectado** | Celda `class MLP(nn.Module)` · `# E-3` |
| **Objetivo de la consulta** | Definir la arquitectura que cumple la actividad: ≥2 capa ocultas, justificación de las activaciones, salida adecuada para clasificación binaria |
| **Indicación (resumen)** | "Diseña la arquitectura del MLP para 6 features escaladas y justifica las activaciones en la teoría del Módulo 1" |
| **Qué devolvió la IA** | Diseño **6→16→16→1**: ocultas con **ReLU** (no saturada; derivada 1 en el semiplano positivo; el gradiente llega intacto desde la salida hacia las ocultas), salida **lineal (logit)** entrenada con `BCEWithLogitsLoss` — evita la «doble sigmoide» (el error clásico que la actividad vetó) y aplica el trato log-sum-exp estable. Alternativa sigmoide+tanh descartada en la explicación (con la caída del gradiente de 0,25 por capa máx.) |
| **Datos compartidos con la IA** | Esquema de features (nombres y rol) — sin filas |
| **Decisión** | **Aceptada** |
| **Justificación** | Teoría del curso (Módulo 1, Cap. 2: no linealidad; Goodfellow, 2016, cap. 6) +	 requisito de la actividad sobre la ruta recomendada logit+BCEWithLogitsLoss (con `torch.sigmoid` solo al evaluar) |
| **Cómo se verificó** | 401 parámetros entrenables; el entrenamiento convergió (pérdida descenso en 50 épocas); y las dos capas ReLU producen superficie de decisión no lineal (evaluada por el desempeño 0,62/0,42 del test) |
| **Evidencia de origen** | N/A |

### E-4 · Pérdida BCEWithLogitsLoss con `pos_weight` (costo asimétrico FN/FP)

| Campo | Registro |
| --- | --- |
| **ID** | E-4 |
| **Fecha y hora** | 2026-11-03 |
| **Fase · paso** | F1 · Sección 4 |
| **Integrante que hizo la consulta** | Juan Sebastián Ulloa |
| **Herramienta y modelo** | OpenCode · MiMo-V2.6-Flash |
| **Celda o *commit* afectado** | Celda `entrenar` — línea del `pw` · `# E-4` |
| **Objetivo de la consulta** | Tratar el desbalance de clases de manera que un falso negativo (alerta no detectada) pese más, en línea con la matriz de costos del cliente (propuesta §7) |
| **Indicación (resumen)** | "Con 720 alertas sobre 2.989 muestras de train, monta el error asimétrico: el,FN debe tener un peso mayor que el FP sin balancear por sobres(formación) ni alterar las clases" |
| **Qué devolvió la IA** | Conseguir `pos_weight = n_neg/n_pos = 2.269/720 ≈ 3,15` aplicado dentro de `BCEWithLogitsLoss` (PyTorch oficial). Se descartó sobremuestrear la clase minoritaria y submuestrear las no-alertas: con 2.989 muestras de train no sobran, y ambas alternativas alteran la base de los datos de alertas reales; `pos_weight` logra el mismo efecto sin tirar datos |
| **Datos compartidos con la IA** | Esquema + conteos por clase (720 y no-alertas) — sin filas |
| **Decisión** | **Aceptada** |
| **Justificación** | La redistribución de pesos preserve la misma muestra y teórica de la métrica, y PyTorch permite un único argumento. La intensidad refleja la matriz de costos del cliente: un FN hace más daño que un FP|
| **Cómo se verificó** | El train/val/test con la misma semilla (barrido E-2) | la pérdida 1,02/1,19 y el recall de la clase alerta 0,42 en prueba |
| **Evidencia de origen** | N/A |

### E-5 · Dropout paramétrico y experimento de sobreajuste inducido

| Campo | Registro |
| --- | --- |
| **ID** | E-5 |
| **Fecha y hora** | 2026-11-03 |
| **Fase · paso** | F1 · Sección 6 (TODO: sobreajuste) |
| **Integrante que hizo la consulta** | Juan Sebastián Ulloa |
| **Herramienta y modelo** | OpenCode · MiMo-V2.6-Flash |
| **Celda o *commit* afectado** | Celda de la Sección 6 (`grande` y `con_do`) · `# E-5` |
| **Objetivo de la consulta** | Fabricar el desafío activacional: **inducir** sobreajuste con una red grande sobre pocos datos y **corregirlo** con Dropout, documentado con curvas y matrices en prueba |
| **Indicación (resumen)** | "Inducción sobreajuste aplica una red grande (3×128, 300 épocas) sobre 200 de 2.989 muestras; corrige con `p_dropout` de la clase MLP, usa la misma semilla en ambas para comparar solo la regularización" |
| **Qué devolvió la IA** | Diseño del experimento: submuestreo de 200 muestras con `np.random.default_rng(SEMILLA)`; `grande` (3×128, p=0) y `con_do` (3×128, p=0.30) con **misma semilla** (misma inicialización); 300 épocas, lr=0.01. Resultados (evidencia literal): **sin reg. train 0,94 · val-loss 10,26 / rec-test 0,33**; **con Drop. train 0,75 · val-loss 7,23 / rec-test 0,37** |
| **Datos compartidos con la IA** | Esquema + conteos por partición — sin filas |
| **Decisión** | **Aceptada** |
| **Justificación** | Las mismas semillas solo varían la variable estudio y la actividad pide el exact ponderación inducir la correcta, la brecha train/val queda documentada con las curvas y las matrices de confusión en el conjunto de prueba |
| **Cómo se verificó** | Re-ejecición literal (semilla 42): `bitacora/evidencia/E-5-sobreajuste-dropout.txt`; coincide con las celdas del cuaderno |
| **Evidencia de origen** | **`bitacora/evidencia/E-5-sobreajuste-dropout.txt`** |

### E-6 · Features de calendario (`sin_mes`/`cos_mes`): propuestas y luego retiradas

| Campo | Registro |
| --- | --- |
| **ID** | E-6 |
| **Fecha y hora** | 2026-11-04 |
| **Fase · paso** | F1 · Secciones 1 (features) y 5 (ablación) |
| **Integrante que hizo la consulta** | Juan Sebastián Ulloa |
| **Herramienta y modelo** | OpenCode · GLM-5.3-Flash |
| **Celda o *commit* afectado** | Celda de la ablación (`modelo8`) · `# E-6` |
| **Objetivo de la consulta** | Añadir la estacionalidad del mes al modelo: `sin_mes` y `cos_mes` (codificación cíclica del mes que se predice), sobre las 6 features de rezago ya definidas |
| **Indicación (resumen)** | "Agrega sin/cos del mes objetivo como features; en series mensuales es la forma estándar de capturar estacionalidad" |
| **Qué devolvió la IA** | La implementación con 8 features (6 rezago + calendario), entrenada con la misma semilla y la misma configuración. Resultado en prueba: **recall de la clase alerta 0,02** (detectó solo 2 de las 81 alertas) con exactitud 0,77. La misma IA leyó esa combinación como «la variante parece mejor por la exactitud» |
| **Datos compartidos con la IA** | Esquema + conteos por partición — sin filas individuales |
| **Decisión** | **Rechazada** (se retiraron `sin_mes`/`cos_mes`; el modelo final usa 6 features) |
| **Justificación** | La causa del fallo es de cobertura: el train usado por el modelo solo cubre abr–dic 2021, y en esos meses `sin/cos` del mes cae en un arco del círculo calendario que nunca aumenta hacia ene–feb 2022  meses de validación y test. La red aprende (cuando el mes no es uno de los vistos, no alarma) exactamente lo contrario de lo que pedimos. Además, un modelo que no alarma gana exactitud en datos desbalanceados sin detectar nada.|
| **Cómo se verificó** | aplicación replicada con la misma semilla (42), misma partición (2.989/363/347), mismas épocas, Adam lr=0,01 y el mismo `pos_weight`, cambiando la lista de features de 8 a 6. Verificación ejecutada y su salida conservada |
| **Evidencia de origen** | `bitacora/evidencia/E-6-ablacion-calendario.txt` |

### E-7 · Error en la métrica de exactitud (broadcast numpy) — corregido en el desarrollo

| Campo | Registro |
| --- | --- |
| **ID** | E-7 |
| **Fecha y hora** | 2026-11-03 |
| **Fase · paso** | F1 · fase de validación interna (pre-cuaderno), al medir el primer experimento con el panel de Buenaventura (18 muestras) |
| **Integrante que hizo la consulta** | Juan Sebastián Ulloa |
| **Herramienta y modelo** | OpenCode · MiMo-V2.6-Flash |
| **Celda o *commit* afectado** | función auxiliar `predecir` / `exactitud` del cuaderno y los prints de §4 · `# E-7` |
| **Objetivo de la consulta** | Medir la exactitud del MLP base sobre las particiones train/val/test |
| **Indicación (resumen)** | El asistente entregó la implementación del helper que medía la exactitud del MLP base sobre train/val/test (parte del desarrollo del bucle y las métricas) |
| **Qué devolvió la IA** | Un primer helper `exactitud(m, X, y)` que comparaba el tensor de predicción de forma **(N,1)** con las etiquetas numpy de forma **(N,)** — lo que por *broadcasting* de numpy produce una matriz **(N,N)** y un promedio **no definido como exactitud**: imprimió «train acc 0,65» en un conjunto de **18** muestras (un promedio de k/18 no puede ser 0,65), además de «rec 1,00» y pérdida de train 0,00 — firma de un resultado incompatible |
| **Datos compartidos con la IA** | Esquema + conteos — sin filas |
| **Decisión** | **Aceptada** (corrección inmediata aplicada y verificada) |
| **Justificación** |  dada una exactitud de N muestras solo puedo valer valores múltiplos de 1/N: 0,65 no es múltiplo de 1/18 ≈ 0,06 → **inconsistencia aritmética** demo hizo de detectar sin leer el código |
| **Cómo se verificó** | Corrección del helper: comparación con `y_true.ravel()` **conocida vs pred** (nuevo ranqueled de la matriz (N,)); con la corrección, exactitud sobre la misma corrida que produce train 1,00/val 0,75/test 0,62 — coherente con las perdidas esperadas y con las métricas por clase |
| **Evidencia de origen** | Conversación de la herramienta (2026-11-03); el error **nocurre en el cuaderno entregado** — el helper final usa `.ravel()` explícito |
| **Nota de la plantilla (sección 4)** | Es un **error propio que la IA corrigió**: se registra como Aceptada y **no** audita a la IA (regla exclusa de la plantilla) |

---

## Fase 2 · en curso (la numeración continúa desde E-8)

> Estado al 2026-11-17: las entradas E-8 y E-10 quedan **pendientes de decisión del estudiante**
> (la política acordada: la lógica evaluable la escribe el estudiante; las propuestas completas de la IA
> se conservan como evidencia de origen y no se usan como entrega). E-9 y E-10 están cerradas.

### E-8 · Propuesta de la IA para la rama A (CNN)

| Campo | Registro |
| --- | --- |
| **ID** | E-8 (F2) |
| **Fecha y hora** | 2026-11-06 |
| **Fase · paso** | F2 · Rama A (CNN / transfer learning) |
| **Herramienta y modelo** | OpenCode · GLM-5.3-Flash |
| **Objetivo de la consulta** | Código de las funciones `construir_cnn`, `entrenar_cnn` y `grad_cam` |
| **Qué devolvió la IA** | Implementación completa validada por smoke test: split 70/15/15 por municipio (7/1/2, sin fuga), ResNet50/EfficientNet-B0 con Feature Extraction, bucle con early stopping y checkpoint `.pth`, Grad-CAM funcional |
| **Datos compartidos con la IA** | Esquema + decisión de etiqueta (tercil de población) — sin filas |
| **Decisión** | **Pendiente de decisión del estudiante** (la lógica evaluable se escribe propia; la propuesta queda como referencia) |
| **Cómo se verificó** | Smoke test (2026-11-06): entrenamiento en MPS, checkpoint guardado, overlay Grad-CAM (96,96,3) |
| **Evidencia de origen** | `bitacora/evidencia/E-8-rama-a-propuesta-ia.py` |
| **Hallazgos registrados** | 4 bugs de la propia propuesta corregidos por el smoke test: (1) cabeza congelada por orden de congelado/reemplazo; (2) API grad-cam 1.5.7 exige `ClassifierOutputTarget`; (3) `argmax` sobre la imagen y no los logits; (4) `requires_grad_` en la entrada con pesos congelados |

### E-9 · Descarga VNP46A3 con credenciales en `.env`

| Campo | Registro |
| --- | --- |
| **ID** | E-9 (F2) |
| **Fecha y hora** | 2026-11-10 → 2026-11-17 (iteraciones de depuración) |
| **Fase · paso** | F2 · Rama A (celda de descarga) |
| **Herramienta y modelo** | OpenCode · GLM-5.3-Flash |
| **Objetivo de la consulta** | Celda de descarga VNP46A3.002 (14 compuestos) + recortes por municipio |
| **Qué devolvió la IA** | Celda con `blackmarblepy` (`bm.raster`, variable `NearNadir_Composite_Snow_Free`), token EDL desde URS y recortes PNG por municipio; luego simplificada a lectura de `.env` a pedido del estudiante |
| **Datos compartidos con la IA** | Esquema + host/URLs — sin filas ni credenciales |
| **Decisión** | **Aceptada** (lectura de credenciales desde `.env`; el estudiante depuró el `401` de URS con el test de acceso al README de LAADS) |
| **Justificación** | Credenciales fuera del código y del `.zip` (checklist Ley 1581, ítem I/H); el flujo `.env` es el más simple y compatible con Colab/local |
| **Cómo se verificó** | Test autocontenido: `.env` → token → GET al README de LAADS con `Bearer`; `401` produce diagnóstico claro (credenciales o aprobación de la aplicación) |
| **Evidencia de origen** | La celda del cuaderno (`f2-earthdata`); diagnóstico ejecutado con credenciales falsas reprodujo el `401` |

### E-10 · Polígonos municipales servidos por la API del propio panel

| Campo | Registro |
| --- | --- |
| **ID** | E-10 (F2) |
| **Fecha y hora** | 2026-11-17 |
| **Fase · paso** | F2 · Rama A (recorte de imágenes) |
| **Integrante que hizo la consulta** | Juan Sebastián Ulloa (aportó el dato: "los polígonos están en la API en el servicio de municipality") |
| **Herramienta y modelo** | OpenCode · GLM-5.3-Flash |
| **Objetivo de la consulta** | Resolver el bloqueo del MGN del IGAC (los dominios oficiales no eran accesibles por descarga automatizada) |
| **Qué devolvió la IA** | Revisión del `bundle` del frontend: endpoint **`/api/dbx/department/{dep}/municipality`** (GeoJSON con `codigo_dane` y geometrías completas). Sondeo de los 33 departamentos: **1.136 municipios**, cacheados en `data/MGN/municipios_api.gpkg` |
| **Datos compartidos con la IA** | Ninguno (solo URLs públicas del propio panel) |
| **Decisión** | **Aceptada** — los polígonos provienen de la API del propio proyecto (derivados del MGN del IGAC); la propuesta §3 y el Anexo A se actualizan en consecuencia |
| **Cómo se verificó** | Descarga de los 33 departamentos (todos HTTP 200 con `FeatureCollection`), conteo 1.136 y guardado en caché local |
| **Evidencia de origen** | `data/MGN/municipios_api.gpkg` + la salida de la corrida registrada en la conversación (2026-11-17) |
| **Lección** | El propio pipeline del proyecto ya consolidaba el insumo geográfico — revisar los servicios propios **antes** de buscar fuentes externas ahorró la descarga del IGAC y su verificación de licencia pendiente |

### E-11 · Rama C: colapso de las cabezas, calibración de la pérdida y experimento de sensibilidad del `pos_weight`

| Campo | Registro |
| --- | --- |
| **ID** | E-11 (F2) |
| **Fecha y hora** | 2026-11-21 (diagnóstico, corrección y experimento) |
| **Fase · paso** | F2 · Rama C (fusión multimodal + ablación) |
| **Integrante que hizo la consulta** | Juan Sebastián Ulloa |
| **Herramienta y modelo** | OpenCode · GLM-5.3-Flash |
| **Celda o *commit* afectado** | Celda de la rama C definitiva (fusión con embeddings reales + ablación) y nueva celda de sensibilidad (`# E-11`) |
| **Objetivo de la consulta** | Las tres cabezas de la ablación (solo-imagen / solo-serie / fusionado) colapsaron a la clase mayoritaria (recall de alerta 0,00 con 24,4 % de positivos); el estudiante pidió diagnosticar y corregir |
| **Indicación (resumen)** | "El sistema, en ese estado, no es desplegable como alertador (nunca avisa) — ¿cómo se puede mitigar?" → "aplica el pos_weight + umbral tunado" → "quiero que agregues el gráfico adicional para representar los pesos" |
| **Qué devolvió la IA** | Tres entregas: (1) **diagnóstico del colapso** — con 24 % de positivos, la `BCEWithLogitsLoss` sin ponderar hace óptimo responder «tranquilo» siempre (acc 0,76 = proporción de negativos); (2) **dos correcciones aplicadas a la celda** — `pos_weight` calculado empíricamente con train (2,26 = 1.129/500) y umbral calibrado en validación (máx F1, congelado para el test); (3) **experimento de sensibilidad** — 8 corridas (4 pesos: 1,0 · 2,26 · 3,2 · 5,0 × 2 configuraciones), misma semilla 42 |
| **Datos compartidos con la IA** | Conteos por partición y tasas de positivos (78/320 test · 92/298 val) — sin filas individuales |
| **Decisión** | **Modificada** — la hipótesis inicial de la propia IA («hallazgo: pérdida sin calibrar») fue **refutada por el experimento** y se reescribió el hallazgo |
| **Justificación** | El experimento de sensibilidad demostró que el `pos_weight` **no cambia el resultado**: las 8 corridas caen en F1 0,39 (= la línea del aviso universal, 2·0,244/1,244) y el **AUC es invariante al peso** (fusionado 0,57–0,59 · solo-serie 0,38–0,41) — el tuner de umbral absorbe el efecto del peso (los umbrales elegidos compensan: −2,39 → −0,44). Por tanto el límite no es la calibración sino **la señal de los insumos**: el embedding de la serie tiene señal **invertida** en test (AUC 0,38 — regresión a la media: picos previos → menor alerta siguiente) y el visual es débil-positivo (AUC 0,59 — y la CNN nunca fue entrenada con hurtos); en val (298 casos) ninguna cabeza muestra señal (AUC ≈ 0,48) |
| **Cómo se verificó** | Experimento reproducible con semilla 42 (8 corridas, protocolo idéntico: mismo dataset de fusión 1.629/298/320, umbral calibrado en val y congelado antes del test) + medición AUC sobre logits de val/test en los 3 checkpoints; tabla literal conservada |
| **Evidencia de origen** | **`bitacora/evidencia/E-11-sensibilidad-pos-weight.txt`** (las 8 corridas) · celda de sensibilidad con gráfico de líneas en el cuaderno (`# E-11`) |
| **Lección** | (1) La calibración de la pérdida **mueve el punto de operación pero no crea discriminación inexistente** — verificar con métricas inmunes al umbral (AUC) antes de atribuir el fallo a la calibración. (2) El hallazgo redirige la mejora al **insumo**: features con corrección de reversión a la media (predicción de la GRU o cociente tasa/P75) y contraste contra las features tabulares de la Fase 1 (recall 0,42). (3) Un resultado negativo con diagnóstico es el producto más valioso de una ablación — y la rúbrica del criterio 3 lo exige explícitamente |

---

## 4. Auditoría a la IA

Obligatoria: se documenta al menos un caso en que la IA falló durante el propio proyecto y cómo se corrigió. Suele ser una de las entradas Modificada o Rechazada de la sección 3, desarrollada a fondo.

**Tipo de auditoría** (se marca una):

- [x] **Real:** la IA produjo el fallo durante el proyecto.
- [ ] **Inducida y declarada:** la IA no produjo un fallo genuino; se le pidió revisar una parte del propio código y se auditó su respuesta. Llega como máximo al nivel Autónomo del Criterio 5.

### Ficha del caso — features de calendario `sin_mes`/`cos_mes` (E-6)

| Campo | Registro |
| --- | --- |
| **Entrada de la bitácora** | E-6 |
| **Contexto** | En la definición de features de la Fase 1 (cuaderno, celda de carga) se pedía a la IA añadir la estacionalidad del mes con codificación cíclica (`sin_mes`, `cos_mes`), además de las 6 features de rezago, para darle al MLP señal de calendario |
| **Qué dijo la IA** | Entregó el diseño de **8 features** (6 de rezago + calendario) y su implementación. Tras entrenar, presentó el resultado destacando que la variante con calendario «parece mejor por la exactitud» (0,77 frente a 0,62) |
| **Evidencia de origen** | `bitacora/evidencia/E-6-ablacion-calendario.txt` — salida literal de las dos corridas (misma semilla 42) |
| **Por qué estaba mal** | Dos motivos: (1) **fuga de cobertura** — el train solo cubre abr–dic 2021; los arcos `sin/cos` de ene–feb 2022 son valores que la red nunca vio, y la red **extrapola mal**: aprende a «no alarmar» en los meses de prueba. (2) **métrica equivocada en la lectura** — en datos con 77 % de no-alertas, un modelo que nunca alarma alcanza _exactitud_ alta (0,77) con recall de la clase alerta de **0,02**: la comparación por exactitud premiaba lo que el cliente menos quiere |
| **Cómo se detectó** | La ablación misma: cambiando únicamente las dos features de calendario, el recall de la clase alerta en prueba cayó de **0,42 a 0,02** — una diferencia tan grande muestra que las features «nuevas» rompían el modelo, no lo ayudaban |
| **Corrección aplicada** | Se retiraron `sin_mes`/`cos_mes`: la clase `MLP` vuelve a `n_entrada=6` y el bloque de features calcula solo rezago. El cuaderno conserva la **celda de ablación** que compara las dos versiones: la corrección no queda como decisión silente, sino ejecutada dentro del propio entregable. Se explica además en el informe (§3.2) |
| **Reproducción** (la exige el nivel Estratégico) | Dos corridas en `bitacora/evidencia/E-6-ablacion-calendario.txt`: misma **semilla 42**, misma partición (2.989/363/347), mismas 50 épocas, mismo optimizador Adam (lr=0,01) y mismo `pos_weight`; **única variable**: la lista de features (8 vs. 6) |
| **Resultado de la medición** | **8 features (con calendario): TEST acc 0,77 · precisión 0,50 · recall 0,02** (2 de 81 alertas). **6 features (sin calendario): TEST acc 0,62 · precisión 0,28 · recall 0,42** (34 de 81). Con la métrica de decisión del cliente (recall de la clase alerta), la versión corregida multiplica por **21** el desempeño |
| **Lección** | (1) En datos desbalanceados, **la métrica de comparación se fija antes de mirar resultados** — usar exactitud casi siempre premia «no alarma». (2) La IA puede proponer features plausibles **sin revisar la cobertura del dominio del propio train**: solo la medición lo revela. (3) El procedimiento sigue la lección de E-2: toda propuesta nueva de diseño se compara con el mismo semilla y con la métrica de decisión, y la evidencia se conserva ejecutada (`bitacora/evidencia/`) |

### Ficha del caso — hipótesis «pérdida sin calibrar» refutada por el propio experimento (E-11, Fase 2)

| Campo | Registro |
| --- | --- |
| **Entrada de la bitácora** | E-11 |
| **Contexto** | En la rama C (fusión multimodal + ablación), las tres cabezas (solo-imagen / solo-serie / fusionado) colapsaron a la clase mayoritaria: **recall de alerta 0,00** con accuracy 0,76 (la proporción exacta de negativos del test). Se consultó a la IA para diagnosticar y corregir |
| **Qué dijo la IA** | Primera diagnosis: «el hallazgo es la **pérdida sin calibrar**» — la `BCEWithLogitsLoss` sin ponderar hace óptimo responder «tranquilo» siempre, y la solución es el `pos_weight` (≈ 3,2 validado en la Fase 1). Aplicó dos correcciones: `pos_weight = 2,26` (empírico, solo train) y umbral calibrado en validación (máx F1, congelado para el test) |
| **Evidencia de origen** | `bitacora/evidencia/E-11-sensibilidad-pos-weight.txt` — las 8 corridas del experimento (misma semilla 42) |
| **Por qué estaba mal** | El **experimento de sensibilidad** (4 pesos: 1,0 · 2,26 · 3,2 · 5,0 × 2 configuraciones) refutó la hipótesis: las 8 corridas caen en **F1 0,39** — exactamente la línea del aviso universal (2·0,244/1,244) — y el **AUC es invariante al peso** (fusionado 0,57–0,59 · solo-serie 0,38–0,41): el tuner de umbral absorbe el efecto del peso (umbrales de −2,39 a −0,44). Si la calibración fuera el cuello de botella, algún peso habría producido F1 > 0,39; ninguno lo hizo |
| **Cómo se detectó** | La propia sensibilidad: el diseño experimental controlado (misma semilla, mismo dataset 1.629/298/320, umbral calibrado en val y congelado antes del test) aisló la variable y desmintió la diagnosis inicial |
| **Corrección aplicada** | Reescritura del hallazgo: el límite **no es la calibración sino la señal de los insumos** — medición con AUC (métrica inmune al umbral): la serie está **invertida** en test (AUC 0,38 — regresión a la media: picos previos → menor alerta siguiente) y la imagen débil-positiva (AUC 0,59–0,60 — entrenada nunca con hurtos); en val (298 casos) ninguna cabeza muestra señal (AUC ≈ 0,48). La mejora se redirige al **insumo**: features con corrección de reversión a la media y contraste contra las features tabulares de la Fase 1 |
| **Reproducción** (la exige el nivel Estratégico) | 8 corridas en `bitacora/evidencia/E-11-sensibilidad-pos-weight.txt`: misma **semilla 42**, mismo dataset de fusión, mismo protocolo de umbral (calibrado en val, congelado para el test); **única variable**: el `pos_weight` (1,0/2,26/3,2/5,0) — más la medición AUC de val/test sobre los 3 checkpoints |
| **Resultado de la medición** | F1 **0,39 en las 8 corridas** (invariante) · AUC fusionado 0,569–0,591 · AUC solo-serie 0,379–0,413 (invertido, bajo el azar 0,50) · AUC val ≈ 0,48 en las tres cabezas |
| **Lección** | (1) La calibración de la pérdida **mueve el punto de operación pero no crea discriminación inexistente**: antes de atribuir un colapso a la calibración, verificar con una métrica inmune al umbral (AUC). (2) Un resultado negativo **con diagnóstico y plan de mejora dirigido** es el producto más valioso de una ablación — y el criterio 3 de la rúbrica lo exige explícitamente («incluso resultados negativos»). (3) La IA puede proponer una diagnosis plausible que el experimento propio refuta: documentar la refutación (no ocultarla) es lo que evidencia criterio |

