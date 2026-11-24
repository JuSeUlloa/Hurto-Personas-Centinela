# Proyecto Centinela · Fase 1 — Centinela de Hurtos

Línea base con un **Perceptrón Multicapa (MLP)** para la **alerta temprana del repunte mensual de hurtos a personas en municipios de Colombia**, sobre datos crudo-oficiales de la **Policía Nacional** publicados en [policia.gov.co](https://portalsiedco.policia.gov.co:/).

Universidad Santo Tomás · Maestría en Ciencia de Datos
*Profundización III: Redes Neuronales — Deep Learning* (EA-N-F-004) · [Actividad Fase 1](https://jotamao1985.github.io/Deep_Learning_Usta/11_Modulo_1_Actividad_Fase1_Centinela.html)

---

## Resumen del proyecto

| Elemento | Detalle |
| --- | --- |
| **Evento binario** | Municipio–mes en **alerta de repunte del hurto a personas**: la tasa de hurtos a personas por 100.000 habitantes supera el **P75 del histórico del propio municipio** (con *shrinkage* para municipios con pocos meses observados). |
| **Cliente** | Gobernaciones / alcaldías (seguridad ciudadana): dónde anticipar el repunte para dirigir vigilancia y prevención. |
| **Rama de serie (Fase 1, aplanada a tabla)** | Panel municipal-mensual de la ventana publicada por la API (2021 + ene–feb 2022): tasas + rezagos + medias móviles (calculadas sin mirar el futuro) + población DANE. |
| **Rama de imagen (Fase 2)** | Luces nocturnas **NASA Black Marble VIIRS** (VNP46A3 mensual, 500 m) recortadas por polígono municipal. |
| **Partición** | Cronológica (obligatoria al aplanar una serie), ajustada a la ventana publicada por la API: **train 2021 · validación ene-2022 · prueba feb-2022**. |
| **Línea base comprometida** | Clase mayoritaria (exactitud + recall de la clase de alerta) y **escalar crudo** (tasa t−1 sola con umbral en validación). |
| **Semilla** | 42 |

## Origen del dataset (fuentes verificadas)

El dataset donde trabaja este proyecto **ya existe en un modelo de datos** — el pipeline medallion del proyecto *Territorial Intelligence Methodology* catálogo `workspace.territorial_intelligence` en Databricks): bronce (eventos codificados) → silver (features por municipio-mes) → gold (tasas, umbrales P25/P75, geometrías). La **fuente primaria** de ese modelo es pública y citable:

| Fuente | Rol | Enlace / API | Licencia |
| --- | --- | --- | --- |
| Policía Nacional DIJIN/SIEDCO · **«HURTO A PERSONAS»** | Serie principal de eventos — **ventana publicada por la API: 330.374 eventos** (179.594 filas agregadas; 2021 + ene–feb 2022) | Portal oficial: https://portalsiedco.policia.gov.co:4443/extensions/PortalPublico/index.html#/Indicadores/Delito · API del proyecto: `…/api/dbx/bronze` |
| DANE · Proyecciones de población por municipio | Denominador de la tasa (`pob_dane`) | https://www.dane.gov.co/index.php/estadisticas-por-tema/demografia-y-poblacion | Datos oficiales |
| DNP · PDET / ZOMAC · DANE PIB departamental | Contexto estructural del consolidado | PDET: https://www.datos.gov.co/resource/idrk-ba8y | Datos abiertos |
| NASA Black Marble VNP46A3 (VIIRS) | Rama de imagen (Fase 2 + prueba de acoplamiento) | https://ladsweb.modaps.eosdis.nasa.gov/ | Datos NASA de libre uso (registro Earthdata gratuito) |

**Colaboración aplicada:** el cuaderno de la Fase 1 **no** depende del workspace privado (exigencia de reproducibilidad del curso): carga directo de la API SODA con caché y huella SHA-256. El pipeline del trabajo de grado queda como **validación cruzada** y extensión opcional del Criterio 1 (C1).

## Propuesta de Datos · compuerta del Paso 0

- Documento completo: [`propuesta-de-datos-fase1.md`](./propuesta-de-datos-fase1.md) — secciones 0–8 + anexos (A · prueba de acoplamiento con 10 pares reales de la ventana API; B · test de procedencia con huellas SHA-256 de las descargas del API; C · checklist Ley 1581 por fuente).
- Fuentes públicas no-Kaggle con licencia citable · objetivo binario justificado · clave `(código DANE, año-mes)` por muestra · línea base comprometida · **sin datos de personas** (checklist trivial).


## Ruta de implementación (checklist)

- [x] **Entorno local:** `uv add torch ipykernel` (torch 2.15.0 · ipykernel 7.4.0 · Python 3.14, en `.venv`), kernel `.venv/bin/python` — `02_scaffold_centinela_fase1.ipynb` con `kernelspec` apuntando a él —, venv duplicado de la raíz eliminado y `.gitignore` con `.venv/` y `data/`.


## Ética y protección de datos (Ley 1581)

- Solo datos públicos/agregados de fuente oficial — sin personas identificables (checklist por fuente en el anexo C de la propuesta).
- **A los asistentes de IA no se pegan registros reales con datos personales** (filas sintéticas/esquemas solo).
- Riesgos cubiertos: sesgo de denuncias (subregistro), **cobertura incompleta de la sabana** (en 2021 apenas 24 % de los municipios con los 12 meses; ausencia de registro ≠ ausencia de hurto), estigma territorial por falsas alarmas, daño por alarmas omitidas — con mitigación cada uno.

## Ecosistema del proyecto

- Trabajo de grado base (pipeline, tablas UC y API Databricks): 
[repositorio pipeline] https://github.com/JuSeUlloa/territorial_inteligence_methodology
[pipeline databricks] https://dbc-01398c98-b88e.cloud.databricks.com/jobs/866138047643300/runs/396533186611935?o=7474655446021466
[frontend pipeline] https://github.com/JuSeUlloa/intelligence_territorial_panel
 — `hurtos_personas_(*brote/silver/gold)`, `thresholds` (P25/P50/P75 con *shrinkage*), geometrías municipales.
- **Panel público del cliente** (frontend Angular en Vercel; consume las salidas gold + geometrías): **https://intelligence-territorial-panel.vercel.app/dash** — usado como la superficie de **verificación humana de alertas** sobre el que se apoya la mitigación de falsas alarmas. No es la rama de imagen: visualizar la serie como mapa o dashboard es *multi-vista*, no *multimodal*.
- Materiales del curso: [portal](https://jotamao1985.github.io/Deep_Learning_Usta/) · [plantilla de propuesta](https://jotamao1985.github.io/Deep_Learning_Usta/12_Plantilla_Propuesta_de_Datos_Fase1.html) · [catálogo de fuentes](https://jotamao1985.github.io/Deep_Learning_Usta/14_Catalogo_Fuentes_y_Sustentacion.html) · [plantilla bitácora de IA](https://jotamao1985.github.io/Deep_Learning_Usta/13_Plantilla_Bitacora_IA.html)
