# Capítulo 4: La zona cruda (Raw Zone) e ingesta inmutable

## La carpeta que alguien ordenó

A las tres semanas de un proyecto, con la ingesta por la mitad, el responsable del repositorio origen tuvo una tarde tranquila y la dedicó a lo que le habían pedido hace meses: **ordenar la carpeta**. Cambió nombres, fusionó subcarpetas, y — con el mejor criterio del mundo — borró trescientos duplicados "que estorbaban". Al día siguiente, la ingesta a medio hacer apuntaba a rutas que ya no existían, la comparación de duplicados daba resultados absurdos y nadie podía afirmar qué se había perdido para siempre en esa limpieza benévola.

El proyecto se salvó por una sola decisión, tomada semanas antes sin aparente urgencia: **la mitad de lo copiado ya estaba en la zona cruda**, byte a byte idéntico a como llegó. Lo perdido en origen se pudo reconstruir casi todo desde el sótano; lo que nunca llegó a copiarse, se re-negoció con la fuente. Sin zona cruda inmutable, aquel jueves hubiese sido el final del proyecto y el origen de una leyenda interna sobre "la IA que no funcionó".

La capa Bronce tiene una definición que cabe en una frase: **el volcado exacto de las fuentes, en un almacenamiento propio, barato e inmutable**. Ni un carácter modificado. Ni un archivo renombrado. Ni un PDF "arreglado". Este capítulo es la defensa razonada de esa aparente redundancia.

## El sótano y sus tres seguros

Acumular lo que luego vas a limpiar parece contraproducente. No lo es: la zona cruda es la póliza de seguro del proyecto entero, y paga en tres coberturas distintas.

**Seguro uno: re-procesar sin volver a la fuente.** El pipeline cambiará. En el mes cuatro cambiarás la estrategia de troceado; en el seis, el modelo de embeddings; en el doce, quizá todo el criterio de curado. Cada cambio exige re-procesar desde el original — y con la zona cruda, eso es un trabajo local sobre almacenamiento propio. Sin ella, cada iteración es una nueva expedición a SharePoint, con sus permisos, su ritmo, su propietario y su riesgo de que aquella carpeta ya no exista. La zona cruda convierte la iteración — inevitable, deseable — en un derecho; sin ella, en una negociación.

**Seguro dos: la auditoría.** Llegará el momento — llega siempre — en que un usuario discuta una respuesta: "esto no puede estar en el convenio". Con Bronce, el original está ahí, byte a byte idéntico a como llegó de la fuente, y la discusión termina en minutos con los datos delante. Sin Bronce, la discusión es *tu palabra contra la memoria del sistema*, y en esos duelos gana quien tiene el original. La trazabilidad no es un lujo regulatorio: es el seguro de vida de la credibilidad.

**Seguro tres: la reversibilidad.** Todo error de transformación es reversible porque el original nunca se tocó. El bug de parsing que rompió las tablas en dos mil documentos es una mala tarde — se re-procesa desde Bronce — en lugar de una pérdida patrimonial. En sistemas sin zona cruda, cada transformación es destructiva por definición, y cada pipeline es un punto sin retorno.

De ahí las dos reglas absolutas que gobiernan esta capa:

1. **Los originales de las fuentes no se tocan.** La ingesta es de lectura; de la fuente solo se copia.
2. **La zona cruda no se toca.** Se escribe una vez, nunca se edita. Las correcciones ocurren en las capas superiores — nunca "arreglando" el original.

Nota la simetría: dos inmutabilidades, una en cada extremo del movimiento de copia. La segunda es la contraintuitiva — el instinto profesional dice "arregla el escaneado torcido" — y es exactamente donde más проектов naufragan: el "arreglo" destruye la evidencia de cómo llegó la información, y con ella la capacidad de distinguir un defecto de fuente (hay que pedir mejor copia) de un defecto de proceso (hay que arreglar el pipeline).

## La estructura de carpetas: referencia, no guía

Un dilema clásico de la ingesta: ¿se conserva la estructura de carpetas de la fuente? Respuesta: **se conserva como referencia, nunca como clasificación**.

La distinción importa porque las carpetas heredadas no son conocimiento organizado — son **arqueología organizacional**. La ruta `Documentos/Varios/Backup_nuevo(2)/Definitivos_ok/` no describe un sistema de conocimiento: describe seis reorganizaciones, tres jubilaciones y dos migraciones. Clasificar por ella sería heredar todas esas cicatrices y llamarlas taxonomía.

La práctica correcta: la ruta original se conserva como **metadato** — en el manifiesto del capítulo 5 y, si aporta, en el nombre del directorio de copia dentro de la zona cruda — pero nadie debe depender de ella para encontrar nada. La clasificación real llega en el capítulo 9, por dominio y facetas: decidida con criterio, no heredada con el mueble.

