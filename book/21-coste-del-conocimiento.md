# Capítulo 21: El coste del conocimiento (crear y mantener)

## La segunda reunión de presupuesto

Primera reunión: se aprueba el proyecto. "Es una inversión", se dice, y se aprueba porque nadie puede rebatir lo que no tiene número. Segunda reunión, doce meses después. Misma sala, mismos participantes, una pregunta distinta: *"¿Cuánto nos cuesta mantener esto?"*

He visto terminar esa reunión de dos maneras, y la diferencia entre ambas no la decide la calidad técnica del sistema — en las dos, el sistema funcionaba. En la afortunada, alguien abre un documento y dice: "La creación costó 3.100 horas y 4.200 euros de cómputo, con un 8% de desviación sobre lo estimado, desglosada aquí por fases. El mantenimiento mensual son 55 horas y 300 euros de cómputo, que escalan con el volumen actualizado. Aquí está el histórico de los últimos seis ciclos que lo demuestra." La renovación se aprueba en quince minutos, porque los números contestaron antes que las opiniones.

En la desafortunada hay silencio. Alguien aventura "pues... lo damos por medio equipo, más o menos". Otro objeta que ese "más o menos" era el mismo que estimó la creación, y que se desvió. Y emerge la conclusión tácita de toda reunión a la que llega el silencio: *nadie sabe lo que cuesta; supongamos que mucho; recortemos.* El sistema no murió porque fallara. Murió porque nadie pudo demostrar lo obvio — que valía más de lo que costaba — porque nadie había anotado lo que costaba.

Este capítulo existe porque la industria documenta con lujo de detalle cómo construir un sistema de conocimiento y casi nunca cuánto cuesta construirlo y mantenerlo. Los tutoriales terminan en la primera query feliz; ningún presupuesto se aprueba con una query feliz. El coste no es un anexo contable del método — **es parte del método**, con la misma disciplina que el manifiesto o la suite de validación. Sin coste no hay rentabilidad evaluable; sin rentabilidad evaluable, no hay segundo presupuesto.

## El coste, parte del método

Conviene justificar esta pertenencia, porque la intuición dice que el coste es "cosa de administración", ajena al diseño. Es al revés: en este método, las decisiones de diseño **son** decisiones de coste, y quien no las puede poner en euros no las puede ni comparar ni defender.

¿Tamaño de chunk grande o pequeño? Es una decisión de calidad — y también una multiplicación del número de vectores, del coste de embeddings, del tamaño del índice y del coste de cada query servida. ¿Curar fino o curar a lo bruto? Es la etapa dominante del presupuesto humano. ¿Auditar con LLM o con muestreo humano puro? Es un intercambio de cómputo por horas de experto. Cada capítulo de este libro toma decisiones que acaban en la hoja de costes; quien las toma sin ver la hoja no está diseñando: está apostando.

Por eso el estudio de costes hereda las disciplinas del resto del método — el rigor del manifiesto (nada sin registrar), el criterio por etapas (cada fase su puerta), la verificación de la Parte VI (nada aceptado sin contraste). Y añade la suya propia: el coste se mide contra el **presupuesto**, que es su dataset áureo particular.

## Los cinco requisitos del estudio de costes

Un estudio de costes que no cumpla estos cinco requisitos no es un estudio: es una opinión con hoja de cálculo. Cada requisito existe porque su ausencia ha matado proyectos.

**Primero: preciado.** Cada elemento del pipeline tiene precio en unidades monetarias, y las unidades importan. Las horas no son intercambiables: una hora del experto del dominio — el recurso más escaso del sistema, el que firma los Gold — no vale lo mismo que una hora de preparación de capturas. El cómputo se divide en partidas reales: embeddings por token, páginas de OCR, llamadas del LLM auditor y del traductor, almacenamiento, índice, licencias. "Esfuerzo estimado: alto" no es un precio; es un estado de ánimo. El requisito es que cualquier línea del estudio se pueda convertir a dinero sin interpretaciones.

