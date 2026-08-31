# Capítulo 16: El dataset áureo de validación (Golden Dataset)

## El examen de acceso

Todo lo construido hasta aquí es una promesa: dominios bien acotados, Silver limpio, Gold trazable. Promesas formuladas con la mejor voluntad del mundo — y gestionadas, hasta ahora, con criterio experto y buena fe. El dataset áureo es donde la promesa se examina: un conjunto de **preguntas del mundo real** con, para cada una, la **lista de chunks que deben aparecer** en la respuesta de búsqueda. Convierte "el Rack funciona bien" — una opinión — en una medida.

Su filosofía es idéntica a la de las pruebas de aceptación del software serio: no se pregunta al sistema si *cree* que funciona; se le presentan casos conocidos y se comprueba el resultado. Y su regla fundamental separa este libro del RAG: **la validación es de recuperación, no de generación** — se comprueba qué devuelve la búsqueda, sin LLM interpretando nada. El Rack entrega conocimiento; que el modelo lo use bien es el problema del inquilino. Este capítulo certifica el primero.

## La anatomía de un registro

Cada entrada del dataset es una fila como esta — dos de llenas, para ver el espíritu:

| Campo | Ejemplo 1 | Ejemplo 2 |
|---|---|---|
| `query` | "¿Cuál es el salario base de un oficial de primera en 2025?" | "¿Cómo se calcula el IRPF 2025 para autónomos?" |
| `chunks_esperados` | `silver_tablas_003` | `gold_001`, `silver_legal_045`, `silver_finanzas_123` |
| `top_k` | 10 | 10 |
| `tipo` | `tabla-datos` | `concepto-cruzado` |
| `origen` | "Ficha del dominio convenios, pregunta 2" | "Sonda 12 de la auditoría (cap. 13)" |
| `razón` | "Contiene la tabla 2025 íntegra con su descripción" | "El Gold del concepto debe ganar a los hechos sueltos" |

Dos propiedades separan un buen dataset de una lista de preguntas:

**La pregunta debe ser real** — redactada como pregunta un usuario, no como adivinanza pensada para hacer coincidir con un chunk. "¿Cuál es el salario base de un oficial de primera en 2025?" es una pregunta real; "¿en qué chunk está la tabla de sueldos?" es una pregunta de laboratorio que nadie hará jamás, y que solo prueba que el índice sabe encontrarse a sí mismo.

**Los `chunks_esperados` se definen mirando el contenido, no el comportamiento actual del índice.** Lo que *debe* aparecer, decidido leyendo las fuentes — no lo que *suele* aparecer, copiado de una búsqueda de ejemplo. Si el esperado se copia de la búsqueda, el dataset valida el status quo y es estructuralmente incapaz de decir "esto está mal".

## Cuántas y cuáles

**Cincuenta es el número de partida**, por una razón práctica antes que teórica: menos de treinta no detectan regresiones con fiabilidad (un fallo mueve la media demasiado y una coincidencia disfraza una rotura); más de cien cuesta mantenerlas al día y — lo que es peor — deja de hacerse. La suite que corre antes de cada despliegue tiene que ser realista de mantener; cincuenta bien elegidas lo son.

El criterio de composición importa más que la cantidad, y se organiza con una **matriz de cobertura**: dominios en filas, tipos de píldora en columnas — hecho tabular, conceptual, de vigencia, de concepto cruzado — y al menos una query por celda que aplique. La matriz es el control de calidad del propio dataset: una celda vacía no significa "no importa", significa "nadie comprobará ese tipo de contenido en ese dominio". Sobre la matriz, la mezcla mínima:

- **Cobertura de dominios**: cada dominio de la ficha del capítulo 2, proporcionalmente a su peso y criticidad.
- **Cobertura de tipos**: preguntas tabulares, conceptuales, de vigencia y de cruce — las cuatro maneras en que el Rack puede fallar.
- **Mezcla Silver y Gold**: las preguntas de hechos deben encontrar Silver; las de conceptos cruzados o conflictos, **el Gold correspondiente**. Que "¿cómo se calcula el IRPF?" recupere el documento Gold es parte del examen — es la prueba de que la capa de autoridad funciona como capa.
- **Las dificultades conocidas**: las preguntas donde la auditoría del capítulo 13 encontró problemas — dispersión, contradicción, enterramiento — entran en el dataset. Son la memoria institucional de los errores ya sufridos; el dataset es la manera de que un error pagado una vez se pague nunca más.

## Criterios para elegir los chunks esperados

