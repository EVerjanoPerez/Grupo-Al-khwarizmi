# Requisitos funcionales y no funcionales

**Proyecto:** Sistema de Administración Interna, Facturación y Cobros (aplicación independiente para la gestión de la startup).

## 1. Requisitos de negocio



## 2. Requisitos de usuario



## 3. Requisitos funcionales

### A. Acceso, usuarios y permisos

- FR1. el sistema debe permitir el alta de nuevos clientes, registrando obligatoriamente tanto a la entidad corporativa (empresa) como a la persona física asociada y sus datos de contacto.
- FR1.1. El sistema permitirá recuperar la contraseña mediante un enlace que caduca.
    - Dependencias: FR1. 
- FR1.2. El sistema cerrará la sesión tras un periodo de inactividad y permitirá cerrarla manualmente.
    - Dependencias: FR1. 
- FR1.3. El sistema aplicará una protección básica ante intentos de login fallidos repetidos.
    - Dependencias: FR1. 
- FR2. El sistema permitirá al superadministrador, o a un administrador con permiso, crear o invitar usuarios internos y asignarles un rol.
    - Dependencias: FR1, FR3. 
- FR2.1. El sistema permitirá desactivar un usuario sin borrar el registro de sus acciones; al desactivarlo se invalidarán sus sesiones activas.
    - Dependencias: FR2. 
- FR3. El sistema implementará un modelo de autorización centralizado con tres roles (Superadministrador, Administrador, Usuario) y permisos asignables, extensible a nuevos roles.
    - Dependencias: FR1. 
- FR3.1. El usuario operativo solo podrá ver y modificar la información de los clientes que tenga asignados, crear facturas si su permiso lo permite, seguir cobros y descargar documentos. No podrá asignarse clientes, cambiar tarifas globales, crear series ni ver el dashboard global.
    - Dependencias: FR3, FR4. 
- FR3.2. El servidor comprobará los permisos en cada petición: acceder por URL directa a un elemento no autorizado será rechazado, y los menús, favoritos o recientes no servirán como mecanismo de autorización.
    - Dependencias: FR3.
- FR3.3. La visualización de datos personales (nombre y contacto) y fiscales (razón social, NIF, dirección) se restringirá según rol y asignación. (Se eliminan los "datos bancarios", que el cliente no mencionó.)
    - Dependencias: FR3.
- FR4. El sistema permitirá a un administrador asignar y desasignar a cada cliente un responsable interno. El nuevo responsable heredará las tareas pendientes y el histórico seguirá indicando quién hizo cada acción.
    - Dependencias: FR3, FR5.
- FR4.1. El sistema permitirá delegar temporalmente los clientes de un usuario en otro durante un intervalo, sin perder la asignación principal. La delegación caducará sola.
    - Dependencias: FR4.

### B. Clientes y contactos

- FR5. El sistema permitirá dar de alta un cliente (empresa) con nombre comercial, razón social, identificador fiscal, domicilio fiscal, país, moneda, sector, ubicación, etiquetas, responsable interno y estado.
    - Dependencias: FR1, FR3.
- FR5.1. El sistema propondrá valores por defecto (país, moneda, vencimiento y tarifa sugerida según el rango) y mostrará qué va a ocurrir antes de confirmar.
    - Dependencias: FR5.
- FR5.2. El sistema validará campos obligatorios, emails e identificadores fiscales cuando se conozca el país, sin asumir el formato español ni que todas las direcciones tengan provincia. Distinguirá los datos imprescindibles para emitir facturas de los que solo son convenientes.
    - Dependencias: FR5.
- FR5.3. Antes de crear un cliente, el sistema buscará posibles duplicados por identificador fiscal y avisará, sin fusionar automáticamente.
    - Dependencias: FR5.
- FR6. El sistema permitirá registrar varias personas de contacto por empresa, cada una con su función (contratación, facturación, cobros), idioma preferido, canales y consentimientos, pudiendo ser modificados en cualquier momento. Los cambios de datos fiscales quedarán auditados.
    - Dependencias: FR5.
- FR6.1. Una persona podrá asociarse a varias empresas.
    - Dependencias: FR6.
