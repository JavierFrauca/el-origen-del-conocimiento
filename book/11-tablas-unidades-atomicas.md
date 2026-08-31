# Capítulo 11: Tablas como unidades atómicas (Table-Aware Chunking)

## El sótano donde viven las respuestas

En casi todo pipeline de RAG, las tablas viven en el sótano del sistema: se les aplica el mismo chunker que al texto — con el mismo número fijo de caracteres — y el resultado es una colección de fragmentos a mitad de fila, números sueltos sin encabezado, y embeddings producidos por una sopa de cifras y barras verticales que no significa nada para nadie. El resultado operativo es predecible y cruel: **las preguntas más concretas y frecuentes del negocio — "¿cuánto cobra un oficial de primera?", "¿qué porcentaje de cotización aplica?", "¿cuántos días por antigüedad?" — son las que el sistema responde peor.**

Y son exactamente las que el usuario pregunta primero. La primera consulta real a un sistema de conocimiento de RRHH casi nunca es conceptual ("¿cuál es la filosofía del convenio?") — es numérica: un sueldo, un porcentaje, un plazo. El proyecto se juzga por lo que la tabla respondía y el sistema no sabe.

Este libro da la vuelta al papel de las tablas: **son ciudadanos de primera clase del Rack**, con derechos propios:

1. **Derecho a la integridad**: una tabla nunca se corta. Ni si, ni medias. Cabe entera en un chunk o se gestiona de otra forma, pero no se parte.
2. **Derecho a la presentación**: se representan en Markdown con su estructura (filas y columnas como tales), nunca como texto plano aplastado.
3. **Derecho a la explicación**: ninguna tabla viaja sola; siempre lleva una descripción de qué contiene.

## Por qué nunca se cortan

La regla "las tablas son atómicas" suena a capricho hasta que se mira una tabla cortada de cerca. Una tabla es una **unidad de significado indivisible**, por dos razones que se refuerzan:

**Cada fila solo se entiende en relación a los encabezados.** "Oficial de primera | 1.890 | 240" — ¿1.890 qué, mensual o anual, base o con complementos? ¿240 de qué? Sin la fila de encabezados, es ruido con formato numérico. Y el corte por presupuesto no respeta encabezados: parte donde toca el número, y la mitad inferior de la tabla queda huérfana.

**Muchas filas solo se entienden en relación al conjunto.** Totales, escalas, progresiones por antigüedad, la nota al pie que dice "las categorías marcadas con * incluyen plus de nocturnidad". La fila 12 de una escala necesita a veces la fila 1 — y el corte no lo sabe.

El daño es doble, y de signo distinto en cada mitad:

- **En la recuperación**: el fragmento sin encabezados casi nunca es el más similar a la pregunta — no contiene las palabras "sueldo", "categoría", "mensual" que el usuario usa. La mitad enterrada de la tabla es prácticamente irrecuperable por vía semántica.
- **En la generación**: si llega a ser recuperado, el modelo recibe números sin contexto y — esta es la parte peligrosa — los *presentará* con seguridad, quizá atribuyendo a la columna equivocada. La tabla cortada es la deriva directa de la respuesta incorrecta con apariencia de precisión.

**La excepción operativa.** Tablas *genuinamente enormes* — cientos de filas que exceden cualquier presupuesto de chunk — existen (las escalas completas de un convenio multisectorial). Ni en ese caso se corta a ciegas: se divide **por grupos de filas completos, repitiendo los encabezados en cada fragmento** y manteniendo cada fragmento en su propio chunk. La decisión se documenta en el manifiesto; el encabezado repetido es innegociable. La regla no es "las tablas no se dividen": es "las tablas no se dividen *por aritmética*" — se dividen por sus filas, con su mapa en cada trozo.

## El formato: Markdown estructurado

El modelo canónico del capítulo 6 ya apuntaba aquí; este capítulo es su comprobación de calidad. Una tabla lista para el Rack en Markdown:

```markdown
| Categoría            | Sueldo base (€/mes) | Complemento (€/mes) |
|----------------------|---------------------|---------------------|
| Auxiliar             | 1.790               | 120                 |
| Oficial de 2ª        | 1.890               | 165                 |
| Oficial de 1ª        | 2.150               | 210                 |
```

La comprobación de calidad, en tres puntos:

- **Encabezados de columna presentes y legibles** — y con las unidades cuando el original las llevaba solo en el título de la tabla ("Sueldo base (€/mes)"): la unidad viaja con la columna, no con el documento.
- **Celdas limpias**: sin restos de celdas fusionadas del original (el parsing las rompe con frecuencia — la celda "2025" que en el Excel abarcaba diez filas llega como una fila vacía o como una fila con diez copias; ambas están mal y se reparan a mano).
- **Notas al pie de tabla conservadas y adjuntas**: el asterisco y su explicación se separan en el parsing con facilidad, y sin la explicación el asterisco es ruido que además confunde.

## La descripción de los datos: obligatoria

Una tabla no se explica sola. "2150 | 1890 | 240" no significa nada sin su mapa. Por eso toda tabla llega al Rack con una **descripción** que responde a tres preguntas: qué contiene, qué representan las columnas, qué periodo y vigencia tiene:

> "Tabla salarial del Convenio del sector X para 2025. Sueldo base mensual y complemento de convenio por categoría profesional. Vigente desde el 1 de enero de 2025; incluye la revisión del segundo semestre pactada en diciembre."

Esta descripción no sustituye a la tabla — la **acompaña**. Y es mucho más que una cortesía documental: es la materia prima del capítulo siguiente, donde se convierte en el campo que se embebe. La aritmética de la recuperación es demoledora: el vector de la tabla cruda es una nube de números sin un solo término de búsqueda natural; el vector de la descripción contiene "sueldo base", "categoría profesional", "convenio", "2025" — exactamente los términos de la pregunta. **La tabla da los datos; la descripción, la recuperabilidad.** Un Rack con tablas bien formateadas pero sin descripciones devuelve "algo parecido"; con descripciones, devuelve la tabla exacta.

## Las tablas como documento atómico

El capítulo 8 ya anticipó el final del recorrido: en corpus normativos y técnicos, las tablas suelen merecer **vida propia como documento segmentado** — las tablas salariales separadas del articulado. Cuando eso ocurre, este capítulo se aplica con ventaja: el documento atómico es esencialmente tabular, su chunking es mínimo (la tabla entera, con su descripción, es el chunk) y su píldora coincide exactamente con lo que el usuario pregunta. Es el caso donde todo el método encaja de punta a punta: dominio denso → píldora tabular → documento atómico → chunk único → descripción embebida → tabla entregada.

La tabla completa del recorrido del pipeline:

| Fase | Qué hace con la tabla |
|---|---|
| Curado (cap. 7) | La conserva íntegra, con encabezados; repara o marca las rotas |
| Segmentación (cap. 8) | Le da documento propio si el dominio lo pide |
| Chunking (cap. 10) | Nunca la corta; chunk propio, con breadcrumb y resumen |
| Este capítulo | Formato Markdown + descripción obligatoria |
| Doble campo (cap. 12) | La descripción se embebe; la tabla viaja al LLM |

## Errores frecuentes con las tablas

**La tabla como imagen.** El PDF con la tabla "como captura de pantalla" y el pipeline que decide que "es una imagen, no se puede hacer nada". Sí se puede: es material de OCR estructurado (cap. 6), y su contenido es demasiado valioso para rendirse. Las tablas-imagen se inventarian como incidencia obligatoria, no como pérdida aceptada.

**El CSV disfrazado de Markdown.** Convertir la tabla a texto separado por comas "que también es legible". No lo es — ni para el modelo ni para el usuario: sin alineación ni estructura visible, el CSV es el TXT de las tablas. El canónico pide tablas Markdown reales.

**La descripción genérica.** "Tabla de contenidos del documento" — como descripción de una tabla concreta que define tipos de jornada. La descripción se escribe mirando la tabla, no la categoría de la tabla; si la misma descripción valdría para diez tablas distintas, no describe ninguna.

**El total sin las partes.** Cortar la tabla y conservar solo "la parte que importa" (los totales, las categorías principales). Quién decide qué importa hoy no sabe qué preguntará el usuario mañana — y la pregunta mañana será, con toda probabilidad, por la fila que se conservó poco.

!!! quote "Regla del capítulo"
    las preguntas con respuesta numérica son las más frecuentes y las menos perdonadas. La tabla íntegra con su descripción es la diferencia entre responder y parece que respondes.

