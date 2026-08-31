# Capítulo 19: El corpus maestro, el activo oro

## Lo que queda cuando la demo se apaga

Visita mental a un proyecto de conocimiento un año después del lanzamiento. El asistente funciona, los usuarios preguntan, la dirección está contenta. Y ahora la pregunta incómoda: **¿quién es dueño del corpus?** ¿Dónde viven los originales ingeridos, el manifiesto, los textos curados, las fichas de dominio? En demasiados proyectos, la respuesta honesta es: en un almacenamiento del que nadie se acuerda, con un propietario que dejó la empresa, y tratados — en el mejor de los casos — como *andamiaje*: aquello que sirvió para construir el edificio y que se puede tirar, porque el edificio ya está.

Es la inversión de valor más cara de esta industria. Lo que se puede reconstruir en una noche — el índice, el Rack entero — recibe todo el cuidado, la monitorización y el presupuesto. Lo que **no** se puede reconstruir sin repetir meses de arqueología, negociación de permisos y criterio humano — el corpus maestro — queda sin dueño, sin copia de seguridad verificada y sin presupuesto propio. Este capítulo existe para corregir esa inversión: **el master de datos de origen es el ORO del sistema, un activo de primer orden que no se desecha una vez ingestado — se custodia para siempre.**

*(Y aquí la distinción del capítulo 1 cobra su factura: Rack y corpus nunca fueron sinónimos. El corpus maestro es el contenido y su historia; el Rack, la base que lo sirve. Confundirlos es confundir el capital con el escaparate.)*

## La inversión del valor: derivado y capital

La distinción es contable antes que técnica, y el patrón A→B del capítulo 18 la demostró en la práctica: el índice completo se puede reconstruir desde el origen en una noche, porque **el índice es un derivado** — una función del corpus maestro y de una configuración (chunking, embedder, dimensiones). Los derivados se regeneran; no se atesoran.

El corpus maestro, en cambio, es **capital**. Miremos qué contiene y qué costaría reponerlo:

- **Las fuentes en Bronce**: convencido nadie de que re-negociar permisos, re-desenterrar la carpeta de quien se fue o re-reclamar el buzón de `nominas@` sea posible dos veces. Algunas fuentes ya no existirán.
- **El manifiesto**: la memoria institucional del pipeline — qué entró, de dónde, cuándo, con qué decisiones en las notas. Irreemplazable: es la biografía del conocimiento.
- **El Silver**: el resultado de semanas de curado, segmentación, clasificación y auditoría — con todo el **criterio humano** incrustado en las decisiones. Re-hacerlo exige re-contratar las mismas horas de juicio experto.
- **El Gold**: verdades declaradas por expertos con nombre y firma. El activo más delicado y el más lento de producir.
- **Las fichas de dominio, las reglas de etiquetado, los criterios de curado y el dataset áureo**: la **propiedad intelectual del método** — lo que permite que el siguiente ciclo de ingesta sea rutina y no proyecto.

La conclusión contable: **el producto real de todo este método no es el Rack — es el corpus maestro.** El Rack es su consumidor más visible; habrá otros.

## Oro en el balance, Bronce en la medalla

Queda una aparente contradicción que conviene resolver de una vez, porque sonará en cada reunión de gobierno. La Arquitectura Medallón llama **Bronce** a la capa de origen — la materia prima cruda — y **Gold** a la capa de síntesis. ¿Cómo puede entonces el maestro de origen ser "el oro"?

Porque la medalla y el balance miden cosas distintas. La medalla mide **madurez de procesamiento**: cuánto ha refinado el pipeline lo que llega. El balance mide **irreemplazabilidad**: qué no se puede volver a comprar ni regenerar. Y en irreemplazabilidad, el corpus maestro — las fuentes intactas, el manifiesto, el curado acumulado — es el activo más valioso del sistema: es el oro del balance, aunque en la medalla sea Bronce. Dicho de otra forma, y es la frase para la reunión de gobierno: *el Gold del Rack descansa sobre el oro del maestro — y el maestro se custodia aunque el Rack se tire.*

