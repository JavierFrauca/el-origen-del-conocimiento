# GLOSARIO

### Apéndice E de *El Origen del Conocimiento*

> **Documento independiente**: pensado para consultarse, imprimirse y repasarse solo — veinte minutos de lectura que cubren todas las tecnologías y conceptos del libro. Pertenece a *El Origen del Conocimiento* ([volver al libro](25-apendices.md)).

Un solo glosario, por orden alfabético. Cada entrada reúne el **nombre que le da el libro**, el **término técnico de industria** y su **definición explicada**. Repasarlo entero: veinte minutos que equivalen a releer el libro.

---

**Agente de IA** — Programa que encadena llamadas a un LLM y a herramientas para alcanzar objetivos por sí solo. Potente y útil, pero **no determinista**: hoy hace una cosa, mañana otra. Por eso en este método nunca conduce el pipeline — procesa piezas dentro de él (cap. 22).

**Aceptación, prueba de (UAT)** — Examen con casos conocidos que un sistema debe superar antes de entrar en producción. Aquí: la suite del cap. 17 ejecutada sobre el dataset áureo del cap. 16 (caps. 16–17).

**A→B, patrón** — Migración de configuración (chunking, motor de embeddings, dimensiones) con el Rack en producción: B se construye en paralelo desde el origen, se valida y solo entonces sustituye a A (cap. 18).

**Búsqueda híbrida** — BM25 (léxico) + vectorial (semántico) fusionadas por RRF. El léxico encuentra los literales; el vector, los significados (cap. 12).

**BM25 (Best Matching 25)** — Ranking léxico clásico: puntúa documentos por coincidencia de términos, con saturación y penalización por longitud. Entiende palabras exactas, no sinónimos: por eso se combina con lo vectorial en la búsqueda híbrida. Robertson y Spärck Jones (1976); Robertson y Zaragoza (2009) (cap. 12).

**Boilerplate / paja semántica** — Texto repetitivo sin información propia: cabeceras, pies, avisos legales, numeración residual. El curado lo elimina conservando el 100% de lo que informa (cap. 7).

**Breadcrumb** — "Miga de pan": la cadena de títulos que preceden a un chunk, incorporada al propio chunk para que sea autocontenido ("Convenio 2025 > Cap. III > Art. 14"). La dirección postal del significado. Junto con el resumen en cabecera, variante propia del *contextual retrieval* de Anthropic (2024) (cap. 10).

**Bronce / Silver / Gold** — Las tres capas de la Arquitectura Medallón aplicada al conocimiento: materia prima intocada, conocimiento estructurado y verdad sintetizada. *Lo que tenemos, lo que sabemos, lo que entendemos* (cap. 1).

**Blue-green deployment / índice gemelo** — El índice nuevo (verde) se construye y valida en paralelo mientras el antiguo (azul) sigue sirviendo; con validación en verde se conmuta el tráfico, y el azul se conserva como rollback. Divulgación de Martin Fowler (2010) (cap. 18).

**CDC (Change Data Capture)** — Mecanismo por el que las fuentes **notifican** sus cambios (eventos, webhooks, diffs) en lugar de que el pipeline los sondee. Horizonte natural de la ingesta incremental. Práctica de replicación de bases de datos; ecosistema Debezium (cap. 18).

**Chunk** — Fragmento de un documento que se indexa como unidad de recuperación: se embebe, se busca y se entrega al LLM como pieza de contexto. El átomo del Rack (cap. 10).

**Chunking contextual / Troceado** — Corte en cascada (sección → párrafo → semántico) con contexto jerárquico (breadcrumb), resumen en cabecera y overlap; su tamaño se hereda de la píldora del dominio (cap. 10).

**Contradiction detection / información que se contradice** — Detección de afirmaciones incompatibles sobre el mismo hecho en el corpus. Superado el 5% de contradicciones en sondas, el pipeline se para (cap. 13).