**Segundo: sistemático.** Un único modelo de costes, con plantilla, aplicado igual a cada dominio, cada ciclo y cada actualización. La razón es la comparabilidad: dos dominios estimados con métodos distintos producen dos números que no se pueden contrastar — y la primera pregunta de cualquier dirección será precisamente "¿y por qué el de Legal cuesta el doble por documento?". Con un modelo sistemático, esa pregunta tiene respuesta honesta: o el dominio es más difícil, o hubo un error — y ambas conclusiones son útiles. Sin modelo, la pregunta solo produce incomodidad.

**Tercero: por etapas.** El coste se adjunta a las fases del pipeline. Un número global — "el proyecto costó 3.000 horas" — no se puede diagnosticar, ni optimizar, ni comparar, ni defender cuando alguien propone recortarlo: ¿se recorta del curado, que es el 40%? ¿del scoping, que es el 5% y evita los re-trabajos? El global sirve para facturar; las etapas sirven para gobernar. Y hay una razón más profunda: las etapas de coste son las mismas que las etapas de calidad — cada puerta del Apéndice D lleva su línea de presupuesto — de modo que calidad y coste se examinan juntos, como ocurre en toda industria seria.

**Cuarto: medible.** Las horas y el cómputo se registran **cuando ocurren**, asociados a su etapa — y a ser posible, al rango de documentos tratados, que es lo que permite calcular costes unitarios reales. El cómputo es fácil: viene facturado por uso y basta mapear cada factura a su fase. Las horas exigen disciplina de registro — tiempos por etapa, no precisiones de cronómetro — pero son la materia prima de todo lo demás. El coste reconstruido de memoria seis meses después no es medición: es nostalgia con formato de tabla.

**Quinto: verificable al final.** Cerrada la creación — y cerrado cada ciclo de actualización — se compara estimado contra real, etapa por etapa, y las desviaciones se explican por escrito. Una estimación que nunca se contrasta es una estimación que nunca aprende: sus errores se repetirán idénticos en el próximo dominio, con la misma cara de sorpresa. La verificación convierte cada proyecto en el entrenamiento del siguiente.

## El coste de creación, etapa a etapa

El desglose natural sigue el propio pipeline. Cada etapa con su partida y su driver — el factor que realmente la dispara:

| Etapa | Qué cuesta | Driver dominante |
|---|---|---|
| Scoping (talleres con expertos, fichas, dominios) | Horas de experto + arquitecto | El número de dominios |
| Bronce (captura, crawling, manifiesto) | Cómputo + horas de ingesta | Volumen de documentos |
| Normalización y OCR | Cómputo de OCR + revisión humana | Calidad de los escaneados |
| Curado y segmentación | **Horas humanas — la etapa dominante** | Densidad y calidad del corpus |
| Clasificación y auditoría automatizada | Muestreo humano + cómputo del LLM auditor | Tamaño del corpus |
| Chunking y embeddings | Cómputo de embeddings, por token | **Nº de chunks** |
| Gold | Horas de experto — el recurso más escaso | Nº de conceptos que merecen síntesis |
| Validación | Horas del dataset áureo + cómputo de la suite | Nº de queries del examen |
| Infraestructura | Almacenamiento Bronce, índice, licencias, modelos | Tamaños y retenciones |

Conviene leer la tabla despacio, porque cuenta una historia contraintuitiva. La etapa más cara no es la más tecnológica: es la más humana. El curado y la segmentación — horas de criterio experto decidiendo qué es paja, dónde parte un documento, qué etiqueta corresponde — dominan el presupuesto de creación en casi cualquier corpus real. Y el Gold, que es puro juicio experto, es el recurso más escaso por hora y el más difícil de ampliar: no se contrata conocimiento del dominio en un supermercado.