## Custodia: quién lo cuida y cómo

Un activo sin custodia no es un activo: es una esperanza. Las reglas mínimas de custodia del corpus maestro:

- **Propietario con nombre.** El corpus maestro tiene dueño — por definición de este método, el arquitecto del dato o quien herede su papel — y no el "jefe del proyecto" que lo lanzó, porque los proyectos acaban y los activos no.
- **Copias de seguridad verificadas.** No "está respaldado": hay copias, y alguien ha **restaurado una** para comprobarlo. Una copia de seguridad que nunca se ha restaurado es una superstición con almacenamiento.
- **Versionado.** El maestro cambia (lo verá el capítulo 20: cambiarlo es su forma de mantenerse vivo) y cada cambio queda trazado — el manifiesto es, de hecho, el histórico del corpus. El maestro se versiona como el código serio. La implementación natural es literal: **el corpus en Git** (cap. 20) — el remoto como copia verificada, la historia como auditoría, cada cambio con su autor.
- **Auditoría de integridad.** Periódicamente, se re-verifican los hashes del Bronce y se comprueba que no hay huérfanos ni filas sin documento. La integridad se demuestra, no se supone.
- **Protección del acceso.** El maestro contiene *todo* — incluido lo confidencial y lo excluido por RGPD que quedó a un lado del pipeline. Su custodia incluye su cerradura: no es un almacenamiento "de todo el equipo".

Y la consecuencia estratégica, dicha en una frase: **el proyecto RAG puede morir; el corpus maestro, no.** Si el RAG muere y el maestro sobrevive, el siguiente proyecto — con otro modelo, otro framework, otra moda — arranca en el kilómetro cinco. Si el RAG sobrevive y el maestro muere, la próxima actualización de fuentes es una arqueología nueva con presupuesto nuevo.

## Un activo multi-consumidor

El maestro no existe *para* el Rack: existe para todos los consumidores de conocimiento que la organización tenga — hoy y mañana. Hoy, el Rack. Mañana, el grafo de conocimiento del epílogo; los agentes que necesiten contexto verificado; las búsquedas de cumplimiento normativo; el onboarding de nuevos empleados. Cada consumidor nuevo cuesta, esencialmente, una ingesta — **porque el maestro existe**.

De ahí la economía que justifica todo el rigor de este libro: cada hora invertida en el maestro se amortiza entre todos los consumidores presentes y futuros; cada atajo tomado en el maestro lo paga cada consumidor, para siempre. El corpus maestro es, también en esto, el oro: un activo que se aprecia con el uso — cada curado, cada corrección, cada Gold lo hace más valioso — frente al índice, que solo se deprecia.

## Errores frecuentes con el maestro

**El corpus-andamio.** Se construye, se ingesta, se abandona: "ya ha hecho su trabajo". A la primera actualización de fuentes, alguien descubre que el manifiesto está congelado en el mes tres y que nadie sabe dónde viven los originales.

**El índice-como-producto.** Todo el cuidado para el índice — monitorización, alta disponibilidad, presupuesto — y ninguno para el maestro. Es cuidar la foto y descuidar el negativo: la foto no se puede revelar dos veces.

**El maestro repartido.** Los originales en un sitio, el manifiesto en otro, los Silver en la máquina de quien los curó. El maestro es **uno** o no es un maestro: es un puñado de piezas esperando divergir.

**La custodia teórica.** El maestro "está respaldado" en un almacenamiento que nadie ha abierto, con una restauración que nadie ha probado y un propietario que nadie ha designado por escrito. La custodia que no se ha ejercitado no existe.

!!! quote "Regla del capítulo"
    el Rack es el derivado; el corpus maestro, el capital. El primero se reconstruye en una noche; el segundo no se reconstruye — se custodia. El oro del sistema no es el índice: es el corpus que lo alimenta.