**Corpus / corpus maestro** — La colección de documentos y conocimiento de la organización. El **corpus maestro** — originales, manifiesto, curado y Gold — es el activo capital que se custodia; el índice es solo su derivado (cap. 19).

**Curador de dominio** — El propietario de un dominio: vigila sus fuentes, dispara los deltas de actualización y mantiene su Gold (cap. 20).

**Dilución semántica** — Pérdida de precisión de recuperación al embeber textos muy largos: el vector promedia demasiado contenido y la pregunta concreta compite contra el bloque entero (cap. 10).

**Doble campo / small-to-big / parent-document retrieval** — Se indexa pequeño (el resumen del campo de representación) y se entrega grande (el chunk crudo completo). Cada campo hace el trabajo para el que es bueno: el resumen, ser encontrado; el crudo, ser entendido. Patrón del ecosistema LangChain / LlamaIndex (cap. 12).

**Dominio / Domain Scoping / acotar dominios** — Categoría mayor y estable del conocimiento, definida con ficha: preguntas que responde, fuentes con autoridad, píldora típica. Acotar no es limitar: es la estrategia de precisión (cap. 2).

**El corpus en Git** — El maestro versionado en repositorio: alta, modificación y baja como commits; ramas protegidas como gobierno mecánico; CD de ingesta idempotente (caps. 20 y 22).

**El examen del Rack / Golden dataset / dataset áureo** — Queries reales del negocio con los chunks que deben aparecer para cada una. El contrato de nivel de servicio del Rack, escrito en preguntas (cap. 16).

**El inventario / manifiesto de datos** — Inventario único del pipeline: una fila por documento con hash, orígenes, estados y notas. *"Si no está en el manifiesto, no existe"* (cap. 5).

**El pasaporte del documento / Data lineage** — La cadena de custodia del conocimiento: origen exacto, autoridad de la fuente, momentos y transformaciones de cada documento (cap. 3).

**El porcentaje de acierto / Recall@K, Precisión@K, MRR** — Las métricas de la prueba de aceptación: el recall mide que la verdad salga; la precisión, que no la acompañe ruido; el MRR, que salga arriba (cap. 17).

**El Rack** — Nombre propio de este método para la base de datos vectorial de conocimiento: el contenedor donde viven Silver y Gold. No es un rack de servidores ni es el corpus: es donde el corpus queda servido (cap. 1).

**El sótano / Raw Zone / zona cruda** — Volcado exacto e inmutable de las fuentes, inventariado en el manifiesto. La primera capa de la Medallón; el almacén al que solo se baja para re-procesar o auditar (cap. 4).

**Embedding / Embedder** — Vector numérico que representa el significado de un texto: textos parecidos producen vectores cercanos. El modelo que los produce (aquí, Qwen3-Embedding-8B, con ventana de 32k tokens) forma parte del contrato del Rack (cap. 12).

**Estudio de costes** — Coste preciado, sistemático, por etapas, medible y verificable; con costes unitarios por documento, chunk, query y Gold para planificar y comparar (cap. 21).

**Facetas / Etiquetas transversales (clasificación facetada)** — Ejes de clasificación transversales y multi-valor que cruzan todos los dominios: año, vigencia, naturaleza, confidencialidad, idioma. Los filtros del Rack; la técnica viene de la biblioteconomía de Ranganathan (1933) (cap. 9).

**Fusión RRF (Reciprocal Rank Fusion)** — Combina rankings de buscadores distintos por posición recíproca, sin comparar scores heterogéneos. Cormack, Clarke y Buettcher (2009) (cap. 12).

**Garbage in, garbage out** — La calidad de la salida la decide la calidad de la entrada. El primer principio del libro (cap. 1).

**Gold / documento de síntesis / síntesis de conocimiento** — Documento experto que unifica criterios, resuelve conflictos y enlaza fuentes con trazabilidad completa. Firmado por quien domina el dominio; la capa human-guided del método (caps. 14–15).

