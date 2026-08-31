# Capítulo 6: Normalización canónica y OCR (Canonical Modeling)

## El museo de la informática

Abre un corpus empresarial real en la primera semana y encontrarás una colección completa: PDFs nativos de diez generaciones distintas — unos con texto seleccionable y otros que son una foto de una foto —, Word con maquetas heróicas donde el "título" es un renglón en negrita de 19 puntos, Excel usados como procesador de textos (con celdas fusionadas a tres niveles, porque sí), escaneados de escáner de 2004 y escaneados de cámara de móvil apoyada en el vaso de agua, correos con hilos de diez niveles y adjuntos dentro de adjuntos, y algún que otro `.wps` y `.odt` como fósiles de eras extintas.

Cada formato guarda el mismo texto de maneras incompatibles, y ninguna fase posterior — ni el curado, ni el troceado, ni el embedding — puede trabajar sobre ese zoológico. No por incapacidad genérica: porque cada fase necesita responder preguntas estructurales ("¿esto es un título?", "¿dónde acaba la fila de la tabla?") y cada formato las responde de forma distinta, o no las responde.

La solución es una sola decisión, tomada una vez: **todo se convierte a un único formato intermedio**, el *modelo canónico*, sobre el que operan todas las fases siguientes. La calidad de esa conversación — del zoológico al canónico — es un techo invisible para todo lo demás: ninguna fase posterior puede extraer estructura que la normalización destruyó.

## Por qué Markdown y no TXT

El texto plano es la respuesta instintiva — "al final todo es texto" — y es la elección equivocada. La razón: un documento no es solo texto; es **estructura**. Títulos, subtítulos, listas, tablas, jerarquías. Esa estructura es semánticamente valiosa: indica dónde empieza una idea, qué depende de qué, cómo se organiza una tabla.

El TXT la destruye por completo: el convenio convertido a TXT es una avalancha de líneas donde el artículo 14 y el pie de página tienen idéntica dignidad. El Markdown la conserva en forma legible tanto para humanos como para modelos:

- Los títulos (`#`, `##`) permiten reconstruir la jerarquía del documento — y son la materia prima del *breadcrumb* del capítulo 10, esa línea de contexto que hace autocontenido a cada chunk.
- Las tablas (`| col | col |`) conservan filas y columnas como tal — requisito sin el cual el capítulo 11 entero no existe.
- Las listas mantienen enumeraciones que en TXT quedarían como líneas huérfanas sin relación entre sí.

Por eso el modelo canónico de este libro es **Markdown**: estructura mínima suficiente, legible sin herramientas especializadas, y el formato que mejor entienden los modelos de lenguaje. Hay alternativas más ricas (HTML, JSON estructurado por bloques) y pueden ser el canónico en proyectos avanzados, pero llegan con coste de complejidad que la mayoría de corpus no necesita: Markdown es el punto de equilibrio entre expresividad y frugalidad.

## Parsing documental: separar el contenido del layout

Convertir PDF a Markdown es un problema de traducción, no de copia. El PDF describe *posiciones en una página* — este texto va en estas coordenadas, con este tamaño de fuente — y el Markdown describe *significado estructural* — esto es un subtítulo de segundo nivel. Entre ambos mundos hay que tomar decisiones de interpretación a cada línea: qué es un título y qué es solo texto grande, dónde termina una celda de tabla y empieza otra, qué es una nota al pie y qué es el cuerpo.

Recomendaciones por familia de formato:

- **PDFs nativos**: usar extractores de calidad que reconozcan estructura (títulos, tablas, columnas) y no solo texto. Es tentador economizar aquí — "al final todo es copiar texto" — y es el ahorro más caro del pipeline: un extractor mediocre aplanará las tablas en textos ilegibles y confundirá encabezados con párrafos, y ese daño es **irrecuperable aguas abajo**. El capítulo 10 no puede trocear bien lo que el capítulo 6 aplanó mal.
- **Word y texto estructurado**: conversión directa, respetando estilos — un "Título 2" del original es un `##` en el canónico. Ojo con los documentos donde nadie usó estilos y todo está "a pelo": son los que requieren las reglas heurísticas (tamaño/negrita = título).
- **Excel**: cada hoja con contenido tabular se conserva como tabla Markdown; las hojas que son "texto en celdas grandes" se tratan como texto. Y una regla de oro: si el documento es esencialmente tabular — tablas salariales, calendarios, baremos — es candidato inmediato a documento atómico propio en el capítulo 8, y conviene marcarlo ya.
- **Correo**: extraer cuerpo, remitente, fecha y adjuntos como documentos independientes. El hilo de respuestas anidadas se depura en el curado (cap. 7); aquí solo se desmonta el sobre.

## OCR: cuándo y con qué

Los escaneados e imágenes necesitan **reconocimiento óptico de caracteres**. La decisión no es *si* usar OCR — sin él, ese contenido simplemente no existe para el sistema — sino con qué nivel de exigencia:

- **Escaneados limpios** (texto nítido, una columna, orientación correcta): el OCR clásico (Tesseract y familia) suele bastar, y es rápido y barato.
- **Escaneados hostiles** (varias columnas, tablas, sellos, rotaciones, fotos de documento con sombras): convienen modelos de detección y reconocimiento más capaces, que devuelven estructura además de caracteres. La diferencia de coste se justifica sola cuando el corpus tiene valor: un OCR mediocre sobre un convenio escaneado produce un texto *parecido* al real, con dígitos alterados — y en una tabla salarial, un dígito alterado es una respuesta incorrecta con aspecto de respuesta correcta.

Y una regla que separa los sistemas serios de los juguetes: **el OCR tiene control de calidad**. Tres prácticas mínimas:

1. **Muestrear resultados**: revisar a mano un porcentaje del corpus escaneado comparando original y salida.
2. **Medir el porcentaje de caracteres dudosos**: los motores devuelven confianza por palabra o región; agregable a nivel de documento.
3. **Marcar los documentos por debajo de umbral** en el manifiesto (`notas: revisión OCR pendiente`) y tratarlos como incidencia, no como fondo: un documento mal reconocido no es un documento normalizado, es un documento *parecido* a su original, con errores que las fases siguientes tomarán como verdad y amplificarán.

## Las imágenes dentro de los PDFs: explorarlas, siempre

Un caso que merece sección propia porque es el clásico origen de datos enterrados: los PDFs que **llevan imágenes incrustadas**. Un PDF "nativo con texto" — de los que no parecen necesitar OCR — puede contener, en la página 40, la tabla salarial incrustada como fotografía; el organigrama como captura de pantalla; el gráfico con la evolución de cotizaciones; el párrafo escaneado que alguien pegó en medio del documento; el sello con la fecha de entrada en vigor. El extractor de texto pasa de largo — eso no es texto — y el documento entra al canónico aparentemente completo, con un agujero invisible justo donde estaba el dato que más se va a consultar.

La regla del método no admite excepciones: **toda imagen incrustada se explora**. Ni se descarta por defecto ni se ingiere a ciegas: se mira, se decide y se registra. El protocolo:

1. **Detectar**: durante el parsing, inventariar las imágenes de cada página (los PDFs llevan su inventario de recursos incrustados; los extractores serios lo exponen).
2. **Clasificar**: cada imagen es o **decorativa** (logos, firmas, fondos, separadores de sección) o **portadora de contenido** (tablas fotografadas, capturas con texto, gráficos con datos, párrafos escaneados incrustados en un PDF de texto).
3. **Extraer**: las portadoras de contenido pasan por el mismo pipeline de OCR/extracción que un escaneado — y si contienen tablas, por el tratamiento tabular del capítulo 11. El resultado se integra en el Markdown canónico **en la posición exacta que ocupaba la imagen**, bajo el breadcrumb de ese punto del documento: una tabla fotográfica sin su sección es un dato sin dirección postal.
4. **Registrar**: en el manifiesto, qué imágenes se extrajeron y cuáles se descartaron como decorativas ("tabla incrustada extraída como tabla, pág. 40" / "logotipo descartado"). La decisión, también, deja rastro.

