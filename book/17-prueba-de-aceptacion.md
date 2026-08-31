# Capítulo 17: La prueba de aceptación (Retrieval Evaluation)

## Cinco minutos que lo deciden todo

Con el dataset áureo construido, la prueba de aceptación es un procedimiento mecánico y repetible — tan mecánico que debería poder ejecutarlo quien sea, el día que sea, sin criterio experto:

1. **Ejecutar cada query** del dataset contra el índice — búsqueda vectorial pura sobre el campo de representación, sin LLM, sin prompt. (Si el consumidor ya tiene política de rerank o boost, se puede validar el conjunto — vector + boost — siempre que el criterio se fije por escrito y no cambie entre ejecuciones: lo que invalida una suite no es su configuración, es su variabilidad.)
2. **Recuperar el Top-K** acordado en el dataset (10 por defecto).
3. **Cruzar listas**: comparar los IDs recuperados con los `chunks_esperados`.
4. **Calcular las métricas** por query y el agregado del dataset.
5. **Veredicto**: aceptar, o parar el despliegue y diagnosticar.

Cinco minutos de ejecución — y la diferencia entre un sistema gestionado y una caja negra con suerte. Todo lo que este libro defendió con argumento se mide aquí con números.

## La métrica principal: Recall@K

Para cada query:

**Recall = (chunks esperados que aparecen en el Top-K) / (total de chunks esperados)**

La pregunta que responde es la única que importa en un contrato de suministro de conocimiento: *de la verdad que debía salir, ¿cuánto salió?* Es la métrica principal porque el fallo catastrófico del Rack es el **enterrado** (cap. prólogo): el dato correcto que existe, está indexado, y nunca aparece. El recall mide exactamente eso.

## La escala de calidad

La escala de este método es deliberadamente exigente — los umbrales suaves fabrican confianza falsa:

- **Recall < 90% → Quality Score = 0**. Suspenso automático de la query, sin matices.
- **Recall entre 90% y 100% → calidad lineal**: `Quality Score = (Recall − 0,9) × 10`.

Es decir: recall del 90%, calidad 0; del 95%, calidad 50; del 100%, calidad 100. La regla de tres castiga con dureza la zona baja — estar "cerca de aprobar" no se premia, porque un usuario real no recibe "casi la respuesta": recibe una respuesta, que o era la verdadera o no lo fue.

**El umbral de aceptación del Rack**: Quality Score **medio ≥ 95%** sobre todo el dataset. Debajo, el despliegue se para: la versión nueva del índice no sale a producción hasta diagnosticar y reparar. Y la escala tiene una propiedad psicológica que conviene apreciar: al hacer que el 90% valga cero, elimina la negociación — nadie "está a un punto de aprobar", porque del 89% al 90% no hay un punto, hay un abismo. Los sistemas con este tipo de escalas se gestionan mejor: el equipo apunta al 100%, no al umbral.

## El ejemplo, completo

Dataset: *"¿Cómo se calcula el IRPF 2025 para autónomos?"* — esperados: el Gold del concepto (`gold_001`), el artículo de la ley (`silver_legal_045`) y la tabla de cuotas (`silver_finanzas_123`). Top-K = 10.

Resultado de la búsqueda: `gold_001` en primera posición; `silver_legal_045` en tercera; `silver_finanzas_123` **no aparece** (puesto 15).

- Recall = 2/3 = 66,7% → **inferior al 90% → Quality Score = 0**. La query suspende.

Y ahora lo importante — la parte que convierte un número en una reparación: el diagnóstico ordenado. Por eso las métricas por query se guardan, no solo la media:

1. **¿Se partió la píldora?** (cap. 10): ¿la tabla quedó en un chunk cortado, o sin su encabezado, o fusionada con contenido ajeno? Si es así, el fallo es de chunking — el primero en revisar, siempre, porque es el más frecuente y el más barato de corregir.
2. **¿Tiene descripción?** (cap. 11–12): una tabla sin descripción en el campo de representación es prácticamente irrecuperable por vía vectorial — ningún embedding de cifras se parece a una pregunta.
3. **¿Está etiquetada correctamente?** (cap. 9): dominio y facetas correctos; ¿no quedó excluida por una vigencia mal asignada, o por un filtro que el consumidor aplica por defecto?
4. **¿El embedding es el problema?** (cap. 12): descripción correcta y aún enterrada — entonces sí, ajustar la descripción (más concreta, con los términos que usan los usuarios) o replantear el campo de representación.

En el ejemplo: la tabla no tenía descripción. Se redacta, se re-ingiere, se re-ejecuta la suite. La media vuelve al verde; se despliega. Nota cómo el diagnóstico recorrió el método al revés — chunking, doble campo, clasificación, embeddings — en orden decreciente de frecuencia de fallo: esa lista ordenada es, de facto, el manual de reparación del Rack.

## Un segundo ejemplo: la query que sí pasa

