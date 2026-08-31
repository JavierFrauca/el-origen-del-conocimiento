# Capítulo 3: Mapeo de fuentes de la verdad (Source Discovery & Data Lineage)

## La pregunta inocente

Todo empieza con una pregunta que parece tener respuesta trivial: *"¿dónde están los convenios?"*. "En el SharePoint", contestan. Y es verdad — y es mentira a la vez. Porque cuando el equipo de ingesta abre ese SharePoint encuentra un sitio bonito con veinte documentos, y detrás de él: otro sitio heredado de la fusión de hace seis años, una unidad de red con permisos que solo tiene dos personas, la carpeta personal de quien llevaba "estos temas" antes de irse de la empresa (marcada, literalmente, `PARA BORRAR_NO_ABRIR`), el buzón de `nominas@` con cuatro mil correos, y un PDF legítimo y vigente adjunto a una incidencia cerrada del sistema de tickets.

El mapa no existía. Existía el mito del mapa. Y la primera semana de trabajo real de cualquier proyecto de conocimiento consiste en descubrir que lo que llamábamos "la fuente" era, en realidad, una red de refugios dispersos, cada uno con su propia autoridad, su propio estado de abandono y sus propios fantasmas.

Este capítulo es el trabajo de convertir esa arqueología en cartografía: un mapa de fuentes con autoridad y un alcance documentado, sobre el que el linaje de datos tendrá algo que decir durante toda la vida del sistema.

## El inventario de fuentes: lugares, no documentos

Una confusión frecuente: el mapa de fuentes **no es un listado de documentos** — eso es el manifiesto del capítulo 5, y se construye en la fase siguiente. El mapa de fuentes es un inventario de **lugares** donde puede vivir el conocimiento del dominio, con una valoración de autoridad para cada uno. Documentar una fuente es responder a estas preguntas:

| Campo | Pregunta que responde |
|---|---|
| Identificador y ubicación | ¿Dónde está? (ruta, sitio, buzón, sistema) |
| Propietario | ¿Quién responde si algo no cuadra? |
| Autoridad | ¿Es fuente de verdad, fuente de referencia o mero archivo histórico? |
| Volumen estimado | ¿Cuántos documentos y de qué tamaño? |
| Tipos de contenido | ¿PDF, Word, Excel, escaneados, correos? |
| Frecuencia de cambio | ¿Estático o en actualización constante? |
| Riesgos | Permisos, formatos problemáticos, contenido sensible |

Cada campo se paga a sí mismo. El **volumen** dimensiona la ingesta y anticipa sorpresas ("¿diez mil archivos? la ficha del dominio hablaba de doscientos..."). Los **tipos de contenido** preparan el capítulo 6 — saber antes de empezar que hay cuatro mil escaneados de 2011 cambia el presupuesto de OCR. La **frecuencia de cambio** decide si la ingesta será un evento o una operación continua. Y los **riesgos**, advertidos aquí, son negociables; descubiertos en plena ingesta, son crisis.

## El campo autoridad: la jerarquía que evita el caos

El campo más importante del mapa, y el que más se omite, es la **autoridad**: qué fuente manda cuando dos dicen cosas distintas.

No todas las fuentes valen lo mismo, y la diferencia no es sutil. Para el dominio de convenios, la jerarquía típica sería:

1. **Boletín oficial** — la publicación del convenio: autoridad máxima. Si dice 21%, es 21%.
2. **Repositorio corporativo** — la copia organizada, con la ventaja del contexto interno (qué convenios afectan a qué centros).
3. **Copias de correo y carpetas personales** — valiosísimas a veces (contienen la interpretación interna), pero **nunca** autoridad: son ecos de la verdad, no la verdad.

Mapear fuentes sin jerarquizarlas es preparar el terreno para las contradicciones del capítulo 13: cuando el Rack contenga el 21% del boletín y el 19% del borrador que alguien reenvió por correo en enero, la única pregunta que importa es "¿cuál manda?" — y esa respuesta no se inventa en el momento del conflicto: se escribió aquí, en el mapa, cuando nadie estaba discutiendo nada.

La autoridad también resuelve la tentación contraria: descartar de plano las fuentes bajas. Las copias de correo y las carpetas personales contienen a menudo el conocimiento *aplicado* — la nota de "ojo, que este año cambia la tabla en los centros de Catalunya" que nunca llegó al repositorio. No se excluyen por baja autoridad: se ingieren con esa marca, y la síntesis del capítulo 14 sabrá qué peso darles.

