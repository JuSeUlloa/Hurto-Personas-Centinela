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