La segunda lectura importante: los *drivers* explican por qué las decisiones de las primeras partes son decisiones económicas. La calidad de los escaneados multiplica el OCR y su revisión — el capítulo 6 avisaba de los "hostiles" y aquí está su factura. El número de chunks — hijo directo del tamaño de chunk elegido en el capítulo 10 — multiplica embeddings, índice y coste por query. La granularidad de la píldora, definida en el capítulo 2, decide cuántas horas de curado y cuántos chunks habrá. **Quien dibuja la ficha del dominio está, sin saberlo, dibujando el presupuesto.**

Un ejemplo con cifras redondas — puramente ilustrativo, para fijar la forma, no los valores — para el dominio `convenios` con un corpus de 2.000 documentos y un 15% de escaneados:

- Scoping: 3 talleres con expertos y las fichas — 60 horas (40 de experto, 20 de arquitecto).
- Bronce y captura: 80 horas entre cómputo y supervisión.
- OCR de los 300 escaneados + revisión muestreada: 60 horas de revisión y su cómputo.
- Curado y segmentación: 250 horas — la partida mayor, como siempre.
- Clasificación con muestreo del 5% y auditoría automatizada: 60 horas más el cómputo del auditor.
- Embeddings del corpus troceado: cómputo según nº de chunks (aquí es donde el tamaño de chunk se convierte en euros).
- Gold inicial: 25 conceptos que la auditoría demostró críticos — 100 horas de experto.
- Dataset áureo y primera validación: 100 horas + cómputo.
- Infraestructura: almacenamiento Bronce, índice vectorial, licencias — partida mensual desde el primer día.

El total — en horas por perfil y en cómputo — es lo que se presenta, se aprueba y, terminado el trabajo, se verifica.

## El coste de mantenimiento

El mantenimiento (cap. 20) tiene dos componentes que conviene separar con cuidado, porque se comportan de manera opuesta.

El **coste fijo** es el del sistema despierto aunque nadie toque nada: almacenamiento del Bronce, índice vectorial, licencias, y la vigilancia mínima — curadores de dominio, revisión de incidencias, las rutinas del capítulo 18. Es la factura de existir.

El **coste variable** es el del ciclo de frescura, y su propiedad fundamental es que **escala con el delta, no con el corpus**: cada ciclo cuesta en función de los documentos nuevos o modificados × el coste unitario del pipeline completo (curado → chunking → embedding → auditoría), más los Gold re-validados por sus expertos, más la ejecución de la suite. Un mes con veinte documentos actualizados cuesta un mes con veinte documentos actualizados — no un mes de re-procesar dos mil.

Y aquí la idempotencia del capítulo 1 cobra su salario en euros, con la regla económica más importante del libro: **el coste marginal de un documento sin cambios es cero.** No re-procesar lo que no cambió no es solo limpieza — es la partida principal del presupuesto de mantenimiento. Un diseño idempotente hace que el coste variable escale con el delta real; un diseño sin él escala con el corpus entero, cada vez. Dicho en la jerga de las reuniones de presupuesto: el primero es predecible; el segundo, una amenaza.

Del modelo salen los **costes unitarios** que permiten planificar: coste por documento ingerido, por chunk, por query servida, por Gold producido. Con ellos, las preguntas difíciles se vuelven aritmética. "¿Cuánto costaría añadir el dominio `normativa-laboral`?" — coste unitario por documento × volumen estimado + scoping + Gold previsto. "¿Qué pesa más a largo plazo, el chunking A o el B?" — coste por chunk × nº de chunks de cada estrategia, más el diferencial de coste por query. Los unitarios convierten el presupuesto en una herramienta de decisión, no en un recibo.

## La elección de modelos: el importe que nadie ajusta

Dentro del coste de creación y mantenimiento hay una partida que casi nadie ajusta y que puede mover el total más que cualquier optimización de fases: **qué modelo hace cada tarea**. La intuición por defecto — "el modelo más potente dará el mejor resultado" — es muchas veces falsa, y casi siempre cara. El criterio correcto es emparejar la **capacidad del modelo con la complejidad real de la tarea**, y la buena noticia es que las tareas del pipeline se clasifican limpiamente.