**Grafo de conocimiento / GraphRAG / Relaciones entre conceptos** — Conceptos como nodos y relaciones como aristas, combinables con la búsqueda vectorial. La frontera hacia la que evoluciona el Rack. Darren Edge et al., Microsoft Research (2024) (epílogo).

**Hash SHA-256 / la huella digital** — Huella criptográfica del contenido: deduplica copias, detecta cambios, dispara la idempotencia y audita la integridad del Bronce (cap. 5).

**Human-guided** — Dirigido por humanos: la capa Gold la define, la escribe y la corrige una inteligencia con amplio conocimiento del dominio. La herramienta propone; el conocimiento del dominio dispone (cap. 14).

**Idempotencia documental** — Si el original no cambia, nada posterior cambia; si cambia, se sustituye sin residuos; si desaparece, sale de la base. Re-ejecutar el pipeline deja el Rack en el mismo estado. Práctica estándar de sistemas distribuidos y HTTP (semántica de métodos, RFC 9110) (caps. 1, 5, 10, 18).

**La marca del Gold / SINTESIS_CURADA** — Label que marca los documentos Gold dentro del índice. La física del dato la pone la BBDD; la política de relevancia (boost, filtros), el consumidor (cap. 15).

**LLM (Large Language Model)** — Modelo de lenguaje generativo. En este libro es un **amplificador**: razona sobre lo que recibe, sin auditarlo jamás (cap. 1).

**Linaje / data lineage / el pasaporte del documento** — La cadena de custodia: origen exacto, autoridad de la fuente, momentos y transformaciones de cada documento (cap. 3).

**Lost in the middle** — Fenómeno por el que los modelos pierden información situada en posiciones intermedias de contextos largos. Uno de los motivos del overlap entre chunks. Nelson F. Liu et al., Stanford (2023) (cap. 10).

**Manifiesto de datos** — Inventario único del pipeline: una fila por documento con hash, orígenes, estados y notas. *"Si no está en el manifiesto, no existe"* (cap. 5).

**Matryoshka (embeddings)** — Modelos cuyos vectores admiten truncado a menos dimensiones conservando calidad relativa: cambia dimensiones sin re-embeber, aunque re-indexando (cap. 18).

**Medallón, Arquitectura (Medallion Architecture)** — Patrón de la industria del dato que organiza la información en capas de calidad creciente. La columna vertebral de este método. Divulgación de Databricks (sin autor único) (cap. 1).

**MRR (Mean Reciprocal Rank)** — Posición media del primer resultado relevante: mide qué tan *arriba* sale lo esperado. Complemento del recall, que solo mira si aparece (cap. 17).

**Muestreo de calidad** — Control por muestreo estadístico heredado de la manufactura: revisar una muestra aleatoria del trabajo automático y fijar un umbral de error aceptable. Walter A. Shewhart (1931), control estadístico de la calidad (cap. 9).

**No-deployment** — Regla de gobierno: si la suite de validación falla, el Rack no se actualiza. El equivalente exacto de no desplegar software con los tests en rojo. Jez Humble y David Farley, *Continuous Delivery* (2010) (cap. 18).

**Normalización de idioma** — El campo de búsqueda en un único idioma objetivo; el usuario pregunta en el suyo y la query se traduce al idioma de los vectores (cap. 12).

**OCR (Optical Character Recognition)** — Reconocimiento óptico de caracteres: convierte imágenes y escaneados en texto. Sin él, ese contenido no existe para el sistema (cap. 6).

**Overlap / sliding window** — Solapamiento entre chunks consecutivos del mismo tema continuo (~10%): garantiza que una idea cortada esté íntegra en al menos un chunk (cap. 10).

**Píldora de información** — Término propio de este método: la unidad mínima autocontenida que responde a una pregunta típica del dominio. Hereda el alcance de la segmentación y el tamaño del chunk; nace en el Domain Scoping (cap. 2).

**PII / RGPD** — Datos Personales Identificables y el Reglamento General de Protección de Datos europeo. La exclusión de PII es una frontera previa a la ingesta, no una faceta posterior (cap. 9).

