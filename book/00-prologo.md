# Prólogo: El síndrome del "copiar y pegar"

Hay una escena que se repite en demasiados equipos. Un comité, una reunión de dirección, o simplemente un jueves por la tarde: alguien enseña una demo de un asistente que responde preguntas sobre los documentos internos de la empresa. El asistente contesta con soltura, cita fuentes, hasta pide disculpas si no sabe algo. Se emociona la sala. Esa misma semana, alguien copia un trozo de código de un tutorial, apunta un modelo de lenguaje a la carpeta compartida donde duerme — desde hace una década — toda la documentación de la organización, y anuncia que la empresa ya tiene IA.

Las dos primeras semanas son el honeymoon. La demo funciona porque las demos se hacen con las cinco preguntas que ya sabemos que funcionan. Después llega la primera asamblea de personal, y el asistente, en público, con total fluidez y citando el documento de origen, afirma que el tipo de IRPF aplicable es el 19% — cuando la norma vigente desde hace dos años fija el 21%. El 19% estaba en un borrador de enero que nunca se promulgó, dormido en una carpeta llamada `Varios/Backup_nuevo(2)/`.

El diagnóstico casi nunca es el que se espera. No falla el modelo. No falla el prompt. Falla lo que le dimos a comer.

Este libro existe porque hay una asimetría cultural muy grave en la industria: toda la atención — cursos, tutoriales, charlas, frameworks, LinkedIn — se concentra en el **consumo** del conocimiento: cómo pedirle cosas a un modelo de lenguaje, cómo escribir prompts, cómo orquestar agentes. Y casi ninguna en el **origen** de ese conocimiento: de dónde sale, cómo se recopila, cómo se limpia, cómo se clasifica, cómo se valida, cómo se mantiene vivo. La preparación del dato se pasa a toda velocidad, como un trámite aburrido entre la idea y la demo. Y sin embargo, ahí — exactamente ahí — es donde se decide el resultado.

## El diagnóstico: una industria mirando al lado equivocado

¿Por qué pasa esto? No por ignorancia técnica, sino por economía de la atención. La demo es visible, inmediata y emocionante; el curado de dos mil PDFs es invisible, lento y aburrido. Las herramientas de orquestación se venden con vídeos de dos minutos; el inventario documental no tiene vídeo. El resultado es un sesgo sistemático: los proyectos se financian por su demo, y la deuda se firma en silencio en el momento en que alguien decide que "la carpeta ya está, eso lo limpiamos sobre la marcha".

Sobre la marcha, no se limpia nada. Sobre la marcha se hereda una década de copias, versiones en conflicto, escaneados ilegibles y políticas derogadas conviviendo con las vigentes. El sistema no distingue: ingiere todo con el mismo apetito, y luego — esta es la parte cruel — lo sirve con la misma seguridad.

## Anatomía de un desastre anunciado

Casi todos los fracasos de sistemas de conocimiento que he visto caben en tres modos, y los tres aparecen en la primera semana de uso real:

**El modo contradicción.** El sistema devuelve dos fuentes que dicen cosas incompatibles sobre el mismo hecho — el 19% y el 21% — y el modelo, que no tiene forma de saber cuál manda, elige una. O las mezcla. O las presenta ambas "para completitud". El usuario deja de confiar en la respuesta número tres, no en la primera.

**El modo enterrado.** El dato correcto existe, está bien extraído, está en el índice — pero nunca se recupera. La tabla salarial completa quedó troceada a mitad de fila por un chunker sin criterio, o su embedding es una sopa de cifras que no se parece a ninguna pregunta. El sistema responde con lo que encuentra — prosa genérica — y la tabla, que era la única respuesta, es invisible. Los usuarios dicen "no lo sabe", y tienen razón a efectos prácticos.

**El modo fantasma.** El sistema cita, con autoridad y enlace incluido, una política derogada hace tres años, una versión en borrador, un convenio sustituido. Nadie le dijo que el conocimiento caduca. Nadie le marcó qué versión manda. El fantasma hablaba con la misma voz que la verdad.

Tres modos, una misma raíz: **problemas del origen del conocimiento, malinterpretados como problemas del modelo**. La respuesta habitual — cambiar de modelo, afinar el prompt, añadir un agente — es cosmética. Este libro es el diagnóstico y el tratamiento del origen.

