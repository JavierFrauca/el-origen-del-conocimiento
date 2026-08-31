# Capítulo 22: Cómo hacer todo: el pipeline repetible

## La reconstrucción imposible

Mes ocho de un proyecto. El índice vectorial se corrompe en una migración del proveedor — pasa, y volverá a pasar. "No pasa nada", dice el equipo con tranquilidad: "reconstruimos". Y entonces llega el descubrimiento, siempre el mismo: el Silver se curó "con el LLM, en el chat de Fulanito"; los resúmenes los hizo cada uno a su manera, con prompts distintos; la ingesta fue un cuaderno que ya no corre sobre unos datos que ya no están; y las decisiones — qué se cortó, qué se etiquetó, por qué — viven en conversaciones que nadie puede volver a ejecutar. No hay pipeline. Hay recuerdos.

La reconstrucción empieza desde cero y nadie puede garantizar que salga lo mismo. Ni siquiera lo parecido.

Tener la idea no basta. Tener el método tampoco. El método solo es real si **se puede ejecutar — y re-ejecutar**. Este capítulo es el paso de la idea a la operación: cada transición de fase del libro se implementa como un **script** — un proceso repetible, con contrato claro — de modo que se pueda **tirar la base de datos al suelo y reconstruirla desde el origen con un grado de exactitud superior**. Superior, y no solo igual: porque la segunda ejecución corre con las reglas corregidas por lo aprendido la primera — los criterios documentados, los knobs ajustados, las incidencias resueltas. Reconstruir no es repetir el pasado: es regenerarlo mejorado.

## La prueba del fuego: reconstruir desde el origen

El requisito se enuncia en una frase y se examina en una tarde: **sobre el corpus maestro (cap. 19), ejecutando los scripts en orden, se regenera el Rack completo.** Nada vive solo en el índice; nada se transformó sin script; nada importante ocurrió solo en una conversación.

La consecuencia de superar esta prueba alcanza a medio libro. Si se puede reconstruir, el índice es realmente un derivado (cap. 19) y las migraciones son seguras; la frescura es barata (cap. 20), porque actualizar es ejecutar el pipeline sobre un delta; el patrón A→B (cap. 18) es rutina, porque B *es* una reconstrucción en paralelo. Si no se puede reconstruir, todo lo demás del libro es teoría bien redactada: el activo no es reproducible y cada día que pasa lo hace menos.

## Cada transición de fase es un script

El inventario es la propia tabla de contenidos del libro, convertida en procesos. Nombres ilustrativos — lo que importa es el contrato:

| Script | Fase (capítulo) | Lee | Produce |
|---|---|---|---|
| `captura` | Bronce (4–5) | Fuentes del mapa (cap. 3) | Documentos crudos + filas de manifiesto |
| `normaliza` | Canónico (6) | Bronce | Markdown canónico + OCR + imágenes extraídas |
| `cura` | Curado (7) | Canónico | Texto limpio + notas de curado + cola de "dudosos" |
| `segmenta` | Segmentación (8) | Curado | Documentos hijos con linaje al padre |
| `clasifica` | Clasificación (9) | Segmentados | Dominio + facetas + cola de muestreo |
| `audita_fidelidad` | Auditoría (9) | Master canónico + derivados | Veredicto 100% o informe de incidencias |
| `trocea` | Chunking (10–12) | Documentos `AUDITED` | Chunks con breadcrumb, resumen y campo de búsqueda |
| `ingesta_silver` | Ingesta (12) | Chunks | Vectores en el índice; sustitución por `doc_id` |
| `ingesta_gold` | Gold (15) | Síntesis firmadas | UPSERT por `concepto_unico` |
| `valida` | Suite (16–17) | Dataset áureo | Quality Score y veredicto de despliegue |

Diez scripts pequeños cosidos por el manifiesto — no un gran programa. Cada uno lee de una fase y escribe para la siguiente, marca estados, registra lo que hace y deja incidencias donde las hay. El manifiesto es la cinta transportadora: el estado de un documento en cualquier momento dice qué script le toca.

En la implementación de referencia del cap. 20, estos scripts no los ejecuta una persona: los ejecuta la **CD** al integrar cada merge en `master` del repositorio del corpus, sobre el delta que Git entrega de regalo — el `diff` entre el último commit ingestado y HEAD. El repo es el manifiesto editorial; la CD, el manifiesto mecánico.

## El contrato de un script

Cada script del inventario respeta seis condiciones — el equivalente exacto de las puertas de calidad, pero para el código:

1. **Entrada y salida definidas**: de qué fase lee, para qué fase produce, qué estados marca en el manifiesto. Un script sin contrato no es una fase: es un efecto secundario.
2. **Determinista**: mismo corpus de entrada + misma configuración del contrato (chunking, modelos, reglas) → misma salida. Es la condición que hace posible la reconstrucción y la regresión comparativa.
3. **Registra**: todo efecto queda en el manifiesto — documentos tratados, estados, incidencias. Lo que el script hace fuera del manifiesto, no lo ha hecho.
4. **Idempotente**: re-ejecutarlo no duplica — sustituye (cap. 1). El script se puede relanzar sin miedo, que era la frase tranquilizadora del principio del libro.
5. **Tolera fallos parciales**: un documento que falla genera incidencia y el lote continúa; el proceso se reanuda sin repetir lo ya hecho.
6. **No decide por su cuenta**: qué merece procesarse lo decide el alcance y el manifiesto — nunca el script explorando por su cuenta. Su mundo es exactamente el trabajo pendiente que el manifiesto le señala.

