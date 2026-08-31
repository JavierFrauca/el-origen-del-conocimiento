# Capítulo 23: De dónde venimos — notas, referencias y deudas intelectuales

Este método no inventa casi nada nuevo — lo dice el prólogo y lo repite el epílogo. Inventar era lo de menos; el valor estaba en la combinación. Pero la combinación se compone de piezas prestadas, y cada una tiene su autor o su casa. Este es el inventario honesto, con una declaración que lo precede: **ninguno de estos patrones es creación de este libro.** Este libro es un integrador — coge piezas ajenas, les añade criterio de montaje y las presenta ordenadas. Esta tabla devuelve a cada pieza su autoría:

| Patrón prestado | A quien se atribuye | Dónde profundizar |
|---|---|---|
| **Arquitectura Medallón** (Bronce/Silver/Gold) | Databricks — divulgación del patrón lakehouse (sin autor único) | Documentación de Databricks |
| **RAG** | Patrick Lewis et al., 2020 | *"Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"* (NeurIPS) |
| **Idempotencia** | Práctica estándar de sistemas distribuidos y HTTP | RFC 9110 (semántica de métodos) |
| **CDC** | Práctica de replicación de bases de datos; ecosistema Debezium | Documentación de CDC |
| **Blue-green** | Divulgación de Martin Fowler (2010); práctica previa del sector | bliki de Martin Fowler |
| **No-deployment / tests de regresión** | Jez Humble y David Farley, *Continuous Delivery* (2010) | Libro y comunidad de CD |
| **Muestreo de calidad** | Walter A. Shewhart (1931), control estadístico de la calidad | Literatura clásica de SPC |
| **Clasificación facetada** | S. R. Ranganathan (1933), biblioteconomía | Teoría de clasificación facetada |
| **BM25** | Stephen Robertson y Karen Spärck Jones (1976); Robertson & Zaragoza (2009) | Literatura de recuperación probabilística |
| **Fusión RRF** | Gordon Cormack, Charles Clarke y Stefan Buettcher (2009) | Paper original de RRF |
| **Breadcrumb + resumen en cabecera** | Variante propia del *contextual retrieval* de Anthropic (2024) | Blog de ingeniería de Anthropic |
| **Lost in the middle** | Nelson F. Liu et al., Stanford (2023) | Paper *"Lost in the Middle"* |
| **Small-to-big / parent-document retrieval** | Ecosistema LangChain / LlamaIndex | Documentación de ambos frameworks |
| **Table-aware chunking** | Práctica consolidada del ecosistema RAG | Artículos y frameworks de chunking |
| **GraphRAG** | Darren Edge et al. (Microsoft Research, 2024) | Paper *"From Local to Global"* |
| **Qwen3-Embedding-8B** | Equipo Qwen (Alibaba) | Ficha del modelo |

Tres notas de honestidad sobre esta tabla. 
    **Primera**: i alguna edición o año difiere, la obra se reconocerá por su título, y las erratas se aceptan por las vías del repo. **Segunda**: los enlaces mueren; las obras permanecen. Por eso se citan obras y autores, no URLs. 
    **Tercera**: alguna deuda es con el sentido común acumulado de una profesión — no todo tiene paper, y pretenderlo sería falsedad académica.

## El contraste que pide un lector serio: ¿y las alternativas?

La reseña también preguntó cómo se hace esto en LangChain, en LlamaIndex, o en los papers recientes de retrieval. La respuesta honesta tiene tres partes:

- **Los frameworks** (LangChain, LlamaIndex, Haystack y familia) aportan los **componentes**: chunkers, retrievers, fusionadores, evaluadores. Este método aporta el **criterio** que decide cómo usarlos: qué píldora define el corte, qué puerta para la ingesta, qué cifra detiene un despliegue. Son compatibles por diseño — el cap. 10 es el criterio, y el chunker de tu framework es la herramienta.

- **Los evaluadores de RAG** (RAGAS, TruLens y familia) miden la **generación**: fidelidad al contexto, relevancia de la respuesta, calidad de la redacción. Este libro mide deliberadamente la **recuperación**, y por eso declara que ambos enfoques se complementan: primero el Rack aprueba su examen; después, y solo después, tiene sentido evaluar lo que el modelo escribe con ese contexto.

- **Los papers avanzan más rápido que cualquier libro.** Nuevas técnicas de chunking, de embedding y de fusión aparecerán entre esta edición y su tercera lectura. El método no compite con ellas: les pone el marco — qué problema resuelve cada técnica, y con qué dataset se demuestra.

En una frase para la solapa: **este libro no compite con los frameworks ni con los papers — les entrega un corpus con contrato.**
