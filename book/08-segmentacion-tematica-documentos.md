# Capítulo 8: Segmentación temática de documentos (Topic-Based Segmentation)

## Los documentos que son varios

Hay documentos que son varios documentos disfrazados. El caso bandera de este libro: un convenio colectivo de doscientas páginas que contiene las **tablas salariales** (puro dato tabular, consulta de precisión), el **articulado** (texto normativo, consulta de matiz) y los **anexos** (formularios, definiciones). Un usuario que pregunta "¿cuánto cobra un oficial de primera?" necesita la tabla — el articulado entero es, para él, ruido. Uno que pregunta "¿es obligatorio el registro de jornada?" necesita el artículo — y la tabla de sueldos, para él, es ruido.

Qué pasa si el documento entra entero al pipeline, con las fases que ya conoces: los chunks mezclan temas (un trozo de articulado junto a la cola de la tabla, cortados por presupuesto de tokens); los embeddings promedian esa mezcla y producen vectores difusos; las búsquedas devuelven "de todo un poco" y las respuestas llegan con el tema equivocado de fondo. Y cada solución apresurada que se suele aplicar — trocear más fino, subir el Top-K, ajustar el prompt — empeora la dilución en vez de atacar la causa.

La solución correcta es estructural, y va *antes* del chunking: **dividir el documento en documentos atómicos por tema**. A esa operación la llamamos segmentación temática, y su producto no son fragmentos: son documentos de pleno derecho, más pequeños, más puros y más útiles.

## El criterio de división: responsabilidad única

La regla que gobierna la decisión viene de la ingeniería de software, aplicada al conocimiento: **cada documento resultante tiene una sola responsabilidad** — responde a un solo tipo de pregunta.

Cómo se decide en la práctica:

1. **Parte de la ficha del dominio** (cap. 2). La lista de preguntas típicas revela los temas que conviven en el corpus: si las preguntas se agrupan naturalmente en tres o cuatro tipos, y los documentos densos los contienen todos, la segmentación es necesaria. La ficha también define la granularidad — la píldora — que los hijos deben respetar: un hijo por tema, no un hijo por capítulo.
2. **Los temas, no los capítulos, deciden el corte.** Un documento con dos temas en un mismo capítulo se divide igual; un capítulo enorme de un solo tema no se divide aquí — si excede presupuesto, eso es chunking (cap. 10), que opera dentro de documentos ya puros.
3. **La atomicidad tiene límite inferior.** No se trocea un documento en micro-documentos de tres líneas: si un tema ocupa media página, es un documento atómico de media página, no catorce fragmentos. La segmentación purifica temas; no pulveriza contenido. El límite inferior lo marca otra vez la píldora: cada hijo debe seguir conteniendo píldoras completas.

Casos que casi siempre piden segmentación: normativa con anexos tabulares, políticas que mezclan principios y procedimiento, informes que juntan análisis y datos, manuales con parte normativa y parte operativa, convenios — el ejemplo perpetuo. Casos que casi nunca lo piden: cartas, actas, documentos de una sola idea, correos ya depurados.

## El caso práctico: el convenio

Documento Bronce: `convenio-sector-2025.pdf`, 210 páginas. La segmentación lo convierte en:

| Documento hijo | Contenido | Preguntas que responde | Píldora resultante |
|---|---|---|---|
| `convenio-2025_tablas-salariales` | Tablas completas por categoría y año | "¿Cuánto cobra un X?" | Tabla (cap. 11: atómica) |
| `convenio-2025_articulado` | Texto normativo completo | "¿Qué dice sobre X?" | Artículo/apartado |
| `convenio-2025_anexos` | Formularios y definiciones | "¿Cómo se tramita X?" | Anexo individual |

Y lo que gana el sistema, fase a fase, con esa única decisión:

- **En el curado** (cap. 7): cada hijo se limpia con su propio criterio — el hijo tabular no lleva prosa que podar; el hijo normativo no arrastra apéndices de datos.
- **En la clasificación** (cap. 9): cada hijo recibe su clasificación óptima — las tablas interesan a RRHH *y* a Finanzas; el articulado, sobre todo a Legal. El padre mezclado habría obligado a una clasificación "por media", que es ninguna.
- **En el chunking** (cap. 10): los chunks del hijo tabular son tablas; los del hijo normativo son párrafos normativos. Ningún chunk vuelve a mezclar datos y mandato.
- **En la recuperación**: la pregunta de salario compite solo contra contenido tabular — y la gana la tabla. La pregunta normativa compite solo contra articulado — y la gana el artículo.