- FR6.2. El sistema permitirá marcar un contacto como inactivo sin perder el histórico de los envíos que recibió.
    - Dependencias: FR6.
- FR6.3. El sistema avisará si se intenta eliminar o desactivar el único contacto de facturación de un cliente activo.
    - Dependencias: FR6.
- FR8. El sistema gestionará los estados del cliente que afectan a la facturación: pendiente de activación, activo, pausado, baja solicitada, inactivo (baja efectiva) y bloqueado administrativamente. Antes de confirmar un cambio de estado mostrará sus consecuencias.
    - Dependencias: FR5
- FR9. El sistema ofrecerá una búsqueda global por nombre comercial, razón social, identificador fiscal, persona de contacto, email y número de factura, y filtros por ubicación, sector, país, estado, responsable y etiquetas.
    - Dependencias: FR3, FR5.
- FR9.2. El sistema ofrecerá acceso rápido a clientes favoritos y recientes.
    - Dependencias: FR9, FR3.2. 
- FR10. El sistema permitirá etiquetar clientes con etiquetas administrables (estratégico, beta, partner…). Las situaciones derivadas de las facturas, como "impagado", no serán etiquetas manuales.
    - Dependencias: FR5.
- FR11. La ficha del cliente mostrará su saldo, sus facturas, sus pagos y una línea temporal de eventos de negocio: alta, cambios de responsable y tarifa, facturas, pagos, incidencias, pausas, baja y reactivación.
    - Dependencias: FR5.
- FR12. El sistema permitirá importar clientes desde CSV con mapeo de columnas, previsualización y listado de errores antes de confirmar, y con aviso de identificadores fiscales duplicados.
    - Dependencias: FR5, FR5.3.
- FR12.2. Cada importación quedará registrada como una operación con resumen de creados, actualizados, rechazados y motivos.
    - Dependencias: FR12.
- FR13. El sistema permitirá reactivar la ficha histórica de un cliente que vuelve, en lugar de crear un duplicado.
    - Dependencias: FR8, FR5.3.
- FR14. La eliminación de clientes será siempre lógica: se conservará el histórico y el estado, sujeto a lo que permita la ley.
    - Dependencias: FR5

### C. Suscripción

- FR15. El sistema registrará para cada cliente su suscripción, separada de las facturas: fecha de inicio, plan o rango, tarifa acordada, conceptos facturables, responsable, condiciones de pago (incluido el vencimiento pactado) y documentos firmados fuera de la plataforma. No existirá permanencia: el servicio es mes a mes.
    - Dependencias: FR5. 
- FR15.1. La suscripción podrá tener una fecha de inicio futura. Hasta esa fecha el cliente aparecerá como pendiente de activación, sin facturas ni recordatorios.
    - Dependencias: FR15, FR8.
- FR15.2. Si el alta se produce a mitad de mes, el sistema emitirá una factura proporcional hasta fin de mes y la mensualidad completa desde el día 1 siguiente. La regla de cálculo (días naturales reales del mes, incluido el alta el último día) estará documentada y el cálculo se mostrará antes de confirmar el alta.
    - Dependencias: FR15.
- FR15.3. El sistema permitirá registrar de forma explícita la excepción comercial de empezar a facturar el mes siguiente sin prorrateo.
    - Dependencias: FR15.2.
- FR15.4. El sistema impedirá activar la recurrencia de un cliente sin datos fiscales suficientes y explicará qué falta.
    - Dependencias: FR15, FR5.2.
- FR16. El sistema permitirá pausar una suscripción con motivo, fecha de inicio y, si se conoce, fecha prevista de reactivación. Durante la pausa no se generarán mensualidades, pero las facturas antiguas seguirán su curso.
    - Dependencias: FR15.
- FR16.1. Al llegar la fecha de reactivación, el sistema avisará al responsable y pedirá confirmación si la política lo exige.
    - Dependencias: FR16.
- FR17. El sistema permitirá registrar la solicitud de baja de un cliente, en la aplicación o recibida por API, y creará la tarea correspondiente.
    - Dependencias: FR15.
