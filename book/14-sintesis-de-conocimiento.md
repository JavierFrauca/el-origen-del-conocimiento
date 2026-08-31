# Capítulo 14: Síntesis de conocimiento (Knowledge Synthesis)

## El hueco entre saber y entender

La capa Silver responde a "¿qué dice tal fuente?". Pero las preguntas del usuario rara vez citan fuentes: preguntan por **conceptos**. Y los conceptos, en un corpus real, viven dispersos: el IRPF aparece en Legal (la norma), en Finanzas (la aplicación práctica), quizá en RRHH (la retención en nómina). Cada pieza es correcta; ninguna, completa. Y cuando hay conflicto (cap. 13), además se contradicen — y el usuario recibe las dos, con la misma seguridad, sin saber cuál manda.

Entre "saber todos los hechos" y "entender el concepto" hay un hueco, y nadie lo rellena solo. Es el hueco que resuelve la capa **Gold**: documentos de **síntesis** que explican un concepto de forma holística — unificando criterios, resolviendo contradicciones y enlazando a cada fuente original. Son el "lo que entendemos" del manifiesto medallón: no un hecho más, sino la **interpretación autorizada** de los hechos.

Vale la pena detenerse en la palabra autorizada, porque contiene la diferencia entre esto y todo lo anterior. El Silver es verificable — cualquier persona puede comprobar que el chunk dice lo que dice la fuente. El Gold es *autorizado* — alguien con conocimiento y responsabilidad declara cómo se cruzan los hechos y qué conclusión se extrae. La IA no fabrica autoridad; la registra.

## La capa human-guided: el criterio no se automatiza

Hay una propiedad de esta capa que conviene declarar antes de cualquier detalle técnico, porque es de las pocas reglas sin matices del método: **la capa Gold es una capa human-guided — dirigida por humanos**. Los documentos Gold los define, los escribe y los corrige una inteligencia con **amplio y profundo conocimiento del dominio**: un experto que conoce las fuentes, su jerarquía, sus matices y su historia. No es una recomendación de estilo ni una salvaguarda opcional: es una **condición estructural de la capa**. Sin eso, no hay Gold — hay un texto con pretensiones.

La razón se entiende mirando lo que un Gold exige. Definir cuál es la verdad operativa de un concepto exige saber cuál de las fuentes manda — y eso no está escrito en ningún sitio: está en la experiencia de quien ha trabajado años con esas fuentes. Redactar la síntesis exige saber qué matiz importa y cuál es ruido. Corregirla exige poder discutirla con la norma delante. Nada de eso se delega: una síntesis escrita por quien no domina el dominio tiene la **forma** de la verdad sin su **contenido** — y en esta capa, la forma sin contenido es el producto más peligroso posible, porque llega al usuario con la marca de autoridad puesta.

Los tres verbos del oficio, y quién los ejerce:

- **Definir**: qué conceptos merecen Gold, qué criterio aplica, qué fuentes lo sustentan. El experto del dominio.
- **Escribir**: el borrador puede proponerlo una herramienta — un LLM bien instruido acelera, y es un uso legítimo — pero bajo la dirección del experto, con su estructura y su criterio, no el de la máquina.
- **Corregir**: cada afirmación, cada enlace a fuente, cada matiz — revisados, corregidos y firmados por el experto. La dirección de la responsabilidad nunca se invierte: **la herramienta propone; el conocimiento del dominio dispone.**

Y la consecuencia operativa, que reaparecerá en la ingesta (cap. 15): **no existe Gold sin experto detrás**. La label `SINTESIS_CURADA` certifica, entre otras cosas, que una persona con amplio conocimiento del dominio ha pasado por ahí a definir, escribir y corregir ese documento. Si eso falla, el resto de la capa es decoración.

## Cuándo se produce un Gold

Tres detonantes, todos procedentes de la auditoría del capítulo 13:

1. **Dispersión legítima**: un concepto repartido entre varios documentos o dominios que las preguntas típicas cruzan — el `relacionado_a` lleno de vecinos. Síntoma: las sondas devuelven siempre el mismo paquete de piezas de dominios distintos.
2. **Conflicto resuelto**: una contradicción marcada (`estado_conflicto`) que un experto ha podido resolver — la fuente correcta es una, y el Gold lo declara. Es la reparación del modo contradicción: no borrar la fuente errónea, sino declarar públicamente qué manda y por qué.
3. **Hueco de conocimiento**: las sondas muestran preguntas frecuentes que ninguna fuente responde de forma completa. El Gold se redacta *ex novo*, citando lo que existe, y el hueco restante queda documentado como tarea de contenido — a veces el hallazgo más valioso de la auditoría: descubrir que la organización no tiene escrito algo que todo el mundo pregunta.

## Los tres tipos de Gold: un catálogo de patrones

Cada detonante produce un tipo de Gold con forma propia — y conviene conocer los tres porque piden cosas distintas a su autor:

- **El Gold unificador** (por dispersión). No resuelve ningún conflicto: junta. Su cuerpo recorre el concepto por sus dominios — "en la norma está X [doc], en la aplicación práctica Y [doc], en nómina se materializa Z [doc]" — y su valor es ahorrar al usuario el recorrido que el experto ya hizo. Es el tipo más frecuente y el más barato de producir.
- **El Gold arbitraje** (por conflicto). El más delicado: declara qué fuente manda y **descarta** explícitamente la otra, con motivo. Su valor es jurídico-operativo: cierra la discusión. Exige más al experto — está firmando que una fuente pierde — y es el que más conviene revisar en las auditorías periódicas, porque la derogación de mañana lo dejará cojo.
- **El Gold relleno** (por hueco). El más creativo: redacta conocimiento que ninguna fuente contenía de forma completa, citando lo que existe y señalando lo que falta. Es también el más temporal: su parte "rellenada" debería terminar convirtiéndose en una fuente formal — cuando la organización escriba lo que faltaba, el Gold relleno se actualiza y deja de inventar.

El error de catálogo es usar el mismo molde para los tres: un Gold arbitraje redactado como unificador esconde el descarte entre párrafos neutros, y el usuario no se entera de que había dos versiones. Cada tipo tiene su estructura; el experto elige el molde.

## El documento Gold

Un Gold no es un resumen — la palabra tentación que conviene desactivar ya. Un resumen condensa un texto; un Gold **explica un concepto con trazabilidad completa**. Su estructura canónica, con el caso del IRPF ya relleno:

```
Concepto: IRPF_autonomos_calculo_2025
Resumen (3 líneas):
  "Documento oficial que unifica el criterio del IRPF 2025 para
   autónomos: aplica el 21% según la ley vigente, descarta el
   borrador del 19% y remite a la tabla de cuotas de Finanzas."

Cuerpo:
  Para el cálculo del IRPF 2025 en autónomos se aplica el tipo del
  21% según [Doc A: ley vigente]. El tipo del 19% que aparece en
  [Doc B: borrador de enero] no llegó a promulgarse y queda
  descartado. La aplicación práctica de cuotas se detalla en
  [Doc C: tabla de Finanzas].

Fuentes: [A], [C]
Resuelve conflictos: [B]
Autor: [experto del dominio, fecha]
```

**Trazabilidad no negociable**: cada afirmación material del cuerpo enlaza a su fuente por `doc_id`. Un Gold sin enlaces a fuentes no es un documento Gold: es un Silver más, disfrazado — y peor, porque su tono de autoridad no lo respalda nada. Su valor — poder señalar con el dedo a cada original que lo sustenta — es exactamente lo que lo distingue.

**Estilo**: escrito para ser la respuesta, no para ser un informe. El Gold del IRPF no relata el proceso de investigación ("se han revisado las siguientes fuentes..."): declara el criterio vigente como lo declararía el experto al experto que pregunta. Ese es el texto que el usuario quiere leer y el que el modelo debe recibir.

**Quién lo redacta**: el experto del dominio — la capa es human-guided, como se declaró al abrir el capítulo. La jerarquía es exacta: una herramienta puede proponer el borrador, pero el experto **valida cada afirmación y cada enlace a fuente**, corrige lo que haga falta y firma. La firma no es un gesto formal: convierte el documento en *su* verdad — y es saber a quién preguntar cuando el Gold se discuta.

## El caso práctico, con sus efectos

