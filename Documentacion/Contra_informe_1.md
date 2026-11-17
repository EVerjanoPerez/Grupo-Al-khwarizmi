# Contrainforme primera entrega

En la primera entrega se obtuvo un 8,86/10. Se perdieron 1,14 puntos. 

En primer lugar hemos analizado las partes afectadas en una tabla:

| Apartado | Nota | Máximo | Perdido | Criterios con observación negativa |
|---|---|---|---|---|
| A. Requisitos | 2,67 | 3,15 | 0,48 | A3, A4, A5 |
| B. Diagrama UML | 1,89 | 2,25 | 0,36 | B2, B3 |
| C. Especificaciones | 2,25 | 2,25 | 0 | Ninguno |
| D. Scrum | 1,10 | 1,35 | 0,25 | D2, D3, D4 |
| E. Presentación | 0,95 | 1 | 0,05 | Exceso de tiempo |

Debido a la espera de este feedback, los ficheros que se han visto comprometidos no se han modificado.

## A. Requisitos

Hemos realizado un análisis de los puntos señalados por el corrector, a continuación se exponen las diferentes justificaciones a esos mismos puntos:

### A3. Calidad de los requisitos funcionales

En este punto se ha hecho hincapié en desglosar los requisitos funcionales 35 y 1, en estos casos concretos:

- FR1: todas sus partes se refieren a un único objetivo (acceso seguro a la sesión) y comparten actor y contexto. Fue una fusión deliberada del 14/10 para reducir dispersión.

- FR35: el consentimiento no es una obligación independiente, es la condición de uso del canal WhatsApp (“si el contacto lo ha consentido”). Separarlo crearía un requisito que no tiene sentido sin el canal. Las obligaciones independientes de FR35 ya están en FR35.1–FR35.3.

Aunque reconocemos que en el caso de FR1 se podría desglosar, por tanto para lo siguiente entrega estará desglosado.

### A4. Requisitos no funcionales verificables.

Hacemos hincapié en los siguientes puntos: ningún NFR indica método de verificación. NFR12 (“miles de clientes”) y NFR13 (“prácticamente inmediata”) no están cuantificados; NFR2 (“minimalista”) es subjetivo. Reconoce NFR1 (1 minuto) y NFR10 (idempotencia).

Aquí hay varios puntos a tratar, primero reconocer que NFR12 y NFR13 sí podrían cuantificarse aunque fuera de forma orientativa, NFR2 es subjetivo.

Sin embargo, se pueden exponer varios puntos para rebajar el descuento: 
1. La plantilla no exigía método de verificación; se penaliza un campo que no se pidió.
2. Estamos en fase sin arquitectura ni tecnología elegidas. Fijar percentiles o concurrencias ahora sería inventar cifras que el cliente no dio.
3. Una parte importante de los NFR es verificable por inspección o prueba aunque no lo diga en un campo aparte. Prueba de ello podemos mencionar esta pequeña lista: NFR1, NFR8, NFR10, NFR13, NFR16, NFR17 y NFR18.
4. NFR2 tiene una referencia comparativa concreta (Holded), que es la forma en que el cliente expresó la necesidad.

Citas textuales de la transcripción de la entrevista con el cliente (Alejandro Ruiz) que respaldan estos NFR:
- **Sobre NFR1 y NFR2 (simplicidad frente a Holded):** *"Hacer un chequeo que no nos lleve más de un minuto. Minimizar el número de clics al máximo. Que la experiencia de usuario no requiera un aprendizaje de un software como Holded, por ejemplo. Si no, contrataríamos uno ya existente. Pero queremos algo lo más simple posible..."* (Apartado *Criterios de éxito del sistema*).
- **Sobre NFR12 ("miles de clientes"):** *"A nivel de datos debe ser totalmente escalable. O sea, nuestro objetivo es tener miles de clientes."* (Apartado *Turno de preguntas y aclaraciones*).
- **Sobre NFR13 y la ausencia de métricas técnicas previas:** *"¿Hay alguna preferencia en cuanto a las tecnologías usadas tanto para desarrollar la app como para la base de datos? Es solo funcional, o sea, que sea lo más rápido posible, multidispositivo, entonces técnicamente no."* (Apartado *Turno de preguntas y aclaraciones*).

  
### A5. Organización, identificadores y dependencias

En este punto se expone que no se ha reflejado la independencia respecto a Brain, en nuestro repositorio encontramos las siguientes citas:

- BR5 (línea 19): “aplicación de gestión exclusivamente interna e independiente de Brain”.
- FR37 (línea 224): “La aplicación no tendrá chat ni acceso para clientes”. Es decir, FR37 no es solo “canal de envío”: implementa la parte “exclusivamente interna” de BR5.
- NFR5 (línea 330) tiene dos partes: independencia de Brain y arquitectura modular con bajo acoplamiento. FR42 exige que la integración de cobros esté encapsulada para cambiar de proveedor sin tocar el modelo, que es justo el bajo acoplamiento de NFR5.

Por tanto, podemos definir los siguientes contraargumentos:
1. BR5 -> FR37 está justificada: BR5 tiene dos componentes y FR37 materializa el primero (aplicación solo interna, sin acceso de clientes).
2. NFR5 -> FR42 está justificada por la segunda frase de NFR5 (bajo acoplamiento): FR42 es el requisito funcional donde ese encapsulamiento se aplica.
3. La independencia de Brain es una restricción que, por naturaleza, no tiene un FR que la “implemente”; se refleja en NFR5.

Por tanto, la supuesta falta de reflejo de la independencia de Brain no es un error de diseño, sino una separación deliberada: BR5 establece la restricción de negocio macro, mientras que FR37 y NFR5/FR42 operacionalizan dicho aislamiento a nivel de componentes internos y pasarelas.