- FR17.1. Al hacerse efectiva la baja, el sistema dejará de generar mensualidades desde esa fecha. Si hay deuda pendiente, el recobro de las facturas anteriores continuará: la deuda no bloquea la baja y la baja no borra la deuda.
    - Dependencias: FR17.

### D. Tarifas, excepciones y aprobaciones.

- FR18. El superadministrador gestionará la tabla de rangos de facturación anual y sus precios como configuración versionada con fecha de efecto. Cambiarla no alterará las facturas ya emitidas.
- FR19. El sistema lanzará una campaña anual (o en la fecha configurada) con los clientes cuya facturación anual debe revisarse.
    - Dependencias: FR18, FR15. 
- FR19.1. El dato de facturación anual podrá obtenerse de una fuente simulada o introducirse a mano indicando fuente y fecha. Si no está disponible, la revisión quedará pendiente.
    - Dependencias: FR19.
- FR19.2. El sistema propondrá el rango correspondiente, pero el cambio requerirá la aprobación de un responsable. Se registrará quién lo aprobó, cuándo, el precio anterior, el nuevo y la fecha de efecto, sin reescribir meses anteriores.
    - Dependencias: FR19.1, FR21.
- FR19.3. El sistema permitirá corregir después un dato de revisión con trazabilidad y calcular su impacto. Cualquier regularización será una acción explícita y aprobada, nunca facturas retroactivas automáticas.
    - Dependencias: FR19.2.
- FR20. El sistema permitirá registrar excepciones comerciales (descuento temporal, mes gratuito, precio pactado de un servicio) con fecha de fin. El usuario las solicita y requieren aprobación; al caducar se dejan de aplicar.
    - Dependencias: FR15, FR21.
- FR21. El sistema dispondrá de un flujo de aprobación para las operaciones sensibles: importe modificado a mano, descuento extraordinario, vencimiento fuera de lo habitual, factura puntual de importe alto, cambio de rango, rectificativa e incobrable, según umbrales configurables. Las recurrentes que salen de una suscripción aprobada no necesitan aprobación. El sistema explicará por qué un elemento está pendiente.
    - Dependencias: FR3.
- FR21.1. Por encima de un umbral superior configurable, la aprobación corresponderá al superadministrador.
    - Dependencias: FR21.
- FR21.2. El sistema tendrá una bandeja de aprobaciones donde el solicitante verá el estado (pendiente, aprobada o rechazada) y el comentario del aprobador.
    - Dependencias: FR21.
- FR21.3. Cuando la regla exija separación de funciones, el aprobador no podrá aprobar su propia solicitud.
    - Dependencias: FR21.
FR21.4. Un administrador podrá reasignar una aprobación o definir un suplente; la reasignación quedará registrada.
    - Dependencias: FR21.

### E. Facturación

- FR22. El sistema gestionará el ciclo de vida de la factura con, al menos, los estados: borrador, pendiente de aprobación, lista para emitir, emitida, enviada, vencida, pagada, con diferencia pendiente, en disputa, incobrable, anulada y rectificada. Cada cambio de estado tendrá una causa registrada.
    - Dependencias: FR21.
- FR23. El sistema permitirá crear facturas puntuales con varias líneas, cada una con concepto, cantidad, precio unitario, descuento e impuesto.
    - Dependencias: FR5, FR27.
- FR23.1. El sistema tendrá un catálogo ligero de conceptos frecuentes con precio orientativo e impuesto por defecto, ajustables al facturar. Cambiar el catálogo no modificará las facturas antiguas.
    - Dependencias: FR23.
- FR24. El día 1 de cada mes el sistema generará automáticamente las facturas recurrentes a partir de los conceptos de cada suscripción activa, aplicando la tarifa y las excepciones vigentes, las pausas, las bajas y los vencimientos pactados.
    - Dependencias: FR15, FR18, FR20, FR16, FR17.
- FR24.1. La generación será idempotente: si el sistema estaba caído, al volver generará las facturas que falten sin duplicar ninguna.
    - Dependencias: FR24.
- FR24.2. Si falla un cliente, el resto seguirá procesándose. Cada excepción (falta contacto, cliente pausado, baja efectiva…) quedará identificada y se notificará a quien pueda resolverla.
    - Dependencias: FR24.