Recordemos la sonda del capítulo 13: "¿Cómo se calcula el IRPF para autónomos?" — Top-10 con el artículo de la ley vigente (tipo 21%), el borrador de enero nunca promulgado (19%), y la tabla de cuotas de Finanzas. Auditoría: conflicto marcado entre los dos primeros; dispersión legítima con el tercero. El Gold resultante es el de arriba.

Tres efectos, inmediatos y medibles. **Para el usuario**: quien pregunte recibe la interpretación correcta con sus fuentes a la vista — no dos tasas compitiendo. **Para el sistema**: el conflicto tiene resolución declarada; el consumidor del Rack (el que aplique el boost de la label del capítulo 15) tiene una razón material para ponerlo primero. **Para la organización**: la verdad queda con nombre, fecha y fuentes — la próxima vez que alguien discuta el 19%, hay un documento que ya respondió.

Y una nota de humildad técnica: sin Gold, este concepto requeriría que el modelo "resolviera" el conflicto en tiempo de respuesta — eligiendo entre el 19% y el 21% según su criterio de momento. Fabricar autoridad en el acto, con apariencia de conocimiento. El Gold es la diferencia entre dejarle la decisión al modelo y habérsela dejado a un experto con nombre y apellido.

## Single Source of Truth

La regla de acero de la capa: **un concepto, un Gold**. Si dos síntesis explican lo mismo, el Rack ha duplicado su verdad — y dos verdades son ninguna: el Top-K las traerá juntas, con matices distintos, y el usuario volverá a estar donde estaba sin Gold, pero ahora con la sensación añadida de que "esto es poco fiable".

El cumplimiento se garantiza con la clave `concepto_unico` y la mecánica de ingesta del capítulo 15 (UPSERT), y se audita periódicamente: ninguna síntesis nueva se crea sin consultar antes si el concepto ya existe. Cuando el conocimiento cambia — nueva reforma, nueva tabla — no se crea un Gold segundo: se **actualiza el existente** y su traza (fecha, fuentes). La historia de versiones vive en el linaje, no en el índice; el índice contiene la verdad, una sola, con su biografía al lado.

## Los Gold no son infinitos

Contrapeso necesario, porque el entusiasmo post-auditoría lleva a excesos: la capa Gold no crece sin criterio. Si cada documento mereciera una síntesis, el Gold sería el Silver repetido con pasos extra — más contenido que mantener, más verdad que vigilar, cero ganancia. El criterio de contención: **se sintetiza lo que las sondas demuestran que se pregunta, se cruza o se contradice — no lo que podría preguntarse.** La auditoría del capítulo 13 es, además del detector de problemas, la lista priorizada de dónde vale la pena invertir horas de experto. Un Gold que nadie necesita es ruido con autoridad — y ruido con autoridad es peor que ruido.

## Qué no hacer: cuando la síntesis se convierte en opinión

Tres formas de estropear un Gold, todas vistas en proyectos reales:

**El Gold que opina sin fuente.** En la redacción, el experto añade "de todos modos, lo recomendable es..." — un consejo sin `doc_id` que lo respalde. El consejo puede ser excelente; sin fuente, rompe el contrato de trazabilidad y convierte el resto del documento en sospechoso. Si el consejo importa, se busca su fuente o se documenta como recomendación explícita, distinguida del contenido normativo.

**El Gold que copia el Silver.** Sintetizar por sintetizar: el "Gold" que parafrasea un único documento sin añadir cruce ni criterio. No aporta lo que su marca promete — y ocupa la posición de autoridad que podría ocupar una síntesis verdadera. Si no hay cruce ni conflicto ni hueco, no hay Gold.

**El Gold huérfano.** Firmado por "el equipo", sin experto identificable. Cuando dentro de un año alguien discuta el criterio del documento — y se discutirán —, "el equipo" no podrá defenderlo. La autoría con nombre no es burocracia: es el plan de mantenimiento del Gold.

!!! quote "Regla del capítulo"
    el Gold es la diferencia entre un Rack que devuelve documentos y un Rack que da respuestas. Pero solo si cada palabra de la respuesta se puede rastrear hasta su fuente — síntesis sin trazabilidad es exactamente el problema que este libro prometió arreglar.