La mayoría no son tareas de razonamiento: son **extracción sintetizada**. Un resumen de tres líneas, una descripción de tabla, una traducción de descripción: el modelo parte de un conocimiento ya dado — el documento está delante — y lo comprime o lo transforma sin hilar nada nuevo. No hay que razonar, no hay que conectar argumentos dispersos, no hay que decidir bajo incertidumbre: hay que leer bien y comprimir fielmente. Para eso, **un modelo barato lo hace igual de bien que uno caro** — y como estas tareas son de alto volumen (una descripción por chunk, una traducción por documento), la diferencia de precio se multiplica por todo el corpus. Aquí, el modelo caro no compra calidad: multiplica la factura.

La excepción está en la capa Gold — y es la excepción que justifica el gasto. Cuando el borrador de una síntesis se propone con IA supervisada por humanos, la tarea sí es de razonamiento: **hilar** conocimiento disperso — veinte chunks de tres dominios — en una explicación coherente, jerárquica y correctamente citada. Ahí un modelo caro y capaz no es un lujo: puede ser un **ahorro a largo plazo**, porque produce mejores borradores a la primera, y cada error que no tiene que corregir el experto es una hora del recurso más caro y escaso del sistema que se ahorra. La aritmética es aplastante: el sobrecoste del modelo caro se mide en céntimos por documento; la hora del experto que no corrige, en decenas de euros. En el Silver, el modelo caro multiplica su precio por el volumen sin comprar calidad; en el Gold, compra la calidad que ahorra las horas más caras del proyecto.

Y la regla que evita decidir por fe: **la prueba del escalón inferior**. Antes de asignar un modelo, comparar en una muestra real del corpus su salida contra la del escalón superior — contra el checklist de fidelidad de la auditoría (cap. 9) y el muestreo humano que ya existen. Si hay paridad en *tus* documentos, se baja de escalón y se ahorra para siempre; si no la hay, la diferencia medida justifica el gasto. La maquinaria de verificación ya existe — esto es la suite del método aplicada al propio presupuesto. Y como todo en el contrato del Rack: los modelos elegidos se registran en el manifiesto (resumidor, traductor, embedder, auditor, borrador de Gold — cada uno con su versión), su cambio dispara el patrón A→B cuando afecta contenido embebido, y se **re-precian periódicamente** — los precios de los modelos caenconstantemente, y el coste que hoy parece fijo, mañana es negociable.

## La verificación: estimado contra real

Cada etapa cierra con su comparación estimado/real, y las desviaciones significativas son **incidencias de gobierno, no de contabilidad**. La regla de lectura: el sobrecoste casi nunca es un problema de precios — es un problema de calidad anunciándose. Si el OCR costó el triple, es que los escaneados eran peores de lo que el mapa de fuentes decía. Si el curado se disparó, es que el corpus tenía más temas mezclados de los que la segmentación previó — o que las reglas de paja fallaron más de lo muestreado. El sobrecoste es el síntoma; el diagnóstico está en el pipeline, y los capítulos de donde viene la desviación son los que hay que revisar.

El cierre de la creación — y de cada ciclo de actualización — produce un documento breve que vale por todo el capítulo: el **acta de costes**. Estimado, real, desviación y lección, por etapa. Ese acta es lo que la segunda reunión de presupuesto tendrá delante; y es, a la vez, el entrenamiento del modelo de costes: la estimación del siguiente dominio se construye sobre los reales del anterior. A cada ciclo, el modelo predice mejor — que es exactamente lo que una organización necesita poder decir: *"la creación costó lo previsto con un 8% de desviación; el mantenimiento mensual es Y y escala con el delta; aquí está el histórico que lo demuestra."*