- FR25. Antes de emitir una factura el sistema mostrará una vista previa y bloqueará la emisión si falta un dato fiscal imprescindible, indicando cuál.
    - Dependencias: FR23, FR5.2.
- FR25.1. El sistema advertirá, sin bloquear, si el importe se sale mucho del patrón habitual del cliente según una regla configurable.
    - Dependencias: FR25.
- FR26. El superadministrador configurará series de facturación (ordinaria, rectificativas y otras). Cada factura recibirá al emitirse un número único dentro de su serie; los borradores no consumen número.
    - Dependencias: FR22.
- FR26.1. La numeración será correcta aunque varias personas emitan a la vez, y reintentar un envío fallido no generará otra factura ni otro número.
    - Dependencias: FR26.
- FR27. Los impuestos de cada línea se seleccionarán entre los dados de alta en la configuración, sin que el sistema deduzca la legislación de cada país. Cada factura se emite y conserva en su moneda.
- FR28. El sistema generará el PDF a partir de los datos de la factura, con la identidad corporativa (logo, datos fiscales, numeración, fechas, líneas, bases, impuestos, total, condiciones de pago y datos de soporte) y en el idioma del cliente.
    - Dependencias: FR22.
- FR28.1. El sistema conservará el PDF original emitido: un cambio posterior de plantilla o logo no alterará las facturas históricas, y el PDF será idéntico en cualquier pantalla o descarga.
    - Dependencias: FR28.
- FR29. Una factura en borrador podrá corregirse libremente. Una factura emitida no podrá modificarse en silencio: los cambios permitidos quedarán trazados y los cambios relevantes se harán mediante rectificativa.
    - Dependencias: FR22, FR30.
- FR30. Los usuarios con permiso administrativo podrán crear una factura rectificativa o nota de crédito a partir de la original. El sistema propondrá los datos a rectificar, generará el documento vinculado y requerirá aprobación según el importe. El saldo del cliente tendrá en cuenta ambos documentos.
    - Dependencias: FR29, FR21, FR26.
- FR31. Solo se podrán borrar facturas en borrador que nunca se emitieron, dejando traza. Una factura emitida nunca se borra.
    - Dependencias: FR22.
- FR32. Un administrador autorizado podrá cerrar periodos (mes o trimestre). En un periodo cerrado no se permitirán cambios ordinarios en elementos económicos, y reabrirlo requerirá permiso de superadministrador y un motivo.
    - Dependencias: FR29.
- FR33. El sistema permitirá consultar y filtrar facturas por fecha, estado, tipo (puntual o recurrente), cliente, responsable, país y moneda, y descargar su PDF según los permisos.
    - Dependencias: FR3, FR28.
- FR34. El sistema permitirá crear presupuestos, enviarlos, marcarlos como aceptados o rechazados con evidencia adjunta y convertirlos en factura reutilizando sus líneas, conservando cada documento su identidad.
    - Dependencias: FR23.
- FR35. El sistema permitirá imputar facturas a centros de coste o proyectos administrables para los informes; renombrar una categoría no alterará el histórico. 
    - Dependencias: FR23.

### F. Envios y comunicaciones.

- FR36. Al emitir una factura, el sistema la enviará automáticamente por email a los contactos de facturación. WhatsApp será un canal complementario si el contacto lo ha consentido y tiene un número adecuado. Los proveedores de envío podrán ser simulados.
    - Dependencias: FR22, FR6, FR37. 
- FR36.1. El sistema registrará cada intento de envío (canal, destinatarios en ese momento, fecha, resultado y plantilla usada) y distinguirá el envío original de los reintentos.
    - Dependencias: FR36. 
- FR36.2. Si el envío rebota, la factura no se marcará como notificada, se abrirá una incidencia de entrega, se avisará al responsable y se permitirá corregir el contacto y reenviar (o probar otro contacto de facturación según la configuración) sin duplicar la factura.
    - Dependencias: FR36.1.
- FR36.3. Cada mensaje tendrá un estado (en cola, enviado, entregado si el proveedor lo informa, fallido o reintentado). El sistema no inventará confirmaciones de lectura.
    - Dependencias: FR36.1.
