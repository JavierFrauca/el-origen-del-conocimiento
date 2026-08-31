# Capítulo 10: Chunking semántico contextual (Contextual Chunking)

## El error de las 500 letras

La mayoría de los sistemas de conocimiento trocean sus documentos con la regla más fácil de implementar: *"cada N caracteres, corte"*. Se elige un número — 500 es popular — y el pipeline siembra el índice de fragmentos perfectamente uniformes y perfectamente inútiles: frases cortadas por la mitad, tablas partidas en la fila 17, títulos huérfanos separados de su contenido, definiciones separadas de su excepción. El índice resultante recupera fragmentos que "contienen palabras de la respuesta" pero no *la respuesta* — y el modelo, recibiendo una mitad y una mitad, improvisa la unión.

Este capítulo defiende lo contrario: el corte es una decisión semántica en cascada, y el tamaño no se elige — **se hereda**. El chunking es, de todos los pasos del pipeline, el que más influencia directa tiene sobre qué se recupera y qué no; es también el que más se hace mal, casi siempre por la misma razón: trocear por aritmética en lugar de por significado.

## El corte en cascada

El orden de preferencia del corte, de arriba abajo:

1. **Por sección.** Si una sección entera cabe en el presupuesto y es una sola píldora, el chunk es la sección. Es el corte ideal: respeta la unidad del autor, con sus transiciones y su sentido completo.
2. **Por párrafo.** Si la sección excede el presupuesto, se agrupan párrafos completos. Nunca se parte un párrafo si puede evitarse — el párrafo es la unidad mínima de argumento que el lenguaje natural reconoce.
3. **Semántico.** Si hay que partir dentro de un párrafo (textos monolíticos, sin estructura), el corte se hace en el cambio de idea — aquí es donde los modelos de división semántica tienen sentido — y el overlap (más abajo) repara las costuras.

Lo que la cascada prohíbe, en cualquier escalón: cortar por número fijo de caracteres a ciegas. Un corte a mitad de frase o mitad de tabla no es un chunk: es un fragmento de chuleta roto que el índice tomará como ciudadano de pleno derecho.

## El tamaño no se elige: se hereda

Aquí está la tesis del capítulo — y una de las ideas del método que más se pregunta en la operación: *"¿de qué tamaño hago los chunks?"*. La pregunta está mal formulada. El tamaño **nace en el Domain Scoping** (cap. 2), a través de la **píldora de información** — la unidad mínima autocontenida que responde a una pregunta típica del dominio:

- **Dominios densos y monográficos** (normativa, convenios, manuales técnicos): la píldora es grande — una tabla completa, un artículo con sus apartados, una definición con sus excepciones. El chunk grande se beneficia: contiene la píldora entera, con su contexto, y el modelo la recibe sin ir al adivino.
- **Dominios atómicos** (FAQs, procedimientos cortos, fichas): la píldora es pequeña. El chunk pequeño se beneficia: precisión máxima, cero paja alrededor, y cada vector apunta a un significado compacto.

La regla de conducta del chunker cabe en dos prohibiciones: **no partir una píldora; no fusionar píldoras ajenas**. Entre las dos prohibiciones vive todo el arte: el chunk es del tamaño exacto de lo que responde a una pregunta — ni menos (píldora partida) ni más (temas ajenos de relleno).

### La calibración, como decisión técnica explícita

Dicho esto, los números existen y conviene situarlos con honestidad. La literatura habitual trabaja con chunks de **256–1024 tokens** porque los modelos de embedding clásicos pierden calidad con textos largos. Este método permite topes mayores — **hasta 8k tokens** — por dos razones específicas que deben ir siempre juntas: los embedders de contexto largo (aquí, **Qwen3-Embedding-8B**, con ventana de 32k tokens) sostienen semántica útil en textos extensos, y la segmentación del capítulo 8 ya garantiza que el chunk grande es *temáticamente puro* — un solo tema, no una mezcla. La combinación "documento atómico + embedder largo + resumen en cabecera" es lo que hace defendible el 8k. Sin esas condiciones, el rango clásico es la elección correcta — y quien copie el número sin las condiciones fabricará exactamente la dilución que este método quiere evitar. Y si la calibración se revela equivocada con el Rack ya en producción, tampoco es una crisis: el **patrón A→B** del capítulo 18 permite reconstruir el índice con la nueva estrategia sin interrumpir el servicio — el coste del error es cómputo, no credibilidad. (Nota de caducidad: los modelos citados en el libro — Qwen3-Embedding-8B, Tesseract, cualquier otro — son ejemplos con fecha, 2026. Los nombres envejecerán; el patrón no. Para eso existe el A→B.)

## El matiz que obliga al equilibrio

Y ahora la advertencia honesta, porque el libro no vende chunk grande a ciegas: **un solo vector por chunk muy grande beneficia la completitud de contexto pero difumina la precisión de recuperación**. El embedding condensa el chunk en un punto del espacio; cuanto más contenido condensa, más "promediado" es ese punto — y una pregunta concreta ("¿cuántos días de asuntos propios?") compite contra todo el bloque entero embebido en un punto. Un chunk enorme puede **contener** la respuesta exacta y aun así **no ser recuperado**, superado por un chunk pequeño y menos completo que se parece más a la pregunta.

Dos amortiguadores de este método compensan esa dilución:

- El **resumen en cabecera**, que concentra el significado del chunk en sus primeras líneas — y por tanto en su embedding.
- La **estrategia del doble campo** del capítulo 12, que embebe un resumen/descripción y entrega el chunk completo — la versión estructural de la misma solución.