## El arte de copiar sin molestar

Recopilar miles de documentos de una fuente corporativa tiene sus propias reglas de convivencia — y violarlas tiene un precio que se paga en reputación, no en tecnología.

- **Ventanas y límites de ritmo.** Descargas por lotes, en horarios de baja actividad, con velocidad acotada. Un crawler agresivo puede degradar un SharePoint para toda la organización — y el proyecto de IA que tiró el portal el lunes por la mañana no vuelve a pedir favores. La cortesía técnica es, aquí, gestión política del proyecto.
- **Incremental por diseño.** La primera pasada es completa; las siguientes, solo de cambios — documentos nuevos o modificados desde la última ingesta, detectados por fecha de modificación y confirmados por huella digital. Esto no es una optimización futura: se diseña desde el primer día, porque anticipa el CDC del capítulo 18 y porque la segunda expedición completa nunca debería existir.
- **Tolerancia a fallos parciales.** Un documento que falla — contraseña, permiso puntual, timeout — se **registra con su motivo** y el proceso continúa. La ingesta no es una operación "todo o nada": es un proceso con su propio registro de incidencias, y ese registro alimenta las siguientes pasadas (las incidencias resueltas son las primeras en la cola de reingesta).
- **Respeto de permisos.** Se ingiere lo que la organización autoriza al proyecto, punto. Los permisos de la fuente no son un obstáculo burocrático que "saltar con el usuario admin": son el requisito que demuestra que el proyecto es serio. Un Rack construido sobre documentos a los que no se tenía acceso legal es un Rack con fecha de demolición.

## Qué se ingiere exactamente

Todo lo que el alcance del capítulo 3 admitió, en su **formato nativo**: PDFs nativos y escaneados, Word, Excel, correos exportados, imágenes con contenido documental. Tres precisiones delimitan bien la frontera de esta fase:

**No se convierte nada todavía.** La normalización es del capítulo 6. Mezclar captura y conversión significa que un bug del conversidor contamina el "original" — y ya no hay original.

**No se filtra por calidad aparente.** Un escaneado borroso entra. Un PDF corrupto entra. El descarte de contenido es una decisión documentada del alcance (cap. 3), jamás un efecto colateral de que el crawler no supiera leerlo. La regla práctica: si el sistema no puede con un documento, el documento entra al Bronce igualmente con nota en el manifiesto — "ilegible, pendiente de OCR avanzado" — y es la fase que corresponda la que lo resuelva. Lo que se descarta en la ingesta no se recupera nunca; lo que se ingiere, siempre.

**El correo es un contenedor, no un documento.** Un buzón exportado produce dos clases de material: cuerpos de mensaje (documentos a todos los efectos, con autor, fecha y contexto) y **adjuntos**, que son documentos independientes con vida propia y que se inventarian como tales. El hilo de respuestas anidadas no se depura aquí — es trabajo del curado (cap. 7) — pero los adjuntos se extraen desde el primer momento: detrás de `RE: RE: RV: presupuestos` vive, con frecuencia, el único ejemplar existente de la tabla definitiva.

## Errores frecuentes en la ingesta

**Limpiar en origen antes de copiar.** El antipatrón definitivo, y por eso abre el capítulo con su historia. "Antes de que ingeras eso, ordenamos la carpeta" — el instinto es amable y el efecto es destructivo: nunca se sabrá qué se perdió. Si el origen necesita limpieza, la recibe *después* de la copia completa, y sobre la copia.

**La zona cruda como zona de trabajo.** Empieza con alguien "corrigiendo una errata" en un documento copiado y acaba con un sótano que ya no garantiza nada. Las correcciones viven en las capas superiores; el Bronce es de escritura única. Si alguien necesita tocar un original, la respuesta correcta es: cópialo a Silver y toca la copia — y deja nota en el manifiesto.

**El crawler heroico.** El que vacía el repositorio en un fin de semana "porque se podía". Ya dicho: la velocidad se pacta, las ventanas se negocian y el informe de éxito no es "cuánto descargué" sino "cuánto descargué sin que nadie se enterara".

**Ingerir sin inventariar.** Copiar primero y "el manifiesto lo generamos luego" — y luego se convierte en nunca. El manifiesto (cap. 5) se construye *durante* la ingesta, documento a documento; un documento sin fila en el manifiesto no es un documento capturado: es un desconocido dentro del edificio.

!!! quote "Regla del capítulo"
    la zona cruda es la única parte del sistema que nunca se arrepiente de nada. Todo lo demás puede equivocarse; el Bronce, no.