- FR37. El sistema tendrá plantillas de comunicación editables por quien tenga permiso (factura, recordatorio previo, vencimiento, impago, disputa resuelta, revisión de datos), con variables (empresa, número, importe, vencimiento, responsable, soporte), historial de versiones y al menos español e inglés. El idioma se elegirá por cliente o contacto.
- FR37.1. La previsualización de una plantilla avisará si falta una variable y permitirá enviar una prueba interna.
    - Dependencias: FR37.
- FR38. Todas las comunicaciones al cliente incluirán los datos de contacto de soporte. La aplicación no tendrá chat ni acceso para clientes.
    - Dependencias: FR36.

### G. Recordatorios, recobro e incidencias.

- FR39. El sistema enviará automáticamente un recordatorio previo (hacia el día 3) y otro al vencimiento (5 días por defecto). Después mantendrá un flujo de impago amistoso cada 2–3 días hasta que se detecte el pago o intervenga una persona. Estos tiempos serán configurables por el superadministrador, y el vencimiento pactado de cada cliente prevalecerá sobre el general.
    - Dependencias: FR36, FR15.
- FR39.1. El sistema tendrá un calendario laboral configurable (fines de semana y festivos) y aplicará una regla definida cuando un vencimiento o recordatorio caiga en día no laborable.
    - Dependencias: FR39.
- FR39.2. Al cambiar las reglas de recordatorio, el sistema mostrará el calendario resultante para un cliente de ejemplo.
    - Dependencias: FR39.
- FR40. El responsable podrá poner una factura "en disputa" o en "seguimiento pausado" con motivo y fecha de revisión. Durante ese tiempo no se enviarán recordatorios, la factura seguirá figurando como no cobrada y el sistema avisará al terminar la pausa.
    - Dependencias: FR39.
- FR41. Un administrador, con motivo y aprobación, podrá declarar una deuda incobrable o cancelarla. Este estado será distinto de "pagada".
    - Dependencias: FR21, FR22.
- FR42. El sistema permitirá abrir incidencias internas de facturación asociadas a un cliente y, opcionalmente, a una factura, con responsable, estado, prioridad, comentarios, adjuntos y resolución al cerrarlas. Una incidencia podrá pausar los recordatorios si así se indica.
    - Dependencias: FR5, FR40.

### H. Cobros y conciliación.

- FR43. El sistema se integrará con una fuente de cobros Stripe simulada, que se podrá consultar y que enviará eventos de pago simulados. La integración estará encapsulada para añadir otras fuentes (Revolut, banco) sin cambiar el modelo de factura.
- FR43.1. Si llega dos veces el mismo evento de pago, el cobro solo se contabilizará una vez.
    - Dependencias: FR43.
- FR44. El sistema intentará asociar cada movimiento (importe, fecha, moneda, concepto, referencia) a una factura. Si la coincidencia es clara, la conciliará o propondrá según la política; si hay dudas, la presentará a una persona. Nunca elegirá al azar entre facturas iguales.
    - Dependencias: FR44.
- FR44.1. Un cobro sin referencia clara quedará pendiente de conciliación manual.
    - Dependencias: FR44.
- FR44.2. Si el importe recibido es menor o mayor que el de la factura, esta no se marcará como pagada: el movimiento quedará asociado y la diferencia pendiente de revisión. No existe el pago fraccionado como modalidad comercial.
    - Dependencias: FR44.
- FR44.3. Se podrá deshacer una conciliación dejando constancia de quién o qué regla la hizo, quién la revirtió y por qué. Los eventos nunca se borran.
    - Dependencias: FR45.
- FR45. Una factura solo pasará a "pagada" cuando haya un cobro detectado o validado. El movimiento que lo justifica será visible, se detendrán los recordatorios y se avisará al responsable.
    - Dependencias: FR44, FR39.
- FR45.1. Un usuario autorizado podrá validar manualmente un cobro aportando el justificante.
    - Dependencias: FR45.
- FR46. El sistema calculará el saldo de cada cliente (facturado, cobrado y pendiente, considerando las rectificativas) de forma trazable hasta las facturas y movimientos que lo componen.
    - Dependencias: FR30, FR45.

