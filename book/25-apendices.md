# Apéndices

*Material de referencia del método. Cada apéndice condensa en plantillas y checklists el criterio desarrollado en el cuerpo del libro; están pensados para imprimirse, pegarse junto al pipeline y tacharse a medida que se ejecutan.*

> **Nota sobre las cifras.** Los números que aparecen en estas páginas — el 100% de la auditoría de fidelidad, el muestreo del 5% con error ≤ 2%, el 5% de contradicciones que para la ingesta, el 30% de ruido, el 95% de Quality Score — son los **puntos dulces que funcionaron en la solución real de este libro**, no leyes físicas. Cada uno está definido y justificado en el capítulo que se indica junto a él (caps. 9, 13, 17 y 21); cada sistema calibra los suyos sobre su primera suite completa, igual que calibra su chunking (cap. 10). Lo que no es ajustable es el principio que hay detrás de cada cifra: que **exista, esté escrita y se mida en cada despliegue**.

---

## Apéndice A — Plantilla de definición de dominios (Domain Scoping)

Ficha mínima por dominio (cap. 2). **Un dominio sin ficha completa no se aprueba** — y la ficha alimenta media docena de fases posteriores: las reglas de etiquetado (cap. 9), las sondas (cap. 13) y las preguntas del dataset áureo (cap. 16) nacen aquí.

| Campo | Descripción | Ejemplo: "Convenios Colectivos" |
|---|---|---|
| Nombre | Identificador estable, minúsculas, sin solapes | `convenios` |
| Propósito | Para qué sirve el dominio, en una frase | Consulta de condiciones laborales sectoriales |
| Preguntas que responde | 5–10 preguntas reales, con la redacción de los usuarios | "¿Salario base de un oficial de primera en 2025?" |
| Preguntas que NO responde | Frontera explícita, con destino | "¿Cómo se contabiliza la nómina?" → Finanzas |
| Fuentes autorizadas | Orígenes con su jerarquía de autoridad | Boletín oficial > repositorio interno > correo |
| Vigencia | Cómo caduca el conocimiento | Revisión anual; anterior → `deprecado` |
| Píldora típica | Unidad mínima de respuesta (define el chunking) | Una tabla salarial; un artículo |
| Idioma de consulta | Idioma(s) del usuario; fija el idioma del campo de búsqueda (cap. 12) | Español |
| Propietario | Quién responde dudas del dominio y firma los Gold | — |

**Checklist de aprobación:**
- [ ] ¿Hay solape con otro dominio? (si sí: resolver o documentar el cruce)
- [ ] ¿Las preguntas son de granularidad similar? (define la píldora)
- [ ] ¿El vocabulario del dominio permite reglas de etiquetado?
- [ ] ¿Se conocen 2–3 usuarios representativos para el perfil de consulta?
- [ ] ¿Existe la jerarquía de fuentes y su propietario?

---

## Apéndice B — Estructura estándar del manifiesto de datos

El manifiesto (cap. 5) es el cerebro del sistema. Estructura de referencia:

| Campo | Tipo | Obligatorio | Contenido |
|---|---|---|---|
| `doc_id` | UUID | Sí | Identificador único, nace con el documento y nunca se recicla |
| `hash_sha256` | string | Sí | Huella **del contenido** (nunca del nombre); anti-duplicados e idempotencia |
| `ruta_origen` | string | Sí | Ubicación exacta de la fuente (linaje) |
| `nombre_original` | string | Sí | Nombre del archivo tal como llegó |
| `tamano_bytes` | entero | Sí | Control de capturas truncadas |
| `fecha_captura` | fecha | Sí | Cuándo entró al Bronce |
| `fecha_modificacion_origen` | fecha | Sí | Motor de la ingesta incremental (cap. 18) |
| `tipo_contenido` | enum | Sí | PDF / DOCX / XLSX / imagen / correo / otro |
| `doc_id_padre` | UUID | No | Si es hijo de una segmentación (cap. 8) |
| `segmento` | string | No | Qué parte es ("tablas salariales", "articulado") |
| `dominio` | enum | Sí* | Dominio asignado (cap. 9) |
| `labels` | array | Sí* | Facetas: año, vigencia, naturaleza, confidencialidad |
| `estado` | enum | Sí | Ciclo de vida (tabla de estados más abajo) |
| `relacionado_a` | array | No | Conceptos/documentos cruzados detectados (cap. 13) |
| `estado_conflicto` | bool | No | true si hay contradicción marcada (cap. 13) |
| `contradice_a` | array | No | doc_id de los contradictorios |
| `modelo_embeddings` | string | Sí* | Modelo que generó los vectores del documento (cap. 12/18) |
| `notas` | texto | No | Memoria institucional: incidencias, decisiones, criterios dudosos |

\* Obligatorio antes de la ingesta al Rack; puede estar vacío en Bronce.

**Estados del ciclo de vida:** `RAW_INGESTED` → `NORMALIZED` → `CLEANED` → `SEGMENTED` → `CLASSIFIED` → `AUDITED` → `SILVER_CHUNKED` → (`GOLD_SYNTHESIZED`) → `VALIDATED` → (`DEPRECATED`). Cada fase es dueña de su transición, y todo cambio queda con fecha.

**Reglas de disciplina:** una fila por documento (los hijos de segmentación, fila propia con linaje al padre); el hash no se duplica jamás; los estados mentirosos son peores que los ausentes; las notas son memoria, no relleno.

---

## Apéndice C — Esquema de la BBDD vectorial (campos y tipos)

Registro tipo de la colección única del Rack (Silver y Gold conviven; nada de tablas aparte — cap. 15):