## Nadie conoce el terreno como quien lo trabaja

El mapa de fuentes tiene la misma condición previa que la definición de dominios, y conviene declararla con la misma contundencia: **no se levanta solo.** El arquitecto puede inventariar sistemas y rutas, pero lo que el mapa necesita — dónde vive realmente el conocimiento, cuál copia es la que manda, qué carpeta esconde lo que nadie más sabe — no está en ningún sistema: está en la cabeza de quienes llevan años trabajando ese dominio. **Dejar el ego a un lado y dejarse guiar por los profesionales del dominio no es renunciar al criterio: es la única manera de que el mapa describa el terreno.**

La escena de arranque de este capítulo — "¿dónde están los convenios?" — "en el SharePoint" — es, literalmente, lo que contesta cualquiera. Lo que contesta el experto del dominio es otra cosa: que además del sitio oficial, quien llevaba los convenios de Catalunya guardaba los suyos en su carpeta personal; que cierto buzón recibe las consultas con los anexos más actuales; que tal repositorio quedó congelado desde la fusión y lo vigente vive en otro lugar; que aquel "archivo histórico" es el único sitio donde existe la serie completa. Ese conocimiento no está documentado — está en la experiencia — y solo se obtiene sentándose con quien lo tiene: **sesiones de mapa con los profesionales del dominio**, donde el arquitecto pregunta y el experto señala en el terreno.

De nuevo, el ego. El arquitecto que llega con su rejilla de inventario y la rellena sin escuchar dibuja un mapa coherente y equivocado — coherente, porque su método lo es; equivocado, porque el terreno no lo ha consultado. Dejarse corregir por el experto — "ese sitio está muerto desde la fusión", "esa carpeta es solo copias, la buena es la otra" — es exactamente el trabajo de esta fase: cada corrección de un profesional es un agujero del corpus que nunca llegará a existir.

Y la advertencia de coste, idéntica en espíritu a la del capítulo anterior pero con matiz propio: **un dominio mal definido al menos deja documentos que se pueden re-etiquetar; una recolección mal definida deja documentos que nunca entraron.** La ausencia no se corrige re-etiquetando — solo se descubre, normalmente por un usuario, delante de todos, preguntando algo que el sistema debería saber. El hueco de origen es el peor defecto posible precisamente porque es invisible hasta que duele, y su reparación no es un ajuste: es reabrir la arqueología completa.

Y el mismo efecto colateral valioso que en el acotado: el experto que ayuda a levantar el mapa queda vinculado a las fuentes — será el **heraldo** que avise de sus cambios (cap. 20) y la primera consulta cuando dos documentos discrepen de autoridad (cap. 13). El mapa es, además, un contrato de colaboración con quienes conocen el terreno.

## El alcance: decidir qué NO entra

Definir el alcance es tan importante como definir las fuentes, y es la decisión que casi nadie documenta. El alcance establece, para cada dominio: qué tipos de documento entran, qué periodos temporales, qué formatos y — crucial — **qué queda fuera y por qué**.

Los criterios de exclusión habituales, con sus casos:

- **Redundancia de autoridad.** Si la fuente A replica a la fuente B con menos garantías, B entra y A no. Ejemplo: si el repositorio corporativo ya contiene los convenios del boletín, ingerir además los PDFs que cada delegación se guardó en su carpeta añade ruido, no conocimiento — las copias locales con pequeñas variantes de nombre son la fábrica de duplicados casi-idénticos, los más caros de detectar porque el hash *no* coincide y la similitud *sí*.
- **Caducidad estructural.** Borradores, minutas, versiones en circulación. El dominio trabaja con versiones vigentes o históricas *identificadas como tales* — el borrador de enero del IRPF no era basura: era un documento histórico valioso *mal etiquetado*. Lo que no entra es el documento sin identidad de versión.
- **Contenido no documental.** Presentaciones de veinte diapositivas que dicen en imágenes lo que el documento oficial dice en texto; plantillas vacías; logos con texto alternativo poético. No aportan píldoras.
- **Contenido sensible sin tratamiento definido.** Datos personales o información que el capítulo 9 marcará como excluyente. Si no hay criterio documentado para tratarla, no entra. La frontera del RGPD se cruza con diseño o no se cruza.