### I. Notificaciones, trabajo diario y colaboración

- FR47. El sistema enviará notificaciones internas al responsable del cliente sobre eventos relevantes: pago confirmado, factura vencida, rebote, disputa, revisión anual, aprobación rechazada y menciones. El administrador verá una bandeja global; el usuario, solo la suya.
    - Dependencias: FR4.
- FR47.1. Cada notificación tendrá prioridad y podrá marcarse como leída o como atendida, que son estados distintos.
    - Dependencias: FR47.
- FR47.2. Cada usuario podrá elegir qué notificaciones recibe en la aplicación, por email o en un resumen diario, dentro de lo que permita la configuración corporativa.
    - Dependencias: FR47.
- FR48. Los administradores podrán configurar alertas corporativas a partir de un conjunto cerrado de condiciones parametrizables (p. ej., factura vencida más de 15 días, cliente con más de X € pendientes, tres fallos de envío, revisión anual sin resolver durante una semana). Un usuario no podrá desactivar los recordatorios de sus clientes salvo pausando un caso concreto con permiso y motivo.
    - Dependencias: FR47.
- FR49. Al entrar, cada usuario verá una pantalla de trabajo diario con las facturas que vencen pronto, las vencidas, los pagos por conciliar, los problemas de entrega, las aprobaciones pendientes, las revisiones anuales y sus tareas asignadas, con acceso directo a cada elemento.
    - Dependencias: FR3, FR47.
- FR50. El sistema permitirá añadir comentarios internos a clientes, facturas e incidencias, con menciones que notifican al usuario mencionado. Habrá dos niveles de visibilidad: normal y restringido a administradores. Los comentarios nunca se enviarán al cliente.
    - Dependencias: FR47.
- FR51. El sistema permitirá adjuntar documentos con tipo y descripción (pedido, presupuesto aceptado, justificante, documento firmado, documento de gestoría) a clientes, facturas e incidencias. Los formatos y tamaños estarán limitados, y el acceso seguirá los permisos del elemento.
    - Dependencias: FR3.
- FR52. El sistema permitirá acciones masivas (reasignar clientes, exportar una selección, cambiar el sector, generar informes) que mostrarán cuántos elementos afectan, pedirán confirmación si son sensibles y devolverán un resultado con éxitos y errores. No habrá un "marcar como pagadas" masivo sin controles.
    - Dependencias: FR3.

### J. Informes, dashboard y exportaciones

- FR53. Los administradores tendrán un dashboard financiero con lo facturado (semana, mes, trimestre y año), lo cobrado, lo pendiente, lo vencido, el tiempo medio de cobro, la evolución, los clientes con más deuda, el recurrente esperado del mes, lo facturado en puntuales y una previsión sencilla basada en las suscripciones activas. Cada cifra llevará al detalle que la compone.
    - Dependencias: FR3, FR46.
- FR53.1. El sistema podrá convertir importes a una moneda base con un tipo de cambio registrado, solo para los informes y sin alterar el importe legal de la factura. 
    - Dependencias: FR53, FR27.
- FR54. El sistema permitirá generar informes filtrados por periodo, cliente, responsable, país, sector, estado, tipo y moneda, y exportarlos a CSV y PDF respetando los permisos del usuario. Cada exportación registrará fecha, filtros y autor, y las exportaciones masivas quedarán auditadas.
    - Dependencias: FR3.
- FR54.1. Los informes pesados se generarán en segundo plano y el sistema avisará cuando estén listos.
    - Dependencias: FR54.
- FR54.2. Los informes y exportaciones generados se conservarán durante un tiempo configurable, con una política distinta de la de los documentos contables.
    - Dependencias: FR54.
- FR55. El sistema permitirá exportar un paquete mensual o trimestral para la gestoría (facturas emitidas, rectificativas, cobros y PDF en un ZIP), registrando cuándo se generó, qué periodo cubre y su versión si se regenera. La gestoría no tendrá usuario en la aplicación.
    - Dependencias: FR33, FR45.

### K. Configuración, automatización y administración.