---

## B. Diagrama UML

A continuación vamos a señalar los diferentes aspectos que se han visto comprometidos junto con su justificación: 

### B2. Casos de uso y nivel de abstracción

En este caso se expone que hay una separación y por tanto una incoherencia entre los casos de uso 20 y 24, ya que son dos casos de uso que se encargan de las copias de seguridad, en este caso vamos a exponer la justificación de los aspectos que aparecen en ambos casos:

1. CU-20 y CU-24 no son el mismo objetivo dividido de forma incoherente: tienen actores principales distintos (persona frente a reloj) y metas distintas (supervisar/restaurar frente a generar).
2. CU-20 responde a un único objetivo del superadministrador: supervisar la salud y trazabilidad del sistema, tal como lo formula UR3.
3. La dependencia CU-20 -> CU-24 está justificada: CU-20 lista y restaura las copias que produce CU-24.

En resumen, la separación entre CU-20 y CU-24 responde a una clara separación de responsabilidades entre la supervisión activa humana y la automatización desatendida del planificador.

Si se considera necesario, que por nuestra parte no lo es, se puede separar el aspecto de restauración de copias.

### B3. Semántica de relaciones UML

Vamos a desmembrar punto por punto estos relaciones:

- B3.1. CU-08 «extend» CU-13:

En el repositorio tenemos la siguiente información: 
1. CU-13. Desactivación de cuentas (línea 352), extensión 3a: “El administrador prefiere reasignar los clientes antes: lo hace (CU-08) y vuelve al paso 4.”
2. Paso 3 de CU-13: el sistema muestra los clientes asignados del usuario a desactivar.
3. CU-08 (línea 216), disparador: “…o un usuario se desactiva”.

Por tanto, la extensión está documentada con todos sus elementos: punto de extensión (paso 3 de CU-13), condición (el administrador decide reasignar antes de desactivar), comportamiento añadido (CU-08) y punto de retorno (paso 4). Es opcional, porque si no se hace los clientes quedan pendientes de reasignar (paso 6).

- B3.2. CU-23 «extend» CU-09:

En el repositorio tenemos la siguiente información: 
1. CU-09 (línea 240) no es solo “consultar métricas”: su escenario principal es listar facturas, filtrarlas y ver el detalle de una factura (pasos 1–6).
2. Extensión 6a de CU-09: “El usuario quiere el PDF: lo descarga (CU-23).”

Por tanto, el punto de extensión existe y está nombrado (paso 6, detalle de la factura), con su condición (el usuario quiere el PDF). Descargar el PDF desde el detalle de una factura es un comportamiento opcional que añade algo al caso base.

- B3.3. CU-30 «include» CU-28

En este caso es debatible la reclamación, ya que podríamos haber hecho un modelo más limpio, aun así se puede exponer que: 
1. CU-30 (línea 768): la conversión está en el escenario principal (pasos 6–7: “continúa en CU-28, paso 4”).
2. Garantía de éxito de CU-30: “El presupuesto aceptado se convierte en un borrador de factura con sus mismas líneas.”
3. Si el cliente rechaza (extensión 5a), el caso termina sin llegar al punto de inclusión.

Por tanto, según nuestra ficha, el caso CU-30 solo termina con éxito si se crea la factura; por eso la creación forma parte del flujo principal y se modeló como «include». El rechazo es un final alternativo que no alcanza el punto de inclusión, lo cual es compatible con «include». Además, diagrama y ficha son coherentes entre sí.

---

## D. Scrum

Aquí vamos a simplemente aclarar la organización y algunos puntos mencionados:

### D2. Objetivos, backlog y reparto

Aquí tenemos que remitirnos al contenido que hay en el repositorio:
- El README citado es Diagramas/README.md, que sí está en el repositorio desde el 01/10/2026 y contiene el reparto completo por persona (Francis, Marcondes, Esperanza, Juan, Álvaro) con los casos de cada uno.
- Planificacion/Planificación.md, subido el 14/10/2026 (antes de la entrega), tiene una tabla con responsable, inicio, fin y estado de cada tarea, incluidas las del Sprint 3 (tareas 5–9: redefinición de RF y RNF, requisitos de usuario y de negocio, plantilla de casos de uso).
- Error propio: en sprint_backlog.md, el enlace al acta del Sprint 3 apunta a acta14_10_2026.md, pero el fichero se llama acta_14_10_2026.md. El enlace está roto, por tanto asumimos responsabilidades si se tratara de ese error.

Aunque cabe recalcar dos puntos:
1. El README no es un archivo externo: forma parte del repositorio entregado y está enlazado desde la planificación.
2. Los responsables del Sprint 3 constan en la planificación del proyecto, entregada en el mismo repositorio.

Por tanto, asumimos el error del hipervínculo erróneo, pero al tener el repositorio accesible, se pueden observar los cambios que se mencionan.

### D3. Seguimiento y adaptación

Aquí es importante ver qué hay exactamente en el repositorio:
1. Planificación.md tiene una tabla con estado por tarea (Asignado, En progreso, Hecho, Revisado) y un cronograma por semanas.
2. Las actas no son externas: están en la carpeta Actas/ del repositorio.

Por tanto, sí existía una tabla de tareas con estado y un cronograma dentro del repositorio; las actas forman parte del entregable. Aunque aceptamos que hubo un error de desactualización del cronograma por parte del Scrum master que por supuesto ya está arreglado.

### D4. Revisiones y retrospectivas

Aceptamos el error, faltó planes de acción en torno a los errores cometidos y una mejor evidencia del trabajo que se realizó.

## E. Presentación

Es una penalización muy pequeña, pero los autores de este documento no entienden exactamente por qué, si el argumento del punto no declara errores, se reste 0.05 en el punto, es una simple aclaración.