El coste de explorar es una pasada de análisis por documento. El coste de no explorar es el modo enterrado del prólogo fabricado en fase temprana: la tabla existe, el usuario la pregunta, el sistema no la tiene — y nadie sabe que la perdió, porque el PDF "entró completo".

## Normalización de los detalles

Cerrar la fase con la higiene que evita bugs a la larga — cada punto parece menor y cada uno ha costado semanas a alguien:

- **Fechas** a formato único (`YYYY-MM-DD`), en metadatos y en texto cuando sea factible. "1/3/2025" es un dato ambiguo que cambia de valor al cruzar el Atlántico; `2025-03-01` no.
- **Codificación** uniforme (UTF-8). Tildes convertidas en `Ã³` y eñes en `Ã±` son la firma más común de un pipeline sin normalizar — y un veneno sutil para la búsqueda léxica: quien busque "salario" no encontrará "salariÃ³".
- **Nombres de archivo** sin espacios, tildes ni caracteres especiales; el nombre original se conserva en el manifiesto. Los espacios y las eñes en rutas son la causa número uno de pipelines que funcionan en el portátil y fallan en el servidor.
- **Eliminación de artefactos** evidentes de conversión: saltos de página convertidos en texto suelto, cabeceras repetidas pegadas al cuerpo, ligaduras mal resueltas (`fi` convertido en un carácter extraño), guiones de corte de línea al final de línea.

Al terminar, cada documento actualiza su estado a `NORMALIZED` en el manifiesto — con fecha, como todo estado — y los casos problemáticos (OCR dudoso, tablas rotas, PDFs que se resisten) quedan con su nota y su incidencia. El caos de formatos ha muerto; empieza el trabajo sobre el significado.

## Errores frecuentes en la normalización

**Confiar el parsing a "lo que venga por defecto".** El extractor que ya teníamos instalado se aplica a todo, y las tablas — el activo más valioso — son las primeras casualties. El parsing se audita por familia de formato, y las tablas se auditan por separado (son las que peor envejecen).

**El canónico mutable.** Cambiar de opinión sobre el formato canónico a mitad del corpus ("los títulos mejor con `**negrita**` que con `##`") sin re-procesar lo ya convertido: dos dialectos conviviendo en el Silver, y cada fase posterior tropezando con el dialecto que no espera. El canónico se versiona, y cambiar de versión es re-procesar — el Bronce existe para eso.

**El OCR sin muestra de control.** "El OCR dice que salió bien" — sí, con una confianza del 99% sobre un documento equivocado. Sin muestreo humano, la confianza del motor mide la seguridad del motor, no la calidad del resultado.

**La imagen que era la tabla.** Descartar de plano las imágenes incrustadas como "decorativas" sin explorarlas — y con ellas, la tabla salarial fotografía de la página 40, el dato exacto que todos van a buscar. El descarte decorativo también requiere haber mirado: decorativa es la imagen que se examinó y no tenía nada que decir, no la que no se miró.

**Normalizar a medias.** Fechas sin tocar, encoding a medias, "que lo arregle el que lo consuma". Cada detalle sin normalizar reaparece multiplicado: una fecha ambigua en el capítulo 6 es una respuesta incorrecta en producción, ya sin contexto para diagnosticarla.

!!! quote "Regla del capítulo"
    la normalización no es cosmética. Todo lo que no se unifica aquí — estructura, fechas, codificación — reaparece después como error de búsqueda, de troceado o de respuesta, ya sin contexto para diagnosticarlo.