- FR56. El sistema separará la configuración global de los datos operativos. La configuración incluirá plazo de pago, cadencia de recordatorios, dominio de correo permitido, datos fiscales y logo del emisor, plantillas, series, monedas, impuestos, umbrales de aprobación, canales de notificación, calendario laboral y políticas de automatización. Los cambios se validarán, se auditarán y solo afectarán a los elementos nuevos.
    - Dependencias: FR3. 
- FR56.1. Los datos del emisor serán configuración. Como ampliación, podrá haber varias entidades emisoras con series y cuentas propias. 
    - Dependencias: FR56, FR26. 
- FR57. El sistema registrará cada ejecución de los procesos automáticos (generación mensual, recordatorios, conciliación e informes) con el número de elementos procesados, creados y fallidos, y el motivo de cada fallo.
    - Dependencias: FR24, FR39, FR44. 
- FR58. Las tareas largas se ejecutarán en segundo plano con un estado visible (pendiente, procesando, completada, completada con errores) que no dependerá de la pestaña abierta. La emisión, el envío y la conciliación estarán protegidos frente al doble clic.
    - Dependencias: FR24, FR36, FR44.
- FR59. El sistema tendrá una pantalla de salud con la última ejecución de la generación recurrente, la cola de emails, los errores de integración recientes y el último backup correcto.
    - Dependencias: FR57.
- FR60. El sistema mantendrá una auditoría de negocio que no podrá editar nadie, ni siquiera el superadministrador. Registrará quién crea clientes, cambia datos fiscales, emite facturas, cambia vencimientos, aprueba descuentos, reasigna clientes, marca disputas, concilia, exporta y cambia la configuración. El superadministrador podrá consultarla y filtrarla.
    - Dependencias: FR1. 
- FR61. El sistema hará copias de seguridad automáticas de datos y documentos según una política definida, y tendrá un procedimiento de restauración documentado y probado.
- FR62. El sistema permitirá localizar los datos personales de un contacto y anonimizarlos o eliminarlos cuando legalmente proceda, teniendo en cuenta los registros activos, el histórico y las copias, sin invalidar las facturas emitidas ni borrar datos con obligación contable. Los datos de facturación no se usarán automáticamente para campañas comerciales.
    - Dependencias: FR14, FR28.1, FR61.
- FR63. El sistema podrá cargarse con datos ficticios para demostraciones, separados de los de producción.
- FR64. Las funciones inteligentes que se añadan (clasificación de conceptos, detección de anomalías) serán opcionales y desactivables, respetarán los permisos y pedirán confirmación humana para cualquier sugerencia económica.
    - Dependencias: FR3.

----
## 4. Requisitos no funcionales

### A. Usabilidad y experiencia de usuario (UX)
- **NFR1.** La gestión del software debe ser sumamente rápida y fluida: acciones críticas como dar de alta un cliente, generar una factura recurrente, realizar un chequeo anual o comprobar el estado de un cobro no deben tomar más de 1 minuto, minimizando drásticamente el número de clics requeridos.
    - Dependencias: FR5, FR15, FR19, FR43.
- **NFR1.1.** La interfaz debe mantener un diseño minimalista, limpio y directo, evitando la complejidad excesiva del software de contabilidad comercial tradicional (por ejemplo, Holded).
- **NFR1.2.** Los controles de la interfaz deben ser utilizables de forma táctil y mediante teclado, garantizando la compatibilidad con herramientas de zoom y accesibilidad sin depender exclusivamente del ratón.

### B. Portabilidad y accesibilidad
- **NFR2.** El sistema debe desarrollarse obligatoriamente como una aplicación web accesible de forma fluida y multiplataforma, optimizada para su uso eficiente desde ordenadores, tablets y teléfonos móviles modernos (Android e iOS), permitiendo la operativa completa en movilidad.

### C. Independencia tecnológica y arquitectura
- **NFR3.** El sistema de facturación y administración interna debe ser una aplicación completamente independiente, sin tocar, depender ni integrarse con el núcleo del producto principal de la compañía (el Brain Operating System).
    - Dependencias: FR43.