## Y la rentabilidad

Este libro se ocupa del coste porque es la mitad del análisis que controla el arquitecto; la otra mitad — el valor — pertenece al negocio, y solo la enmarca. Lo que sí cabe afirmar es que el Rack produce, por diseño, sus propios indicadores de valor: queries servidas con su calidad medida (la suite del cap. 17), horas de búsqueda manual evitadas (cada respuesta con fuentes trazadas que antes exigía una tarde de arqueología documental), errores evitados (las contradicciones resueltas del cap. 13 son errores que ya no ocurren), conocimiento que no se fue con quien se marchó. La rentabilidad se discute juntando ambas columnas — y la primera condición para tener la discusión es que la columna del coste exista, desglosada y verificada. Sin ella, la conversación no es un análisis: es un pulso.

## Errores frecuentes en el coste

**El coste invisible.** Nadie anotó nada en su momento — ni horas, ni cómputo, ni por dónde. Llegado el momento de renovar, no hay números, y la renovación compite contra proyectos que sí los tienen. Es la muerte más injusta: no por fallar, sino por no poder demostrar que funcionaba a precio.

**El coste global.** Un número único sin etapas: "el proyecto costó 3.000 horas". No se puede diagnosticar dónde se dispara, ni optimizar, ni responder a "¿y si lo reducimos a la mitad?" — porque nadie sabe qué mitad.

**La estimación huérfana.** Se estima al inicio con mimo y jamás se contrasta con el real. El modelo no aprende; las mismas desviaciones se repiten ciclo tras ciclo, siempre con cara de sorpresa y siempre imputadas a "lo imprevisto" — que, verificado, habría dejado de serlo.

**El mantenimiento gratis.** Asumir que actualizar "no cuesta nada" porque no hay un número escrito. A la primera presión presupuestaria, lo gratuito se corta sin dolor aparente — y el Rack empieza a caducar en silencio (cap. 20). El mantenimiento sin coste declarado no es gratuito: es impopular por sorpresa.

**Ahorrar donde no importa.** Recortar el scoping — la etapa barata que evita los re-trabajos — mientras el curado, la etapa dominante, corre sin control. La economía del método es asimétrica: gastar en lo temprano y vigilar lo pesado. Hacerlo al revés es el único error de costes que además de cara, es evitable con dos horas de lectura de este libro.

**El modelo caro por inseguridad.** Usar el modelo insignia para cada resumen, descripción y traducción "por si acaso". La factura se multiplica por el volumen entero del corpus; la diferencia de calidad, cuando el trabajo es extracción sintetizada, no existe o es invisible en la auditoría — y nadie se atrevió a bajar de escalón porque nadie midió. La cura es barata: la prueba del escalón inferior. Y su inverso igual de erróneo: economizar el modelo del borrador de Gold, donde un modelo incapaz de hilar fabrica borradores mediocres que el experto corrije a mano — pagando la hora más cara del proyecto para ahorrar céntimos.

!!! quote "Regla del capítulo"
    el conocimiento se crea una vez y se mantiene para siempre — y sin coste por etapas, medido al ocurrir y verificado al terminar, no hay rentabilidad demostrable. Sin rentabilidad demostrable, no hay siguiente presupuesto.

---

Y una referencia cruzada que este capítulo obliga a hacer: la **conversión A→B** del capítulo 18 — cambiar chunking, embedder o dimensiones con el Rack en producción — es, económicamente, el coste más evitable de todos. Se estima antes de decidir, se valida contra el beneficio esperado en métricas, y viene con una advertencia que no admite suavizado: **es un coste extra por no haber analizado correctamente en las fases iniciales.** Cada A→B es la factura de una decisión tomada deprisa en los capítulos 2, 10 o 12. El patrón sigue siendo el mejor seguro disponible — pero el estudio de costes está ahí para recordar que iterar también cuesta, y que la forma barata de iterar es equivocarse menos al principio.