Y la regla de oro del alcance, que es también una filosofía de trabajo: **una exclusión documentada es una decisión; una exclusión no documentada es una sorpresa**. Dentro de seis meses, alguien preguntará por qué el sistema "no sabe" de cierto tema — en la peor de las hipótesis, delante de un cliente — y la respuesta debe estar escrita en el alcance, con fecha y firma, no improvisada en una reunión de crisis. La memoria institucional de lo que *no* se hizo es tan valiosa como la de lo que se hizo.

## El linaje de datos: el pasaporte de cada documento

El **linaje** (*data lineage*) es la cadena de custodia del conocimiento: de dónde viene cada documento, quién lo firma y en qué contexto. En sistemas de datos estructurados es una práctica consolidada; en el mundo RAG casi nadie la aplica — y por eso casi nadie puede explicar de dónde sacó el sistema una respuesta.

La metáfora del pasaporte es literal. Cada documento lleva, desde su captura:

- **Origen** — la "nación": la fuente concreta de la que se extrajo. No basta "SharePoint": la ruta completa, el sitio, la biblioteca.
- **Autoridad** — la posición de su fuente en la jerarquía del mapa, heredada.
- **Momento** — cuándo se capturó, cuándo se modificó por última vez en origen.
- **Transformaciones** — qué capas ha atravesado (Bronce → Silver → Gold), cuándo, y con qué versión del proceso.

Para qué sirve, en un caso real de auditoría. Usuario indignado: *"el sistema dice que el plazo de alegaciones es de veinte días y son treinta, esto no vale para nada"*. Con linaje: del chunk de la respuesta se sube al `doc_id`; el manifiesto dice que es hijo del documento `X`, capturado de la ruta `Y` del repositorio, cuya fuente autorizada es el boletín, con captura del 12 de marzo. Se abre el Bronce — byte a byte idéntico al original — y ahí está: el convenio errata corregido dos semanas después por la publicación de un rectificado que nunca entró. Diez minutos para diagnosticar: fuente desincronizada, ingesta del rectificado, caso cerrado con el usuario convencido. Sin linaje: la misma conversación acaba en "el sistema se equivoca" y en la muerte lenta de la confianza.

Este capítulo pone el principio; las fases siguientes lo materializan: **nada se transforma sin dejar rastro**. Cada fase añade información de linaje, nunca la sustituye. El documento Gold del capítulo 14, que sintetiza varias fuentes, es el ejemplo extremo: su valor entero depende de poder señalar con el dedo a cada documento original que lo sustenta.

## El mapa vivo

El mapa de fuentes no es un entregable: es un **artefacto vivo con propietario**. Las fuentes cambian — el SharePoint se migra, el propietario se va, aparece una fuente nueva cuando un departamento entera su disco compartido — y cada cambio pasa por el mapa: alta con su ficha, cambio de autoridad documentado, baja con explicación. La cadencia mínima: revisión del mapa con cada revisión de dominios (cap. 2) y en cada auditoría anual del capítulo 18.

La consecuencia operativa más importante: **ninguna fuente entra al pipeline si no está en el mapa**. Que alguien "suelte un disco compartido" para que el sistema lo ingiera no es una ingesta; es una fuente pidiendo pasar por el capítulo 3. El mapa es la puerta, y la puerta es la diferencia entre un método y una acumulación.

## Errores frecuentes en el mapeo

**La fuente-pantasma.** En el mapa, con volumen, sin propietario. Cuando el OCR falle de forma sistemática o los permisos cambien, no habrá nadie a quien preguntar. Sin propietario no es una fuente: es una caja negra con buen aspecto.

**El barrido universal.** "Ingierimos todo por si acaso" — la negación del alcance. El "por si acaso" tiene coste real: tokens, curado, indexación y, sobre todo, ruido en cada búsqueda futura. Lo barato de no ingerir no lo devuelve nunca nadie.

**El archivo histórico tratado como vigente.** Confundir autoridad con actualidad: el archivo de 2011 es una fuente *legítima para historia*, no para vigencia. Si entra, entra con las facetas correctas (cap. 9), jamás como contenido actual.

**La fuente ilegible.** Aquel sitio con permisos que "algún día resolveremos". La ingesta arrancó sin él, el corpus nació cojo, y el tema que vive ahí es exactamente el que los usuarios preguntan. O se resuelven los permisos antes de empezar, o se documenta la exclusión — pero no se arranca rezando.

!!! quote "Regla del capítulo"
    el mapa de fuentes es una decisión de arquitectura, no un trámite de recopilación. Si una fuente no tiene propietario ni autoridad asignada, todavía no es una fuente — es una caja negra.