Para calibrar el ojo, una query en verde. Dataset: *"¿Cuántos días de asuntos propios corresponden?"* — esperados: el artículo del convenio (`silver_art_012`) y el Gold que unifica convenio y acuerdo de condiciones (`gold_003`). Top-K = 10.

Resultado: `gold_003` en puesto 2; `silver_art_012` en puesto 1. Recall = 2/2 = 100% → **Quality Score = 100**. Y la lectura fina que el registro permite: el Gold salió segundo, no primero — ¿problema? No: el primer puesto lo ocupa el artículo que el propio Gold cita como fuente. Para una pregunta de hecho con matiz conceptual, ese orden es correcto. La suite mide; la interpretación sigue siendo humana — pero ahora se interpreta sobre números, no sobre impresiones.

## Leer los patrones de fallo: el diagnóstico epidemiológico

Con varias ejecuciones acumuladas, los fallos del dataset dibujan patrones que se diagnostican como un médico lee síntomas:

- **Falla un dominio entero** → problema de ingesta o clasificación de ese dominio (revisar sus etapas, caps. 6–9), no del índice global.
- **Fallan solo las queries numéricas** → chunking de tablas o descripciones ausentes (caps. 10–12). Es el patrón más frecuente en primer despliegue.
- **Fallan solo las queries con Gold esperado** → la capa de autoridad no está emergiendo: revisar la label `SINTESIS_CURADA` y la calibración del boost que el consumidor aplica.
- **Falla una query que antes pasaba, sin cambios en su dominio** → regresión por la última ingesta: algo nuevo enterró lo viejo. Revisar qué entró desde la última suite en verde.
- **Degradación lenta y generalizada, semanas** → caducidad silenciosa: fuentes desincronizadas o vigencias sin actualizar. El síntoma del modo fantasma llegando.
- **Recall en verde pero ruido alto** (más del 30% del Top-K bajo el umbral de ruido — la regla del 30% de la sección siguiente) → ruido estructural: descripciones genéricas, píldoras mezcladas o facetas flojas (caps. 12, 10 y 9).

Cada patrón apunta a un capítulo. Esta tabla de traducción — síntoma → capítulo — es, probablemente, la página más práctica del libro.

## Los límites de la métrica, dichos en voz alta

El recall mide que la verdad **salga**; no mide que salga **sola**. Un Top-K con los 3 esperados y 7 irrelevantes puntúa 100% — y el usuario recibió mucha basura elegante. Dos complementos, que el dataset permite calcular sin coste adicional:

- **Precisión@K**: proporción del Top-K que era relevante. Penaliza el ruido recuperado — el complemento exacto del recall: él mira lo que falta, esta mira lo que sobra.
- **MRR** (*Mean Reciprocal Rank*): en qué posición media aparece lo esperado. Un Gold en puesto 1 y otro en puesto 9 no son la misma calidad de servicio; el recall los empata, el MRR los distingue. En particular, el MRR vigila la eficacia de los boosts: si los Gold "aparecen" pero en el puesto 7, algo de la política de ponderación no funciona.

Y una advertencia de granularidad que conviene asumir como diseño y no como defecto: con listas de esperados pequeñas (2–4 chunks), el recall es grueso — un fallo en 3 esperados es 66%, suspenso directo. Es **intencionado**: este método prefiere falsas alarmas a confianza ciega. Pero obliga a definir la **tolerancia de despliegue**: cuántas queries individuales pueden suspender mientras la media ≥ 95%. Recomendación de partida: **ninguna con Gold en los esperados** — si una síntesis autorizada no aparece, la capa de verdad está fallando y no se negocia; el resto, hasta un 5% con diagnóstico documentado.

Y sobre las cifras de este capítulo — 90%, 95%, 50 queries, tolerancia del 5%: son los **puntos dulces que funcionaron sobre una solución real**, no leyes físicas. Cada dominio puede — y debe — calibrarlas contra su primera suite completa, igual que calibra su chunking (cap. 10). Lo que no es ajustable es lo que representan: que la verdad salga, que salga arriba, que el ruido esté acotado y que nada se despliegue sin examen. Quien discuta una cifra discute una calibración; quien pretenda quitar el examen, discute el libro entero.

## El tratamiento del ruido: la regla del 30%

La precisión@K mide el ruido, pero cobra horas humanas: juzgar los siete chunks sobrantes de cada query es trabajo manual que se repite en cada despliegue. El score que ya traen los resultados permite medirlo **gratis** — y por eso la suite incorpora una segunda puerta de aceptación, automática: **la regla del 30%**.

El problema que resuelve primero es el de cualquier umbral de score: los scores **no son comparables** entre queries ni entre modelos. Un 0,72 puede ser excelente para una pregunta y mediocre para otra, y al cambiar de embedder (patrón A→B) los números cambian de escala. Un umbral fijo — "todo lo que baje de 0,7 es ruido" — se rompe por construcción. Por eso el umbral de este método **no es un número: se calibra solo, query a query**:

```
umbral_q = score del PEOR chunk esperado de esa misma query
ruido_q  = nº de chunks del Top-K con score < umbral_q
```

Los chunks esperados — cuya relevancia ya fue decidida leyendo las fuentes (cap. 16) — definen en cada query dónde acaba lo relevante; todo lo que puntúe por debajo de ellos es, por definición operativa, ruido. La regla no depende del embedder ni de la escala de puntuación: sobrevive al patrón A→B sin tocar una sola línea.

**La regla, con Top-K = 10: máximo 3 chunks de ruido por query** — mínimo 7 de cada 10 resultados por encima del umbral. Una query que supere el 30% de ruido suspende esta puerta aunque su recall sea perfecto: es exactamente el caso "los 3 esperados y 7 irrelevantes" que el recall no ve.

**La cifra es un default, no un dogma.** El 30% es el punto dulce encontrado sobre el corpus real de este libro — el despacho laboral — pero cada dominio tiene el suyo, y encontrarlo es parte de la puesta a punto: en la primera suite completa, mirar la distribución de ruido de las queries sanas (las que pasan recall con claridad) y fijar el techo en un margen razonable por encima de esa base. Un dominio muy cruzado — Legal y Finanzas compartiendo concepto — genera más vecinos legítimos que uno de FAQs, y su techo puede ser algo más generoso. Lo que no es negociable: que la cifra **exista, esté escrita y se mida en cada despliegue**. El ruido no medido no se mantiene — se acumula.

Y una precisión sobre qué **no** cuenta como ruido para esta regla: los **chunks redundantes** — dos documentos legítimos distintos que dicen lo mismo con redacción distinta — puntúan alto, porque son relevantes. No hay que purgarlos: son el seguro de vida del sistema (si uno de los dos documentos se da de baja, la información sobrevive en el otro — cap. 13). La redundancia entre fuentes legítimas se acepta; su priorización es política del consumidor, con los instrumentos de siempre: rerank, dominios y labels. Lo que la regla del 30% caza es lo otro: los chunks que puntúan bajo el peor esperado — irrelevantes, no repetidos.

**La frontera, como siempre**: como **criterio de aceptación**, la regla es del Rack — la suite la mide y puede parar un despliegue. Como **corte en producción** — no enseñarle al usuario los chunks que quedan bajo el umbral — es política del inquilino: el Rack garantiza que el score existe y que el ruido es medible; aplicar el filtro en cada pregunta es su decisión.

## Un caso de éxito: la noche que el examen detuvo un despliegue

Cerramos con la escena que justifica la suite entera. Viernes, cinco de la tarde: ingesta del nuevo procedimiento de gastos de viaje — ochenta documentos, todo correcto, todo validado en curado. La suite corre antes del despliegue, como siempre. Media: 96%. Verde aparente. Pero una query suspende: *"¿Qué percibos exige un desplazamiento internacional?"* — y entre sus esperados estaba la tabla de perfiles del antiguo procedimiento, que la nueva ingesta ha empujado al puesto 14.

Sin suite: despliegue, y durante semanas los usuarios reciben el procedimiento nuevo en lugar de la tabla que necesitaban — nadie se entera hasta que alguien reclama, y entonces nadie sabe qué cambió. Con suite: el despliegue se para a las 17:40, el diagnóstico apunta a la ingesta de la tarde, se descubre que el nuevo procedimiento y la tabla vieja compartían el concepto sin etiquetar, se corrige la clasificación, la suite vuelve al verde a las 19:00. Ochenta documentos, una corrección, un viernes salvado. **Eso es lo que hace el examen: no evitar los errores — cogerlos antes de que cobren.**

Y no es una fábula edificante: en la CD real de este método, una ingesta green con **23 documentos rechazados bloqueó su propio promote** hasta resolverse — el no-deployment trabajando en producción, con logs que lo certifican.

## Qué NO es esta prueba

No evalúa la calidad de la respuesta final del LLM — fidelidad al contexto, redacción, utilidad: eso es evaluación de RAG, fuera del ámbito del libro. Evalúa el **contrato de suministro del conocimiento**: *pregunta X → verdad Y disponible en Top-K*. Ese contrato es el que permite, con datos, decir la frase más útil de toda discusión técnica: "el problema está en la base y no en el modelo" — o al revés, que también pasa, y también conviene saberlo.

Y una última propiedad, la que justifica su existencia operativa: la suite es **comparativa**. Cada ejecución se compara con la anterior — misma muestra, mismo criterio — y esa comparación es la que convierte una ingesta cualquiera en un despliegue gobernado: no "¿funciona?", sino "¿funciona igual de bien que antes, más lo nuevo?".

!!! quote "Regla del capítulo"
    si el Rack no supera el recall, ningún prompt del mundo salvará la respuesta final. Y si lo supera, el día que algo empeore, la suite lo gritará antes que los usuarios.