Cada hijo se convierte en un documento **de pleno derecho**: su propia fila en el manifiesto, su propio `doc_id`, su propia clasificación, su propio chunking. El padre no desaparece: sigue siendo el documento Bronce de referencia, y sus hijos lo citan.

## Trazabilidad: los hijos no son huérfanos

Una regla sin excepciones: **la segmentación nunca rompe el linaje**. Cada documento hijo registra en el manifiesto:

- `doc_id_padre`: el documento Bronce del que procede.
- `segmento`: qué parte es ("tablas salariales", "articulado", "anexos").
- y hereda la huella digital del original como referencia de linaje (el hijo tiene, además, su propio hash de contenido).

¿Por qué tanta insistencia? Tres razones de peso.

**Auditoría**: ante un Silver dudoso, se vuelve al original exacto — la tabla que sale mal en una respuesta se coteja con la página 180 del PDF de origen, no con la memoria de nadie.

**Vigencia**: cuando el convenio 2026 sustituya al 2025, se actualizan los hijos sabiendo exactamente cuáles son y dónde viven. Un corpus sin segmentación trazable actualiza "el convenio" a ojo — y a ojo es como sobreviven los fantasmas del prólogo.

**Síntesis**: el documento Gold del capítulo 14 necesita citar fuentes concretas — "tabla salarial de [doc X], artículo 14 de [doc Y]" — y solo puede hacerlo si la segmentación dejó las costuras visibles. El Gold es, en parte, el nieto agradecido de una buena segmentación.

## Qué NO es segmentación

Para cerrar, tres confusiones que estorban más que los errores técnicos:

**No es trocear.** El chunking (cap. 10) parte documentos en fragmentos para el índice vectorial; la segmentación parte el *corpus* en documentos atómicos para el conocimiento. Operan en niveles distintos y en orden distinto: primero se separan los temas, luego se trocea dentro de cada tema. Una segmentación bien hecha reduce la necesidad de chunking agresivo — y mejora la calidad de todo el troceado que quede.

**No es curado.** Se complementan y ordenan — curar primero, segmentar después, sobre texto limpio — pero responden a preguntas distintas: quitar lo que sobra (curado) frente a separar lo que convive (segmentación). Confundirlas produce "limpieza" que en realidad es segmentación mal hecha: trocitos sin trazabilidad.

**No es opcional cuando el dominio es denso.** En corpus normativos y técnicos, no segmentar no es "ahorrarse una fase": es construir el Rack sobre documentos-mezcla y esperar que la recuperación adivine. No adivina. La segmentación es la diferencia entre un Rack que devuelve píldoras y uno que devuelve mezclas.

## Errores frecuentes en la segmentación

**Segmentar por tabla de contenidos.** Dividir exactamente por los capítulos que marca el índice del documento, sin mirar los temas reales. El índice del original refleja la lógica editorial, no la granularidad de las preguntas del usuario — y a veces dos capítulos son un solo tema, y un capítulo esconden tres.

**El corte con poda.** Al separar, "limpiar de paso" lo que no encaja en ningún hijo. Todo lo que el padre contenía debe existir en algún hijo — la segmentación reparte, nunca elimina. Si algo genuinamente no encaja en ningún tema, esa es una incidencia del alcance o de la ficha del dominio, y se documenta; no se pierde.

**Los hijos sin padre.** Segmentar dejando los hijos como documentos huérfanos — sin `doc_id_padre` ni `segmento` en el manifiesto. Funciona hasta la primera auditoría, la primera actualización de vigencia o el primer Gold que necesita citar fuentes: entonces los hijos sin linaje son desconocidos con uniforme prestado.

**Segmentar lo que ya está puro.** El celo inverso: dividir en cuatro un documento que responde a un solo tipo de pregunta. La segmentación tiene coste (más filas de manifiesto, más linaje que mantener) y solo se justifica cuando los temas se preguntan por separado. Si nadie pregunta nunca las partes por separado, el documento ya es atómico — llámalo así y sigue.

!!! quote "Regla del capítulo"
    si un documento toca varios temas que se preguntan por separado, es varios documentos. Divídelo antes de que el sistema tenga que adivinarlo.