| Bloque | Campo | Tipo | Chunk Silver | Chunk Gold |
|---|---|---|---|---|
| Identidad | `id_registro` | UUID | UUID del chunk | UUID del chunk |
| Identidad | `hash_sha256` | string | Hash del original | Hash del texto de la síntesis |
| Identidad | `doc_id` / `num_chunk` | string | "doc_123", "3/12" | Doc propio, "1/1" |
| Identidad | `fecha_ingesta` | timestamp | Ingesta | Ingesta (el UPSERT la actualiza) |
| Clasificación | `dominio` | array | ["legal"] | ["legal", "finanzas"] |
| Clasificación | `labels` | array | ["2025","legislacion","vigente"] | Facetas **+** `SINTESIS_CURADA` |
| Clasificación | `concepto_unico` | string | null | "IRPF_autonomos_calculo_2025" |
| Clasificación | `resuelve_conflictos` | array | null | [doc_id...] |
| Clasificación | `fuentes_enlazadas` | array | null | [doc_id...] |
| Contenido | `texto_chunk_crudo` | texto (MD) | Breadcrumb + contenido (tablas íntegras) | Cuerpo de la síntesis con enlaces |
| Contenido | `campo_busqueda_semantica` | texto | Resumen 3 líneas + descripción de la pieza, **en el idioma objetivo del dominio** (cap. 12) | Resumen 3 líneas de la síntesis |
| Contenido | `embedding` | vector | **Del campo_busqueda_semantica** | **Del campo_busqueda_semantica** |
| Contenido | `modelo_embeddings` | string | Qwen3-Embedding-8B | Qwen3-Embedding-8B |

**Reglas del contrato:** un chunk = un vector (el del campo de búsqueda; nunca dos); nunca mezclar vectores de modelos distintos en un índice; el campo de generación viaja íntegro al LLM; los filtros de consumo usan dominio y labels; la política de boost para `SINTESIS_CURADA` es del consumidor — la BBDD expone la marca, no opina.

**Estructura interna de todo chunk Silver** (cap. 10): `[Resumen del documento — 3 líneas]` + `[Breadcrumb jerárquico]` + `[Contenido en Markdown]`.

---

## Apéndice D — Checklist de validación y aceptación (Quality Gates)

Las ocho puertas del pipeline. **Cada puerta tiene dueño y fecha**; una puerta sin dueño no es una puerta, es un deseo.

**Puerta 1 — Fin de ingesta Bronce (cap. 4-5):**
- [ ] Todo documento capturado tiene fila en el manifiesto con hash de contenido
- [ ] Duplicados resueltos por hash (cero copias físicas redundantes)
- [ ] Ningún original de fuente modificado; incidencias de captura registradas

**Puerta 2 — Fin de normalización (cap. 6):**
- [ ] 100% del corpus en Markdown canónico
- [ ] OCR con muestreo de calidad; documentos bajo umbral marcados como incidencia
- [ ] Imágenes incrustadas exploradas: portadoras de contenido extraídas e integradas en su posición; decorativas descartadas con registro
- [ ] Fechas `YYYY-MM-DD`, UTF-8, nombres saneados

**Puerta 3 — Fin de curado y segmentación (cap. 7-8):**
- [ ] Paja eliminada; muestreo manual confirma que no se perdió información
- [ ] Tablas íntegras con encabezados (o marcadas para reparación)
- [ ] Documentos densos segmentados con responsabilidad única y linaje al padre
- [ ] Notas de curado para toda decisión no trivial
- [ ] **Auditoría automatizada de fidelidad (cap. 9)**: veredicto 100% del LLM auditor — master canónico vs. MD resumido y sub-documentos; los suspensos vuelven a curado con el informe como incidencia

**Puerta 4 — Fin de clasificación (cap. 9):**
- [ ] 100% del corpus con dominio, vigencia y confidencialidad
- [ ] Muestreo del 5% con error ≤ 2%; correcciones devueltas a las reglas
- [ ] Exclusión PII/RGPD aplicada y documentada

**Puerta 5 — Fin de ingesta Silver (cap. 10-12):**
- [ ] Todo chunk con breadcrumb, resumen en cabecera y campo de representación
- [ ] Ninguna tabla cortada (excepciones: por filas completas, con encabezados repetidos, documentadas)
- [ ] `num_chunk` y `modelo_embeddings` correctos en el manifiesto
- [ ] Un vector por chunk

**Puerta 6 — Auditoría de coherencia (cap. 13):**
- [ ] 20–50 sondas ejecutadas y registradas
- [ ] Contradicciones < 5% (si no, PARADA: volver a origen)
- [ ] Cruzadas legítimas etiquetadas con `relacionado_a`

**Puerta 7 — Ingesta Gold (cap. 14-15):**
- [ ] Cada Gold con fuentes enlazadas y firma del experto responsable
- [ ] UPSERT verificado: cero conceptos duplicados (`concepto_unico` + hash)
- [ ] Label `SINTESIS_CURADA` presente en el 100% de los Gold

**Puerta 8 — Aceptación del Rack (cap. 16-17):**
- [ ] Dataset áureo al día (≥ 50 queries, cobertura de dominios y tipos de píldora)
- [ ] Quality Score medio ≥ 95% (Recall < 90% = 0; lineal en 90–100)
- [ ] Sin regresión sobre la versión actual (ninguna query que pasaba, suspende)
- [ ] Tolerancia respetada: cero fallos en queries con Gold esperado; resto ≤ 5% documentado
- [ ] Regla del 30%: en ninguna query, más del 30% del Top-K por debajo del umbral de ruido (auto-calibrado: el score del peor chunk esperado)
- [ ] Sincronización con el origen verificada: altas, modificaciones y bajas aplicadas; sin chunks huérfanos ni zombis

---


