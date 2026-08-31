# Créditos y agradecimientos

## Autores

**Javier Frauca** — Arquitectura del método, visión y decisiones de diseño. La tesis del libro es suya: todo el mundo se centra en usar los LLMs y nadie en el origen de su conocimiento. También suyas son las aportaciones que definen el carácter del método: el concepto de **píldora de información** y la regla de que el tamaño del chunk se hereda del scoping del dominio; la decisión de que los documentos Gold viven en el mismo índice que el Silver, identificados por una etiqueta y sin tablas aparte; la escala de calidad de la validación (recall mínimo del 90% como suelo duro); y la experiencia de taller que hay detrás de cada "regla del capítulo". El nombre lo dice el propio método: Bronce, Silver y Gold — y la verdad operativa siempre en Gold.

**ZCode** (agente de IA, modelo GLM 5.3 Flash de Z.ai) — Estructuración del manuscrito, curado de nomenclatura, redacción del borrador y revisión crítica. Aportó el rigor de nombrar cada idea con su término técnico de industria — Domain Scoping, faceted classification, table-aware chunking, small-to-big retrieval, golden dataset — y de señalar los límites honestos de cada decisión: la dilución semántica de los chunks grandes, el punto ciego de los resúmenes ante búsquedas literales, lo que el recall no mide.

## Agradecimientos

Este libro nació de una conversación: un arquitecto de datos cansado de ver RAGs convertidos en desastres, sentado a diseñar con una IA el método que le habría ahorrado esos desastres. Y se escribió, capítulo a capítulo, mientras ese método se ponía a prueba en un RAG real — el de un despacho laboral — cuyos convenios, tablas salariales y contradicciones del 19% contra el 21% habitan estas páginas. El resultado — un guion de trinchera que se convirtió en índice, y un índice que se convirtió en libro — es un experimento de coautoría entre humano y máquina, y quizá la mejor demostración de su tesis: **el conocimiento del origen es humano; la herramienta, cualquiera que ayude a cultivarlo.**

A los arquitectos de datos que llevan años haciendo esto sin ponerle nombre — este libro les debe los términos y les devuelve el método. Y a ese RAG real, que padeció el libro mientras lo enseñaba.
