# Capítulo 7: El arte del curado (Data Cleansing)

## La economía del token

Antes de la técnica, la aritmética que la justifica. El contexto de un modelo de lenguaje es un recurso **finito y caro**: cada palabra que entra compite por el mismo presupuesto. Y en un corpus empresarial típico, la proporción de envoltorio es humillante: entre pies de página repetidos en cada página, cabeceras corporativas, avisos legales idénticos en treinta documentos, numeraciones residuales y muletillas, un PDF de veinte páginas fácilmente contiene ocho de paja. Ingerido tal cual, ese documento ocupa el doble de índice de lo necesario, produce el doble de chunks, y — lo peor — cada chunk de contexto lleva dentro su ración de ruido que compite por ser "parecido" a las preguntas.

Pero el curado no se justifica solo por ahorro — se justifica por **verdad**. La paja no solo ocupa: confunde. Un pie de página legal repetido en cada página del convenio produce chunks que parecen "hablar de jurisdicción" cuando hablan de salario; un hilo de correo con diez niveles de reenvío hace que la respuesta de 2019 aparezca como vigente junto a la de 2025. Curar bien no es hacer bonito: es **definir qué es información y qué es decoración**, antes de que el sistema decida por su cuenta — y decida mal.

## La paja semántica: definición operativa

Todo documento real lleva envoltorio. Y el oficio del curado empieza por una distinción que debe quedar escrita como criterio, no como intuición:

- **Paja**: contenido que se repite mecánicamente, no aporta significado y sobra en cualquier fragmento. Numeración de página, cabeceras corporativas, pies legales idénticos, marcas de agua, muletillas de transición vacías.
- **Información**: todo lo que responde a una pregunta posible del dominio. Puede parecer accesorio — una cláusula de entrada en vigor, una excepción al final de un apartado, una nota de pie con la fuente oficial — y ser exactamente lo que alguien pregunte.

El criterio operativo que evita arbitrariedades: **si una pieza de texto podría ser objeto de una query sonda, no es paja**. "¿Cuándo entra en vigor?" es una pregunta real; la cláusula que la responde, jamás se corta. "¿Qué dijo el legislador exactamente?" también; la nota de pie con la referencia oficial, tampoco. Cuando dudes, no se corta — el coste de una línea de paja superviviente es marginal; el de una información eliminada, una respuesta que el sistema no podrá dar jamás, porque el dato ya no existe en ninguna capa posterior.

## Casos concretos, con su criterio

- **Cabeceras y pies repetidos**: se eliminan en todas las páginas, **conservando la primera aparición** si contiene identidad del documento — el organismo emisor, la referencia oficial, la fecha de publicación son contexto que el breadcrumb y el resumen aprovecharán.
- **Saltos de página dentro de párrafos**: se unen. Un párrafo partido por una página es un único párrafo; si además hay cabecera repetida en el corte, se retira antes de unir.
- **Numeración estructural**: se conserva cuando es semántica — "Artículo 14.3", "Apartado 2.b" es contenido que el breadcrumb y las citas necesitan — y se elimina cuando es residual: los números de página sueltos, la paginación del original convertida en líneas huérfanas.
- **Hilos de correo**: se depuran las respuestas anidadas repetidas (el clásico árbol de `RE: RE: RV:` donde la misma firma aparece once veces) y se conserva la **secuencia de opiniones distintas con su autor y fecha**, si el dominio la aprovecha. El hilo es, a menudo, la historia de una decisión: la secuencia importa; la repetición, no.
- **Caracteres basura**: restos de encoding, ligaduras rotas, símbolos de conversión, espacios dobles. El capítulo 6 hizo la pasada gruesa; esta es la fina, la que se ve leyendo el texto en lugar de inspeccionando bytes.
- **Muletillas y referencias rotas**: "como se ha comentado anteriormente" que remiten a un apartado que la segmentación (cap. 8) separará. Cuando la referencia apunta a contenido que seguirá en el mismo documento, se conserva; cuando apunta a otro tema, es señal temprana de que esos dos temas quizá no debían convivir — y alimenta la decisión del capítulo siguiente.

## Las tablas: el patrimonio a proteger

Si hay un error de curado imperdonable, es dañar una tabla. Las tablas salariales, los porcentajes de cotización, los calendarios, los baremos — son la respuesta exacta a las preguntas más frecuentes y las estructuras más frágiles del pipeline entero.