- **Concretos, no aproximados**: `doc_id` y `num_chunk` exactos, no "algo de la ley del IRPF". La métrica del capítulo siguiente compara identificadores; la vaguedad aquí es ruido allí.
- **Suficientes, no exhaustivos**: 2–4 chunks esperados por pregunta. Si se listan diez y solo deben aparecer "algunos", la métrica se diluye hasta la inutilidad — el dataset es una prueba, no un inventario.
- **Con su Gold cuando existe**: si el concepto tiene síntesis, el Gold está en la lista — su presencia es un requisito, no un bonus. Y con vigencia correcta: los esperados de una pregunta sobre normativa vigente son los `vigente`; que el sistema devuelva el derogado es un fallo aunque el contenido "se parezca" — de hecho, es el modo fantasma completo, que el dataset está precisamente ahí para atrapar.
- **Con su razón anotada**: una línea en el registro de por qué ese chunk debe salir ("contiene la tabla 2025"; "es el Gold del concepto"). Dentro de un año, cuando se discuta una fila del dataset, la razón evita re-escribir la historia.

## Cómo se construye el primero: un taller de tres horas

La primera vez, la construcción tiene un método sencillo que evita tanto la página en blanco como el exceso:

1. **Sacar el material bruto** (30 min): las preguntas de las fichas de dominio (cap. 2), las sondas de la auditoría (cap. 13), los tickets históricos si existen. Normalmente hay materia para sesenta preguntas; sobra.
2. **Repartir por la matriz de cobertura** (45 min): colocar las preguntas en su celda dominio × tipo, ver los huecos, y redactar las queries que faltan para completar el tablero.
3. **Definir esperados leyendo las fuentes** (60 min): para cada pregunta, abrir los documentos y decidir — con `doc_id` y `num_chunk` — qué debe salir. Es el paso más lento y el que no se delega.
4. **Revisión cruzada** (45 min): otra persona lee el dataset entero preguntándose por cada fila: ¿la pregunta es real? ¿los esperados son los correctos? ¿la razón está anotada?

Al final: cincuenta filas que ya son el contrato del Rack — y el subproducto más valioso del taller, que nadie prevé, es que los cuatro participantes acaban conociendo el corpus mejor que nadie en la organización.

## De dónde no sale el dataset

Tres orígenes prohibidos, por distintos motivos:

- **No se genera con el propio índice** ("lanzo preguntas y marco lo que sale"): eso validaría el status quo, no la calidad. Los esperados nacen del contenido y de la auditoría, nunca de la búsqueda. Es el error más común en los primeros datasets y el que invalida todo lo demás.
- **No se delega sin revisión**: un LLM puede proponer borradores de preguntas desde la ficha de dominio — y ayuda a cubrir ángulos que el equipo no ve — pero cada registro lo valida una persona que conozca el corpus. El dataset es el patrón oro: un patrón oro con errores es peor que ninguno, porque se le obedece.
- **No se congela para siempre**: el dataset vive con el Rack. Cada hallazgo importante de producción — la pregunta que un usuario hizo y el sistema falló — termina, tarde o temprano, siendo una fila nueva. El dataset que no crece con los incidentes es un examen que ya se sabe de memoria.

## El dataset también se audita

Y para cerrar, un espejo: el dataset exige para el Rack el rigor que él mismo debe soportar. Una **auditoría periódica del dataset** — trimestral encaja bien — revisa:

- ¿Sigue cada pregunta siendo real y vigente? (Una pregunta sobre "la política vigente" puede haber quedado obsoleta por el propio negocio.)
- ¿Siguen siendo correctos los esperados? (Un Gold actualizado puede haber cambiado de contenido; un documento deprecado no puede seguir siendo "esperado" para una pregunta vigente.)
- ¿La matriz de cobertura sigue cubierta? (El dominio nuevo de hace tres meses, ¿ya tiene sus filas?)
- ¿Las razones están al día?

Un dataset sin auditoría envejece como el Rack que examina — y un examen caducado es más peligroso que ningún examen, porque se aprueba con la misma seguridad que el primero.

## Dónde vive

Es un artefacto de primera clase del proyecto, versionado junto al manifiesto, con su propio control de cambios — quién añadió cada fila, cuándo y por qué. Cuando el capítulo 18 hable de "ejecutar la suite antes de cada despliegue", este es el activo que se ejecuta; cuando el capítulo 17 ponga números al resultado, esta es la muestra sobre la que se calculan. Y cuando alguien discuta si el Rack "está bien", la respuesta comienza siempre en el mismo sitio: *"¿qué dice el dataset?"*

!!! quote "Regla del capítulo"
    el dataset áureo es el contrato de nivel de servicio del Rack, escrito en preguntas. Si no está en el dataset, el despliegue no garantiza que funcione; si está en el dataset y falla, no se despliega.