Con ambos, el chunk grande es defendible. Sin ellos, no lo es: ahí está el equilibrio que este capítulo promete — y por eso el orden de lectura importa: este capítulo no funciona sin el 12.

## Sliding window con overlap

Cuando el corte parte contenido continuo (escalones 2 o 3 de la cascada), los chunks consecutivos **se solapan**: el final del anterior se repite al inicio del siguiente. Un solape del **10%** es el punto de partida razonable — suficiente para que una idea cortada en dos esté completa en al menos uno de los dos chunks, sin duplicar media colección.

Dos precisiones que evitan abusos:

- **El overlap solo tiene sentido entre chunks del mismo tema continuo.** Si el corte es un cambio de sección — cambio de tema de verdad — no se solapa: el solape artificial entre temas distintos ensucia ambos chunks con paja de recambio, y fabrica falsas similitudes.
- **El overlap no sustituye a la cascada.** Un overlap generoso (20-30%) intenta reparar cortes mal hechos: es parche, no solución. Si sientes la necesidad de solapes enormes, la cascada se está saltando — el corte está cayendo donde no debe.

## El contexto jerárquico: el breadcrumb

Un chunk se lee fuera de su documento — en la búsqueda, en el contexto del modelo, en la respuesta al usuario. Sin corrección, el chunk que empieza en "El plazo será de veinte días hábiles..." no dice de qué plazo, en qué norma, ni en qué condiciones. La solución es el **breadcrumb**: cada chunk incorpora, al inicio, la cadena de títulos que lo precede en el documento:

> `Convenio colectivo 2025 > Capítulo III. Jornada y horarios > Artículo 14. Registro de jornada`
> *(texto del chunk...)*

Con esa línea, el chunk es autocontenido: cualquier consumidor — humano, LLM o auditor — sabe dónde está, de qué documento y bajo qué norma. Los títulos no son decoración: son la dirección postal del significado. Y hay un efecto técnico que conviene apreciar: el breadcrumb también **entra en el embedding**, y aporta los términos generales ("registro de jornada") que el texto del chunk puede dar por sabidos — exactamente los términos que las preguntas de los usuarios usan.

La estructura final de todo chunk Silver queda así:

```
[Resumen del documento origen — 3 líneas]
[Breadcrumb: jerarquía de títulos hasta el punto del corte]
[Contenido del chunk en Markdown]
```

Cada chunk sabe quién es, de dónde viene y qué contiene.

## El resumen en cabecera

Completando la autocontención, cada chunk lleva al inicio un **resumen de tres líneas del documento origen** — no del chunk: *del documento*. Su función es anclar el significado global: "Tabla salarial del Convenio X del sector metal (2025). Sueldos base y complementos por categoría profesional. Vigente desde enero de 2025."

La distinción parece menor y es esencial. El resumen del chunk describiría el trozo; el del documento ancla el trozo a su totalidad — y es lo que permite que una pregunta general ("¿qué sabe el sistema del convenio del metal?") encuentre *cualquier* chunk del documento, no solo el que literalmente habla de "el convenio". Las tres líneas son el ADN del documento replicado en cada célula.

## El registro del troceado

Cada chunk generado actualiza el manifiesto: los chunks son filas propias, con su `doc_id` y su `num_chunk` respecto al documento ("3/12") — imprescindible para recomponer el documento, para el enriquecimiento por proximidad (los vecinos del 3 son el 2 y el 4) y para la auditoría de los fallos de recuperación — y el estado del documento pasa a `SILVER_CHUNKED`.

Y aquí la idempotencia documental (cap. 1) cumple su vertiente de chunks: **la re-ingesta de un documento sustituye sus chunks; no los acumula**. Si el original cambia en origen, el documento se re-procesa y sus chunks anteriores se retiran del índice — todos a la vez, porque el `doc_id` los identifica como conjunto — y entran los nuevos con su nueva numeración. Si el original no cambia, los chunks no se tocan: no hay re-troceado por moda ni por nerviosismo. La regla complementaria la aplicará la sincronización del capítulo 18: un documento dado de baja en origen se lleva sus chunks consigo; un chunk sin documento vivo detrás es un zombi — el ingrediente principal de los RAG que citan cosas que ya no existen.

## Errores frecuentes en el chunking

**El troceador uniforme.** Ya dicho en la apertura, pero queda en la lista porque sigue siendo el defecto número uno: un solo número de tokens para todo el corpus. Cada dominio — quizá cada tipo de documento — merece su calibración heredada de su píldora.

**El chunk-mosaico.** Fusionar píldoras ajenas para "aprovechar" el presupuesto: la definición de jornada junto a la tabla salarial "porque cabían". El chunk debe ser puro; el espacio sobrante no es desperdicio que rellenar.

**El breadcrumb incompleto.** Incluir solo el título inmediato y no la cadena entera ("Artículo 14" sin "Capítulo III" ni el nombre del convenio). El contexto jerárquico vale por su totalidad: cada eslabón que falta es ambigüedad que el modelo resolverá mal.

**El overlap entre temas.** Ya dicho, se repite porque es sutil: el solape es una técnica de continuidad, no un relleno. Entre cambio de temas, el solape ensucia.

**Trocear sin registrar.** Chunks creados sin filas en el manifiesto o sin `num_chunk`: invisible hasta el primer diagnóstico de recuperación ("¿por qué no aparece esta tabla?" — ¿esta tabla es el chunk 3 o el 7 de qué documento?), cuando ya no hay respuesta.

!!! quote "Regla del capítulo"
    el chunk perfecto es autocontenido (breadcrumb + resumen), temáticamente puro (píldora íntegra) y del tamaño exacto de su píldora. Ninguno de los tres criterios es un número: todos vienen del dominio.

