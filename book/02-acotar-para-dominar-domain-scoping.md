# Capítulo 2: Acotar para dominar (Domain Scoping)

## La reunión de arranque

Todo proyecto de conocimiento empieza con la misma frase, dicha con el mejor de los ánimos: *"que sepa todo lo de la empresa"*. Se dice en la reunión de arranque, la firma quien patrocina el presupuesto, y suena a ambición sana. Este capítulo existe para defender la tesis contraria — y para dar las herramientas de negociación con las que un arquitecto del dato puede responderla sin sonar a conservador: **un RAG universal no es un RAG potente; es un RAG impreciso con pasos extra**.

## El error de querer saberlo todo: la mecánica de la dilución

La razón por la que "saberlo todo" es mala idea no es filosófica ni organizativa: es **mecánica**. La recuperación vectorial — el mecanismo que casi todo sistema de este tipo usa para encontrar — funciona comparando significados en un espacio matemático donde *todo* el corpus convive. Cada pregunta se convierte en un punto, cada chunk en otro, y la búsqueda devuelve los vecinos más cercanos.

Ahora bien: en ese espacio no hay cartel de "esto es de Legal y esto es de RRHH" — los vecinos son vecinos, sin más. En un corpus universal, la pregunta "¿cuántos días de asuntos propios me quedan?" compite contra el articulado de treinta convenios distintos, contra la política de vacaciones de 2019, contra el manual del ERP que menciona "días" en otro contexto. La similitud superficial sobra por todas partes, y el Top-K — los diez puestos de la búsqueda — es un bien escaso: cada vecino irrelevante que entra, expulsa al relevante que podría haber entrado.

Acotar el corpus es, exactamente, **reducir el espacio de competencia**. Menos pretendientes por puesto, más precisión en cada búsqueda. Es la primera gran sinergia del método: la decisión organizativa del capítulo 2 (qué dominios existen) es también la decisión técnica más rentable de todo el pipeline (cuánto ruido compite en cada búsqueda).

Un matiz que evita malentendidos: acotar en dominios **no es** trocear el sistema en chats separados que no se hablan. Los dominios conviven en un Rack único, etiquetados; el filtrado por dominio es una decisión de consumo que el consumidor aplicará según la pregunta. Lo que el acotado garantiza es que, cuando ese filtro se aplique, cada dominio tenga contenido limpio y específico — y que cuando alguien decida *no* filtrar, al menos las facetas y los Gold declaren las relaciones entre mundos.

## La paradoja de la precisión

En las negociaciones internas, el argumento mecánico convence a los ingenieros; el que convence a las direcciones es otro, y por eso lo llamo la paradoja de la precisión: **menos conocimiento visible, más respuestas correctas**.

El usuario no valora lo mucho que el sistema *contiene* — nadie pregunta "¿cuántos millones de páginas tienes?". Valora lo bien que *responde*, y su percepción se forma en las tres primeras preguntas que le interesa hacer. Un Rack que solo sabe de convenios colectivos, pero que acierta siempre, genera más confianza y más uso que uno universal que acierta a medias. Y la confianza, en estos sistemas, no se recupera: la respuesta errónea número tres mata el proyecto, aunque las cincuenta siguientes hubieran sido perfectas.

La paradoja tiene una lectura estratégica para el arquitecto: **el alcance se vende pequeño y se crece por victorias**. Arrancar con dos dominios bien hechos y expandir cuando la suite de validación lo permita es un plan de crecimiento; arrancar con quince dominios a medio hacer es un plan de cancelación con seis meses de retraso.

## Ontología y taxonomía sin morir en el intento

Estas dos palabras asustan — vienen de la filosofía y de las telecomunicaciones, dos disciplinas capaces de matar cualquier reunión — y no deberían. Para nuestro propósito:

- La **taxonomía** es la lista de dominios: las categorías grandes bajo las que clasificamos el conocimiento. Legal, Finanzas, RRHH, Procesos internos... Pocas, estables, sin solapes.
- La **ontología** es el acuerdo sobre qué cosas existen en cada dominio y cómo se llaman: en el dominio "Convenios" existen *tablas salariales*, *articulado*, *anexos*, *revisiones anuales*; y "salario base" y "sueldo base" son la misma cosa, no dos.

El objetivo no es la elegancia académica: es tener un **vocabulario compartido** que las fases siguientes puedan operar. La ontología alimenta las reglas de etiquetado del capítulo 9, las queries sonda del capítulo 13 y las preguntas del dataset áureo del capítulo 16. Un vocabulario sin definir se paga tres veces: en clasificación inconsistente, en búsquedas que no encuentran y en validación que no puede ser estricta.

Reglas prácticas para no morir en el intento:

1. **Empieza con 3–7 dominios.** Si la lista natural sale con veinte, no tienes dominios: tienes etiquetas. Los detalles van en las facetas del capítulo 9 (`naturaleza`, `año`, `vigencia`), no en la taxonomía.
2. **Los dominios no se solapan.** Si un documento "podría ser de dos" — la famosa tabla salarial que interesa a RRHH y a Finanzas — la decisión se toma una vez, se documenta en la ficha y vale para siempre. El solape entre dominios no lo resuelve el dominio: lo resuelven las facetas multi-asignables y, en el límite, el documento Gold de la Parte V, que es precisamente el especialista en conceptos que viven en varios mundos.
3. **Los dominios se describen, no solo se nombran.** Un dominio sin descripción es una etiqueta muerta: nadie sabe qué mete y qué no, y a los seis meses hay tres criterios conviviendo.
4. **Los nombres son estables y en minúsculas.** Renombrar dominios es re-etiquetar el corpus entero. `rrhh`, no "Recursos Humanos (Personal) (antiguo: Personal y Nóminas)".

## Plantilla de definición de dominio

Cada dominio se formaliza con una ficha como esta — y conviene insistir: **el dominio no existe hasta que la ficha está completa**. La ficha no es burocracia; es la fuente de la que beberán media docena de fases posteriores.

| Campo | Descripción | Ejemplo: "Convenios Colectivos" |
|---|---|---|
| **Nombre** | Identificador estable del dominio | `convenios` |
| **Propósito** | Para qué sirve este dominio, en una frase | Consulta de condiciones laborales sectoriales |
| **Preguntas que debe responder** | 5–10 preguntas reales, con la redacción que usaría un usuario | "¿Cuál es el salario base de un oficial de primera en 2025?", "¿Cuántos días de asuntos propios?" |
| **Preguntas que NO responde** | Frontera explícita, con destino | "¿Cómo se contabiliza la nómina?" (→ Finanzas) |
| **Fuentes que lo sustentan** | Orígenes autorizados, con su jerarquía | Boletín oficial > repositorio interno > correo |
| **Vigencia** | Cómo caduca el conocimiento del dominio | Revisión anual; la anterior queda `deprecada` |
| **Píldora típica** | La unidad de respuesta característica | Una tabla salarial; un artículo del articulado |
| **Idioma de consulta** | Idioma(s) en que pregunta el usuario; fija el idioma objetivo del campo de búsqueda (cap. 12) | Español |
| **Propietario** | Quién responde dudas sobre el dominio | — |

Tres filas merecen comentario.

**Las preguntas que NO responde** son las más incómodas de escribir y las más útiles. Definen la frontera, y una frontera sin destino — "eso no lo sabremos" — es una queja futura; una frontera con destino — "eso lo responde Finanzas" — es un enrutamiento. Cuando el usuario pregunte lo que el dominio no sabe, el consumidor del Rack tendrá dónde decirle a dónde ir.

**Las fuentes con jerarquía** anticipan el capítulo 13: cuando dos documentos del dominio discrepen, la jerarquía de fuentes es la primera pista de arbitraje. El boletín oficial manda sobre la copia interna; la política del portal manda sobre el PDF que circuló por correo.

**La píldora típica** enlaza con el concepto del siguiente apartado — el que sostiene media librería.

## El perfil de consulta y la píldora de información

Y llegamos al concepto central del capítulo — aportación de este método, y la respuesta correcta a la pregunta más técnica que se hará sobre él: *"¿de qué tamaño hago los chunks?"*.

Al acotar un dominio no solo definimos *qué* sabe. Definimos **a qué granularidad pregunta el usuario**. Y esa granularidad determina la **píldora de información**: la unidad mínima autocontenida que responde a una pregunta típica del dominio.

- En un dominio de **consultas frecuentes**, la píldora es pequeña: una pregunta y su respuesta, dos o tres líneas que viven solas.
- En un dominio de **normativa densa**, la píldora es grande: una tabla salarial completa, un artículo legal con sus apartados, una definición con sus excepciones. Responder "¿cuánto cobra un oficial de primera?" exige la fila *y* su encabezado *y* la vigencia — la píldora es la fila contextualizada, no un número suelto.

La píldora no es un detalle técnico: es la decisión de diseño que **heredan las fases siguientes**. La segmentación del capítulo 8 la materializa (separa los temas que no deben convivir en un documento). El troceado del capítulo 10 la respeta — no parte una píldora, no fusiona píldoras ajenas — y calibra su presupuesto de tokens a su tamaño. El doble campo del capítulo 12 la describe. Y la validación del capítulo 17 comprueba, pregunta a pregunta, si las píldoras llegan a destino.

Si el tamaño del chunk te parece hoy un número arbitrario que se decide con un tutorial delante, es porque aún no has escrito la ficha del dominio. Después de escribirla, deja de ser una elección: es una consecuencia.

### Cómo se descubre el perfil de consulta

La granularidad no se inventa: se **observa**. Fuentes de verdad para reconstruirla:

- **Tickets y consultas históricas**: si la organización tiene mesa de ayuda de RRHH o Legal, sus últimas cien consultas son la mejor muestra de granularidad real que existe.
- **Las FAQ existentes**, si las hay — no por su contenido, sino por su *formato*: qué preguntan y a qué nivel de detalle responden.
- **Entrevistas de quince minutos** con tres o cuatro usuarios representativos: "si tuvieras un sistema que respondiera sobre X, ¿qué le preguntarías mañana mismo?" Sus cinco primeras preguntas son el núcleo de la ficha.

El perfil que sale de ahí es siempre más humilde y más útil que el que se inventa en la reunión de arranque: nadie pregunta "¿cuál es el marco normativo íntegro del convenio?"; preguntan cuánto cobra un auxiliar.

## Los expertos del dominio: la ficha no se escribe solo

Hay un requisito previo a todo lo anterior que conviene declarar sin rodeos, porque determina el éxito de la fase entera: **la definición de dominios no se hace solo.** El arquitecto aporta el método — la estructura de la ficha, las reglas del acotado, el criterio de granularidad — pero el contenido de la ficha no sale de su cabeza: sale de la experiencia de quienes llevan años trabajando en ese dominio. **Dejar el ego a un lado y dejarse aconsejar por los profesionales del dominio no es una cortesía: es un requisito de la fase.**

La razón es la misma que recorre todo el libro desde el prólogo — el coste asimétrico de los errores tempranos. Una mala definición de dominios no es un error local: es una **decisión que se propaga en cascada**. Dominios mal partidos → documentos clasificados mal → segmentación que no respeta los temas reales → chunks mezclados → búsquedas que devuelven mezclas → respuestas que nadie puede fiar. Y corregir no significa retocar la ficha: significa **re-clasificar el corpus entero** — re-ingesta, re-validación, dataset áureo rehecho. Si partimos de una mala definición, difícilmente podremos corregirla después sin pagar el corpus completo. La definición inicial es, con diferencia, la decisión más barata de tomar bien y la más cara de tomar mal.

Por eso el método exige que la ficha de dominio se construya **en taller con los expertos**: sesiones de trabajo donde el arquitecto pregunta y el experto dice — qué temas existen, cómo se agrupan de verdad (no en teoría), qué preguntas llegan cada semana, dónde termina un tema y empieza otro, qué matices separan dos temas que a un ojo externo parecen el mismo. **El arquitecto pregunta; el experto sabe.** Y cuando el experto corrige una frontera que al arquitecto le parecía elegante, la respuesta correcta no es defenderla: es anotarla y agradecerla — cada corrección temprana de un experto es un re-trabajo futuro que no ocurrirá.

Y hay un efecto colateral valioso: el experto que ayuda a definir el dominio queda enganchado a él. Será el **propietario** de la ficha, el validador del etiquetado (cap. 9), el autor de los documentos Gold (cap. 14) y el curador que vigila la frescura (cap. 20). La fase de acotado es, sin quererlo, la fase de reclutamiento del sistema entero — y empieza, como debe empezar toda buena arquitectura, preguntando en vez de afirmar.

## Errores frecuentes en el acotado

Para cerrar, los cinco antipatrones que más veces he visto pagar:

**El dominio-orgánico.** Definir los dominios copiando el organigrama: "Catalunya", "Zona Norte", "Dirección Comercial". El organigrama cambia con cada reorganización; el conocimiento no sigue al organigrama. Los dominios son de conocimiento, no de estructura.

**El dominio-trámite.** "Varios", "Documentación general", "Otros". Un dominio que existe para clasificar lo que no encaja es la basura con nombre propio — y todo lo que entra ahí está perdido. Si algo no encaja, o falta un dominio o el contenido no entra.

**El dominio-proyecto.** "Convenio 2025", "Proceso de fusión". Lo que hoy es un proyecto es mañana contenido de un dominio. `convenios` con la faceta `año: 2025` sobrevive a la siguiente revisión; "Convenio 2025" como dominio, no.

**El dominio omnisciente de vuelta.** Y el más traicionero, porque llega disfrazado de pragmatismo: " vale, pero mete esto también" — y de vuelta, en seis meses, el dominio Legal contiene también las comunicaciones comerciales y las actas del consejo. La ficha tiene frontera por algo: la expansión de un dominio es una decisión con ficha nueva, no una acumulación.

**El arquitecto en solitario.** Definir los dominios desde el escritorio, con el organigrama y una intuición, sin sentarse nunca con quien vive el dominio. La ficha resultante tiene buena pinta y mal contenido — y cada error de partida se multiplica por todo el corpus, en cada fase posterior. Es el antipatrón que habilita a los otros cuatro.

!!! quote "Regla del capítulo"
    si no puedes listar 5–10 preguntas reales que el dominio debe responder, no tienes un dominio — tienes una intuición. Y si no sabes a qué granularidad pregunta el usuario, no tienes un dominio — tienes una carpeta.