**Pipeline repetible** — Scripts por transición de fase con contrato (determinista, idempotente, registrador); reconstrucción completa del Rack desde el origen; el LLM procesa piezas, nunca conduce el flujo (cap. 22).

**Precisión@K** — Proporción del Top-K recuperado que era realmente relevante. El complemento exacto del recall: él mira lo que falta, esta lo que sobra (cap. 17).

**Probe queries / queries sonda** — Consultas reales del negocio lanzadas contra el índice para auditar coherencia, cruces y contradicciones (cap. 13).

**Prompt engineering** — La disciplina de optimizar instrucciones al modelo. Deliberadamente fuera del ámbito de este libro (prólogo).

**Qwen3-Embedding-8B** — Embedder de contexto largo (32k tokens) usado como ejemplo del método: lo que permite defender chunks grandes con resumen en cabecera. Modelo con fecha de caducidad: el patrón sobrevive al cambio vía A→B. Equipo Qwen (Alibaba) (caps. 10, 12, 18).

**Rack** — Ver *el Rack* (cap. 1).

**RAG (Retrieval-Augmented Generation)** — Arquitectura que alimenta a un LLM con conocimiento recuperado de una base externa en el momento de cada pregunta. Lewis et al. (2020) (cap. 1).

**RAG vertical** — RAG construido sobre un dominio acotado (también llamado *vertical RAG*): la estrategia de precisión que defiende el cap. 2.

**Raw Zone / zona cruda** — Ver *el sótano* y *capa Bronce* (cap. 4).

**Recall@K** — Fracción de los chunks esperados que aparecen en el Top-K. La métrica principal de aceptación: mide si la verdad salió (cap. 17).

**Redundancia legítima** — La misma información en documentos distintos y legítimos, con redacción distinta. Se acepta como resiliencia — la baja de uno no mata la información — y su priorización es del consumidor (caps. 13, 17).

**Regla del 30%** — Puerta de ruido de la validación: ningún Top-K puede tener más del 30% de sus chunks por debajo del umbral de ruido auto-calibrado (el score del peor chunk esperado de esa query). Default calibrable por dominio (cap. 17).

**Re-embedido** — Regeneración de todos los vectores al cambiar de modelo de embeddings, con reconstrucción del índice en paralelo y validación completa antes de conmutar (cap. 18).

**Rerank / rerank boost** — Reordenación fina de los candidatos recuperados y suma de puntuación a los chunks marcados como Gold. Señales del Rack; política del consumidor (cap. 15).

**Síntesis de conocimiento / documentos curados** — Ver *Gold / documento de síntesis* (cap. 14).

**Small-to-big** — Ver *doble campo* (cap. 12).

**Sondas** — Ver *probe queries* (cap. 13).

**SSOT (Single Source of Truth) / la verdad única** — Un concepto, un Gold: unicidad garantizada por UPSERT y auditada periódicamente. La historia de versiones vive en el linaje, no en el índice (cap. 14).

**Table-aware chunking / Tablas que no se cortan** — Troceado que respeta las tablas como unidades atómicas indivisibles, con encabezados y descripción obligatoria. Práctica consolidada del ecosistema RAG (cap. 11).

**Tesseract** — Motor OCR clásico de código abierto: suficiente para escaneados limpios, insuficiente para los hostiles (cap. 6).

**Token** — Unidad mínima de texto con la que trabajan los modelos (~tres cuartos de palabra). El contexto de los LLM y el coste de embeddings y generación se miden en tokens (caps. 7, 10 y 21).

**UPSERT** — Operación "actualizar si existe, insertar si no": la forma idempotente de ingestar. El Gold se UPSERTEA por `concepto_unico` (cap. 15).

**YAML front-matter** — Cabecera de metadatos estructurados al inicio de un archivo Markdown. Porta las facetas mutables y los campos de gobierno del Gold (cap. 20).