Reglas del curado respecto a tablas:

- **Se conservan íntegras.** Ninguna fila se descarta por "parecer redundante", ninguna celda se vacía por estar mal alineada en el original. La fila que parece un duplicado suele ser la revisión semestral — y borrarla es fabricar una respuesta antigua con aspecto de nueva.
- **Se conservan sus encabezados junto a cada fragmento**, aunque eso repita texto. Una fila sin encabezado es un número sin unidad: "1.890" no dice ni siquiera si son euros o días.
- **No se "explican".** El curado no convierte la tabla en prosa. La descripción de la tabla es trabajo del capítulo 11, y se **añade**, no se sustituye — el curador que "resume la tabla en un párrafo para que se entienda" ha destruido el dato para ahorrar una lectura.
- **La tabla rota por el parsing** (cap. 6) se marca para revisión manual en el manifiesto. Repararla mal es peor que repararla tarde: una tabla recompuesta con columnas desplazadas produce respuestas numéricas incorrectas con total seguridad — el peor producto posible de este sistema.

## La regla de oro del curado

Todo lo anterior cabe en un principio con dos mitades igualmente importantes:

> **Se elimina todo lo que sobra. Se conserva el 100% de lo que informa.**

La primera mitad da limpieza; la segunda, fiabilidad. Y conviene asumir la asimetría de sus fallos: un curado flojo en la primera mitad produce un sistema **gordo** — ineficiente, con ruido, pero honesto. Un curado flojo en la segunda produce un sistema **limpio y erróneo** — la peor combinación posible, porque nadie desconfía de un sistema limpio. Cuando haya que decidir bajo duda, la duda siempre favorece a la conservación.

## Trazabilidad del curado

Cada documento curado actualiza su estado a `CLEANED` en el manifiesto, y los casos no triviales dejan nota: qué se eliminó y por qué, qué se marcó para revisión, qué criterio dudoso se aplicó. No hay que narrar lo obvio — quitar veinte pies de página idénticos no merece párrafo — pero sí todo lo que un lector futuro podría preguntar.

¿Por qué tanto rastro por unas comas eliminadas? Porque el curado es la fase con mayor componente de criterio humano de todo el Bronce-Silver, y por tanto la que más preguntas genera a posteriori. Un año después, cuando alguien compare el Silver con el Bronce y eche en falta un párrafo, la nota de curado es la diferencia entre una decisión documentada ("se eliminó el 14-03 por duplicado del apartado 3.2") y un agujero negro. Y hay una segunda razón, menos evidente: las notas de curado son la **materia prima de las reglas del futuro** — cada decisión manual que se repite tres veces es un candidato a regla automática, pero solo si quedó escrita.

## Errores frecuentes en el curado

**El curado entusiasta.** El revisor que "mejora" el texto: reescribe frases, unifica estilo, corrige lo que el original decía mal. Fatal: el Silver no es una edición mejorada del original — es una copia limpia del original. Si el original dice 19%, el Silver dice 19%, aunque el curador sepa que es 21% (el conflicto se marca, cap. 13; no se "corrige" — corregir sin traza es fabricar una verdad sin fuente).

**El curado a ciegas.** Aplicar reglas globales sin mirar el resultado: la regla de "eliminar líneas que empiezan por número" se come la numeración estructural del articulado legal. Toda regla de curado se muestrea sobre documentos reales del propio corpus antes de generalizarse.

**El curado informatizado al 100%.** Creer que todo se automatiza. Se automatiza lo mecánico (pies repetidos, encoding); se decide a mano lo dudoso (hilos de correo, qué es paja en un dominio nuevo). El 70-80% automático es un buen objetivo; el 100% es una bandera roja.

**El curado que no consulta al dominio.** Decidir qué es paja en convenios colectivos sin hablar con quien trabaja con convenios. La persona que lleva diez años consultando convenios sabe en dos minutos qué partes nunca nadie mira — y cuál es la tabla exacta que lo consultan todo el tiempo. La entrevista de treinta minutos con un experto del dominio es el mejor revisor de criterios que existe.

!!! quote "Regla del capítulo"
    curar no es leer — es transformar con criterio y dejar rastro. El objetivo no es que el documento quede bonito: es que cada token que llegue al Rack valga su sitio en el contexto del modelo.