## La falacia del LLM omnisciente

El modelo de lenguaje no sabe nada de tu empresa. Merece la pena detenerse en esta frase porque la intuición popular dice lo contrario: los modelos "saben" muchísimo, contestan de todo, parecen cultísimos. Pero ese saber es sobre el **lenguaje**, no sobre tus **hechos**. Tu convenio colectivo, tu política de teletrabajo, tu tabla de retenciones: el modelo no los ha visto nunca. Es un motor de razonamiento sobre lenguaje extraordinariamente capaz, pero ciego: solo ve lo que le pones delante en el momento de la pregunta.

Y lo que hace con lo que le pones delante no es verificarlo: es **amplificarlo**. Si el contexto que le entregas es limpio, coherente y vigente, razona sobre esa base con brillantez. Si está sucio, contradictorio o desactualizado, lo presenta con la misma fluidez y la misma seguridad que la verdad — y ahí está el agravante: la fluidez del modelo convierte los errores del origen en errores *invisibles*. Un dato mal extraído por un OCR mediocre llegaba al usuario como un PDF con faltas, y desconfiaba. Llega ahora envuelto en una redacción impecable, y confía.

Por eso la regla que gobierna todo el libro es el más viejo de los principios de la informática: **garbage in, garbage out**. O en su versión optimista: la calidad de entrada es la única métrica que importa.

## El 80/20, invertido

La industria dedica el 80% de su esfuerzo al 20% del problema. Este libro propone lo contrario, y con una asimetría de costes que conviene verbalizar desde ya: **el 80% del éxito de un sistema de recuperación aumentada ocurre antes del prompt**, y hacer bien ese 80% en su momento es barato; hacerlo mal es carísimo de reparar después.

Barato en su momento: un manifiesto bien diseñado cuesta una tarde. Carísimo después: reconstruir la trazabilidad de cincuenta mil documentos sin manifiesto cuesta meses, si es que se puede. Un dominio bien acotado en el arranque cuesta una reunión. Re-etiquetar todo un índice porque los dominios se solapaban cuesta semanas y una migración. La preparación del conocimiento es la parte del proyecto donde el tiempo invertido temprano se multiplica y el tiempo ahorrado temprano se multiplica también — en contra.

El 20% restante — el modelo, el prompt, la orquestación — es la parte fácil, la visible, la que menos tolerancia al error transmite: no hay prompt que rescate un dato que no existe o que contradice a su vecino.

## El manifiesto del arquitecto del dato: agricultores, no mineros

El minero arranca lo que encuentra y sigue adelante: la veta se agota, y a otra cosa. El agricultor prepara el terreno, elige la semilla, riega, cuida, cosecha — y vuelve a la misma tierra la temporada siguiente. El conocimiento de una organización no es una veta que se explota una vez: es un **cultivo**. Un sistema de conocimiento no se construye y se entrega; se siembra y se mantiene. Tiene temporadas (las revisiones anuales de convenios), plagas (los documentos en conflicto), podas (lo deprecado) y cosechas (las respuestas fiables).

De esa manera de mirar se derivan las tres decisiones de carácter que estructuran todo el libro:

**Primera: nada se transforma sin dejar rastro.** Cada documento lleva pasaporte desde que entra (de dónde viene, cuándo, con qué firma digital) y bitácora en cada transformación. No por burocracia: porque sin trazabilidad no hay diagnóstico posible, y sin diagnóstico no hay mejora.

**Segunda: la verdad se declara, no se deduce.** Cuando dos fuentes discrepan, el sistema no elige la más parecida a la pregunta: alguien — un experto, con nombre y apellido — ha declarado cuál manda, y esa declaración vive en el sistema como un documento de primera clase.

**Tercera: "funciona" es un número, no una opinión.** El sistema se examina con una batería de preguntas reales con respuestas conocidas, y esa batería corre antes de cada cambio. Si el porcentaje de acierto baja, no se despliega. Igual que lleva décadas haciendo el software serio con sus tests.

## Para quién es este libro y cómo leerlo