## El LLM dentro del script, nunca como script

Y aquí el principio más importante del capítulo, porque la era de los agentes invita a romperlo todos los días: **no se le puede encomendar a un LLM que ejecute una fase.** Decirle al LLM "busca por internet y en la intranet y descárgame todo lo del dominio" para cumplir la captura es entregarle el volante: hoy detecta unas cosas, mañana otras; su cobertura no es verificable, su detección no es repetible, y el corpus que deje — con huecos que nadie ve y duplicados que nadie espera — tampoco. El LLM es probabilístico por naturaleza; un proceso conducido por él se vuelve probabilístico por contagio, y todo lo que este libro ha construido — alcance, manifiesto, idempotencia, verificación — presupone lo contrario: un proceso determinista.

El patrón correcto invierte la dirección: **el script decide el flujo; el LLM procesa piezas dentro de él.** El código determina qué documentos se procesan, en qué orden, con qué instrucción fija y qué pasa con cada resultado; y el LLM se invoca para tareas acotadas, de entrada definida y salida validada: resumir este chunk, revisar este par contra el checklist de la auditoría, proponer etiquetas para este documento, traducir esta descripción, proponer el borrador de este Gold bajo supervisión. En todas ellas el LLM aporta su capacidad donde vale — lenguaje, compresión, síntesis — y su salida atraviesa siempre una validación: la auditoría del cap. 9, el muestreo, la firma del experto. Así el proceso conserva la propiedad que lo convierte en activo — la repetibilidad — y el modelo aporta lo suyo sin apropiarse del volante.

Dicho en una línea para la pared: *el LLM es una herramienta dentro del pipeline; convertirse en pipeline es lo único que un LLM no puede ser.*

## La reconstrucción, paso a paso

Con el inventario y los contratos en pie, la reconstrucción — la prueba del fuego — es un procedimiento aburrido, que es el elogio máximo:

1. Verificar que el corpus maestro está íntegro (hashes del Bronce, manifiesto al día) y que la configuración del contrato está a la vista: chunking, modelos, reglas — cap. 18.
2. Crear el índice nuevo, vacío — en paralelo, patrón *blue-green*: el Rack en servicio no se toca.
3. Ejecutar los scripts en orden sobre todo el corpus. El manifiesto dirá en cada momento cuánto falta.
4. Ejecutar la suite completa sobre el índice reconstruido.
5. Comparar contra la referencia: métricas del dataset áureo contra las de la versión anterior. (El resultado será igual o mejor — las reglas corregidas desde la primera ejecución están en el contrato.)
6. Conmutar. Y guardar la referencia anterior hasta descartar la vuelta atrás.

Es el patrón A→B sin usuario que lo note: lo que en un proyecto sin scripts sería una pesadilla de meses — re-curar a mano, re-decidir lo ya decidido — aquí es una noche de cómputo y una mañana de validación. Esa es la recompensa material del capítulo 19: el maestro más los scripts son el activo; el índice, una noche de máquina.

## Qué no hacer

**El agente-arqueólogo.** "LLM, explora la intranet y descárgate lo que veas del dominio." Ya dicho, merece su nombre: detección no repetible, cobertura inverificable, corpus irreproducible. La captura es crawling programado contra las fuentes del mapa del capítulo 3 — nunca exploración libre.

**El chat como pipeline.** El proceso vivido en conversaciones no existe como activo: no se puede re-ejecutar, ni auditar, ni transferir a quien llegue después. Si no hay script, no hay fase — hay anécdotas.

**El script-monolito.** Un solo proceso de punta a punta: si falla a mitad, no se sabe qué reanudar; sin estados por fase, el manifiesto pierde su tablero; sin etapas, el coste del capítulo 21 no se puede desglosar. Scripts pequeños por transición, cosidos por el manifiesto.

**La ingesta snowflake.** La hecha a mano "solo esta vez", con pasos que nadie documentó. Ese contenido es irrecuperable en la próxima reconstrucción — y toda reconstrucción llega. Si fue necesario hacerlo a mano, lo necesario es convertirlo en script antes de que se olvide cómo fue.

**Scripts que no escriben el manifiesto.** Efectos fuera del libro de registro: duplicados a la re-ejecución, huérfanos sin linaje, documentos que "existen" sin haber sido procesados. Un script sin manifiesto es un desconocido con acceso al edificio.

## Un caso de éxito

El equipo que había seguido el método perdió su índice — migración fallida del proveedor, un martes cualquiera. Escenario de crisis en cualquier otra organización; aquí, procedimiento: el corpus maestro íntegro y verificado, la configuración del contrato a la vista, los diez scripts en cola. Noche de cómputo sobre los 14.000 documentos del corpus; mañana de validación: la suite completa en verde, sin una sola regresión sobre la referencia anterior — mejor, de hecho, porque tres reglas de curado corregidas desde el último ciclo entraron en la reconstrucción. A las 11:00, el Rack reconstruido estaba sirviendo, y nadie de los que preguntaban se enteró de nada.

Lo que fue una noche de cómputo y una mañana de validación habría sido un trimestre de arqueología — y una renegociación de contenido con las fuentes — en un proyecto sin scripts. Esa es la diferencia entre tener un pipeline y tener recuerdos.

!!! quote "Regla del capítulo"
    si no se puede reconstruir desde el origen ejecutando scripts, no hay pipeline: hay recuerdos. El código conduce el flujo; el LLM procesa piezas dentro de él — y nada de lo que importa vive solo en una conversación.