- **NFR3.1.** La arquitectura debe seguir un diseño modular limpio (preferiblemente un monolito modular bien estructurado) que evite la sobreingeniería, garantizando un bajo acoplamiento y una alta cohesión entre componentes.
- **NFR3.2.** Las integraciones con servicios externos (pasarelas de pago como Stripe, servicios de correo o WhatsApp simulados) deben estar completamente encapsuladas, de modo que sustituir un proveedor simulado por uno real no exija rediseñar el modelo de negocio ni las facturas.
    - Dependencias: FR36, FR43.

### D. Consistencia de datos y precisión
- **NFR4.** El documento PDF de la factura generado de forma automática debe mantener una consistencia absoluta, siendo idéntico independientemente de la sección, dispositivo o plataforma desde donde se consulte, imprima o descargue, respetando la identidad corporativa de Turbine de forma inalterable frente a futuros cambios de plantilla o logotipo.
    - Dependencias: FR28.
- **NFR4.1.** El sistema debe garantizar una precisión decimal estricta en todos los cálculos monetarios (bases, impuestos, descuentos y totales), evitando la pérdida de céntimos o errores derivados del uso incorrecto de tipos de datos de punto flotante (`float`).
    - Dependencias: FR23, FR27.
- **NFR4.2.** Las marcas de tiempo e instantes de negocio deben almacenarse de forma normalizada para evitar fallos por zonas horarias, y las exportaciones (como ficheros CSV) deben garantizar la compatibilidad con caracteres internacionales y tildes.
    - Dependencias: FR54.

### E. Fiabilidad y tolerancia a fallos
- **NFR5.** Ningún dato, notificación o proceso periódico programado (como la generación mensual de facturas el día 1) podrá fallar en silencio. Las tareas automáticas deben ser completamente reintentables e idempotentes (si el servidor sufre una caída a medianoche, al recuperarse generará las facturas pendientes sin duplicar ninguna).
    - Dependencias: FR24, FR39.
- **NFR5.1.** Si un elemento o cliente específico falla durante un procesamiento masivo (como una importación CSV o un envío), la excepción debe quedar aislada y notificada, permitiendo que el resto del lote continúe procesándose con normalidad.
    - Dependencias: FR12, FR24.2, FR36.

### F. Escalabilidad y rendimiento
- **NFR6.** El sistema debe soportar un crecimiento masivo de miles de clientes activos y cientos de miles de facturas históricas sin requerir un rediseño de la arquitectura.
- **NFR6.1.** Las búsquedas globales y la visualización de listados amplios deben responder de forma prácticamente inmediata en condiciones normales de red, evitando cargar todo el histórico de golpe en el navegador mediante mecanismos adecuados de paginación o carga asíncrona.
    - Dependencias: FR9, FR33.
- **NFR6.2.** Las operaciones pesadas o de larga duración (como la generación masiva de PDFs o informes anuales extensos) deben ejecutarse de forma asíncrona en segundo plano sin congelar la interfaz de usuario, mostrando estados claros de avance.
    - Dependencias: FR28, FR54.1.

### G. Seguridad, privacidad y cumplimiento legal
- **NFR7.** Todos los datos personales y fiscales deben protegerse mediante un diseño que permita atender solicitudes de localización, anonimización o eliminación de contactos cuando legalmente proceda, asegurando que esto no invalide las facturas emitidas ni suprima los datos sujetos a obligación contable o fiscal.
    - Dependencias: FR14, FR28.1, FR62.
- **NFR7.1.** Las facturas y archivos sensibles no se almacenarán en URLs públicas o adivinables; su descarga requerirá autenticación rigurosa, permisos de rol activos y enlaces temporales caducables.
    - Dependencias: FR3, FR28, FR33.
- **NFR7.2.** Los tokens, credenciales o claves de acceso a servicios externos (aunque sean simulados) no deben incluirse jamás en duro dentro del repositorio de código fuente.
    - Dependencias: FR43.
- **NFR7.3.** Debe existir una separación absoluta entre los datos de producción y los entornos de prueba o demostración (*mock data*), prohibiendo el uso de datos reales de clientes en repositorios o pruebas.
    - Dependencias: FR63.