Para el **arquitecto de datos** que ve llegar la IA generativa y sospecha — correctamente — que su oficio está en el centro de la tormenta. Para el **ingeniero** que ha montado un RAG y descubre que el 90% de sus bugs no son de código. Para el **responsable técnico** que tiene que decidir si ese proyecto de "IA sobre nuestros documentos" se apoya sobre algo. No hace falta saber programar para leerlo — no hay una línea de código, deliberadamente: el código envejece en meses y el criterio vale años — pero sí pensar como técnico: aceptar que las decisiones se documentan, que las fases tienen puertas de calidad y que "más o menos bien" en el origen es "del todo mal" en producción.

El método sigue la **Arquitectura Medallón** que la industria del dato usa desde hace años, aplicada de principio a fin al conocimiento no estructurado: **Bronce** (lo que tenemos), **Silver** (lo que sabemos), **Gold** (lo que entendemos). Dos casos correrían de principio a fin como hilos conductores: una normativa fiscal con dos tasas en conflicto — el caso IRPF — y un convenio colectivo con sus tablas salariales y su articulado. Entre ambos concentran todos los problemas del género: versiones, densidad, tablas, dispersión entre dominios, contradicción.

## El laboratorio: escrito mientras se construía

Una declaración de origen, porque explica media librería: **este libro no se escribió para contar cómo se hace — se fue escribiendo mientras se hacía.** Cada regla que vas a leer se puso a prueba, en vivo, sobre un RAG real que se construyó en paralelo: el de un despacho laboral. Los convenios con sus tablas salariales, el IRPF con su borrador del 19% conviviendo con la ley del 21%, las nóminas y los asuntos propios: no son ejemplos elegidos por un experto en pedagogía — son el terreno en el que el método se fue rompiendo y soldando a la vez. De ahí el tono: no es pose, es cicatriz.

Eso tiene dos consecuencias para el lector. La primera, de honestidad: si algún ejemplo te resulta excesivamente concreto — un convenio estatal con su código completo, una revisión salarial de 2025 — es porque lo es: son piezas reales de un corpus que existe. La segunda, de garantía: **nada de lo que sigue depende del despacho laboral.** El dominio fue elegido por el azar de quien lo necesitaba, y resulta que además es uno de los más exigentes posibles — normativa densa, tablas críticas, versiones que se pisan cada enero, conceptos que cruzan Legal y Finanzas. Si el método funciona ahí, la traslación a mundos más amables es directa:

| En este libro (despacho laboral) | En tu dominio puede ser... |
|---|---|
| Convenios colectivos y sus tablas salariales | Protocolos clínicos y sus dosis; manuales técnicos y sus listas de repuestos |
| El IRPF del 19% frente al 21% | La política de devoluciones de 2023 frente a la vigente |
| El articulado y sus apartados | El reglamento de un software, apartado por apartado |
| Legal y Finanzas cruzados por el IRPF | Producción y Calidad cruzadas por el mismo procedimiento |

La regla de traslación cabe en una frase: **donde este libro dice "convenio", di en tu mundo "documento denso y normativo"; donde dice "tabla salarial", di "dato tabular que el usuario pregunta exacto".** La ficha del dominio (cap. 2) hará el resto: el método es dominio-agnóstico; los ejemplos, uno solo de los mundos posibles — elegido, por una vez, por existir.

Y una convención de lectura: cada capítulo cierra con su **Regla del capítulo**, enmarcada — la frase que quedaría si el lector solo pudiera quedarse con una. Son las veinte frases que, pegadas en la pared, valen por el libro entero.

## La promesa

Al terminar, tendrás un método completo para construir y mantener lo que aquí llamamos el **Rack**: la base de datos de conocimiento que alimenta a un sistema de recuperación aumentada. *(Sí: **Rack**. Es el nombre propio que le daremos durante todo el libro — y no, no hablamos de los racks de servidores del centro de datos, aunque el malentendido es tan buen chiste que lo adoptamos: tu conocimiento entero, apilado y organizado en un mueble. Con una diferencia sustancial: aquel rack sí tiene botón de apagado.)* Podrás acotar dominios con precisión quirúrgica; recopilar e inventariar cualquier documentación, por caótica que esté; normalizarla y trocearla respetando su semántica, con las tablas intactas y cada fragmento con memoria de dónde viene; distinguir lo que sabes de lo que entiendes; resolver contradicciones con documentos de síntesis que se convierten en la verdad única; y validar el conjunto con pruebas medibles donde "funciona bien" deja de ser una opinión y se convierte en un número con umbral.

## La brújula: mapa, planos y hoja de ruta

Un libro se lee; un proyecto se planifica. Esta última sección es el puente entre ambos: **el mapa entero del viaje en una página** — llámenlo brújula, planos, workflow u hoja de ruta — para volver a ella cada vez que se empiece un dominio, se pierda en una fase o se discuta el orden. Primero el workflow completo; luego, fase a fase, qué produce cada una y qué puerta hay que cruzar para avanzar.

```
DOCUMENTOS DISPERSOS
        │
        ▼
[1. DEFINIR]   dominios + píldora + fuentes de la verdad ....... caps. 2-3
        │
        ▼
[2. BRONCE]    capturar → inventariar (manifiesto + hash) ...... caps. 4-5
        │
        ▼
[3. SILVER]    normalizar → curar → segmentar → clasificar ..... caps. 6-9
        │            └── auditoría de fidelidad: 100% o no pasa
        ▼
[4. TROCEAR]   chunks con breadcrumb + doble campo ............. caps. 10-12
        │
        ▼
[5. ENTENDER]  sondas → conflictos → Gold human-guided ......... caps. 13-15
        │
        ▼
[6. VALIDAR]   dataset áureo → suite → Quality Score ≥ 95% ..... caps. 16-17
        │
        ▼
[7. MANTENER]  ciclo de vida + frescura + coste + CD ........... caps. 18-22
        │
        ▼
RACK = Silver + Gold en el mismo índice, listos para el inquilino
```

Y los planos, fase por fase — qué entregable produce cada una y qué puerta hay que cruzar para avanzar a la siguiente:

| Fase | Entregable | Puerta de salida |
|---|---|---|
| 1. Definir | Fichas de dominio + mapa de fuentes con autoridad | 5–10 preguntas reales por dominio; ninguna fuente sin propietario |
| 2. Bronce | Originales intactos + manifiesto con hash | *"Si no está en el manifiesto, no existe"* |
| 3. Silver | Markdown canónico, curado, segmentado, clasificado | Auditoría de fidelidad: veredicto 100% |
| 4. Trocear | Chunks con breadcrumb, resumen y doble campo | Píldora íntegra; tablas sin cortar; un vector por chunk |
| 5. Entender | Conflictos marcados + Gold firmados | Contradicciones < 5%; un Gold por concepto |
| 6. Validar | Dataset áureo ejecutado | Quality Score medio ≥ 95%, sin regresiones, ruido ≤ 30% |
| 7. Mantener | Ciclo de frescura + costes verificados + CD | Suite en verde en cada despliegue |

Tres usos del mapa. **Primera lectura**: en lineal, del prólogo al epílogo — el orden de las partes es el orden del trabajo. **Por proyecto**: el mapa es el plan — cada dominio nuevo repite el viaje completo desde [1], y las puertas de salida son su checklist de avance (la versión operativa, tachable, está en el Apéndice D). **Como diagnóstico**: cuando algo falla, el mapa dice dónde mirar — el síntoma apunta a una fase, y cada fase a sus capítulos.

Y para que nadie la compre esperando otra cosa, lo que este libro **no** es:

> No es un libro de **retrieval ni de rerank**: los algoritmos que ordenan resultados son del inquilino — el Rack solo le entrega los metadatos para que pueda (la label que pondera, el campo de búsqueda en el idioma de las queries, el contenido fiel para el contexto). No es un libro de **prompt engineering**: aquí no hay prompts que perfeccionar, hay datos que preparar. No trata la **evaluación de las respuestas del LLM** — fidelidad, utilidad, redacción: eso es la historia de después. Tampoco de infraestructura de serving, agentes ni UX de chat. Todo eso es **la historia del inquilino, y se cuenta en otro libro.** Este termina donde aquél empieza: en una base de datos sana, verificada y mantenible — lista para que el mejor modelo posible encuentre la mejor verdad posible.

Empecemos por el principio: entender qué estamos construyendo realmente.

---

*Javier Frauca — ZCode*

