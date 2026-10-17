# Casos de uso

---------

# Desactualizados 

## Actores 

| Actor | Tipo | Descripción |
|---|---|---|
| Usuario | Primario | Empleado de Turbine. Crea, consulta y edita solo los clientes y facturas que tiene asignados. |
| Administrador | Primario | Puede hacer todo lo del usuario sobre todos los clientes. Además gestiona usuarios, asigna clientes, aprueba solicitudes y consulta el dashboard. |
| Superadministrador | Primario | Puede hacer todo lo del Administrador. Además configura el sistema y consulta la auditoría y las copias de seguridad. |
| Planificador (tiempo) | Primario | Reloj del sistema que lanza los procesos periódicos: generación mensual, recordatorios, consulta de cobros, revisión anual y copias de seguridad. |
| Sistema externo | Primario | Otro sistema de Turbine que se comunica con la aplicación por API (por ejemplo, para notificar una baja). |
| Sistema de cobros | Secundario | Stripe (simulado) y, opcionalmente, Revolut. |
| Servicio de mensajería | Secundario | Email y WhatsApp. |

-----------

**Convenciones**
- **Ámbito:** en todos los casos es la *aplicación de facturación y cobros de Turbine*.
- **Actor «Usuario»:** cuando el actor principal es el Usuario, también pueden iniciar el caso el Administrador y el Superadministrador, que heredan sus casos por generalización. Del mismo modo, el Superadministrador hereda los del Administrador.
- **Visibilidad del Usuario:** el Usuario solo ve y gestiona sus clientes asignados. El Administrador ve todos.
- **Referencias:** "(CU-xx)" remite a otro caso de uso.
- **Numeración:** las extensiones se numeran con el paso del que salen (por ejemplo, 3a). "Cualquier paso" indica que la extensión puede ocurrir en cualquier momento.
- **Autenticación:** todos los casos iniciados por personas tienen como precondición que el actor se ha autenticado (CU-01). No se repite en cada ficha.

## Descripción de los casos de uso

Descripción de los casos de uso segun plantilla subida:

## CU-01 · Login
- **Actor principal:** Usuario
- **Actor secundario:** Servicio de mensajería (recuperación de contraseña)
- **Ámbito:** Aplicación de facturación
- **Nivel:** Subfunción
- **Descripción:** El usuario accede de forma segura a la aplicación con su correo corporativo. Tras entrar, solo puede acceder a lo que permite su rol. Incluye recuperar la contraseña y cerrar la sesión.
- **Interesados:**
  - Usuario: quiere entrar rápido y ver su trabajo.
  - Turbine: quiere que solo accedan sus empleados, con los permisos de su rol.
- **Precondiciones:** El usuario tiene una cuenta activa.
- **Garantía de éxito:** El usuario tiene una sesión iniciada con los permisos de su rol.
- **Garantía mínima:** No se da acceso sin validar las credenciales, y el intento queda registrado.
- **Disparador:** El usuario quiere acceder a la aplicación.
- **Escenario principal de éxito:**
  1. El usuario introduce su correo y su contraseña.
  2. El sistema comprueba que el correo pertenece al dominio corporativo.
  3. El sistema valida las credenciales.
  4. El sistema inicia la sesión y muestra al usuario su pantalla de trabajo diario (CU-35).
- **Extensiones:**
  - 2a. El correo no es corporativo: el sistema rechaza el acceso e informa del motivo.
  - 3a. Las credenciales son incorrectas: el sistema informa del error y permite reintentar. Tras varios intentos fallidos, bloquea la cuenta temporalmente.
  - 3b. La cuenta está desactivada: el sistema deniega el acceso.
  - 3c. El usuario ha olvidado la contraseña:
    - el sistema le envía un enlace de recuperación por correo (Servicio de mensajería);
    - el usuario define una contraseña nueva y vuelve al paso 1.
  - Cualquier paso: si el usuario cierra la sesión, el sistema la finaliza.

## CU-02 · Registro de usuarios internos
- **Actor principal:** Administrador (con permiso de gestión de usuarios)
- **Actor secundario:** Servicio de mensajería
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se registra o invita a un nuevo usuario interno y se le asigna el rol que controla sus accesos.
- **Interesados:**
  - Administrador: quiere dar acceso rápido a un empleado nuevo.
  - Turbine: quiere que cada persona tenga solo los permisos que le corresponden.
- **Precondiciones:** Si el actor es un administrador, tiene el permiso de gestión de usuarios.
- **Garantía de éxito:** La cuenta queda creada con su rol y el usuario recibe la invitación.
- **Garantía mínima:** No se crea ninguna cuenta duplicada ni ninguna con un correo que no sea corporativo.
- **Disparador:** Se incorpora un empleado que necesita usar la aplicación.
- **Escenario principal de éxito:**
  1. El administrador indica que quiere registrar un usuario interno.
  2. El administrador introduce el nombre, el correo corporativo y el rol.
  3. El sistema comprueba que el correo es corporativo y que no está ya registrado.
  4. El sistema crea la cuenta y envía la invitación por correo (Servicio de mensajería).
  5. El sistema registra la operación en la auditoría.
- **Extensiones:**
  - 2a. El administrador intenta asignar el rol de superadministrador: el sistema lo impide, porque solo puede hacerlo un superadministrador.
  - 3a. El correo no es corporativo: el sistema informa y vuelve al paso 2.
  - 3b. El correo ya está registrado: el sistema lo informa. Si la cuenta está desactivada, ofrece reactivarla.
  - 4a. Falla el envío de la invitación: la cuenta se crea igualmente, el sistema informa del fallo y permite reenviar la invitación.

## CU-03 · Configuración global del sistema
- **Actor principal:** Superadministrador
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se modifican los parámetros globales del sistema:
  - plazos de pago y calendario de recordatorios;
  - calendario laboral;
  - tarifas por rango;
  - series, impuestos y monedas;
  - umbrales de aprobación;
  - datos del emisor y de soporte;
  - plantillas;
  - funciones opcionales.
- **Interesados:**
  - Superadministrador: quiere adaptar el sistema sin depender de un programador.
  - Turbine: quiere que los cambios no alteren lo ya facturado y queden trazados.
- **Precondiciones:** El actor es superadministrador.
- **Garantía de éxito:** La nueva configuración queda guardada y se aplica desde ese momento, sin modificar facturas ya emitidas.
- **Garantía mínima:** Si no se guarda, sigue vigente la configuración anterior. Todo cambio guardado queda auditado.
- **Disparador:** El superadministrador necesita cambiar un parámetro del sistema.
- **Escenario principal de éxito:**
  1. El superadministrador elige el grupo de parámetros que quiere modificar.
  2. El sistema muestra los valores vigentes.
  3. El superadministrador introduce los valores nuevos.
  4. El sistema valida los valores.
  5. El superadministrador confirma los cambios.
  6. El sistema guarda la configuración y registra en la auditoría quién la cambió, cuándo y los valores anterior y nuevo.
- **Extensiones:**
  - 3a. El superadministrador edita una plantilla y pide una previsualización o un envío de prueba: el sistema la genera con datos de ejemplo y vuelve al paso 3.
  - 3b. El superadministrador pide cargar datos de demostración: el sistema solo lo permite en un entorno de pruebas.
  - 4a. Hay un valor no válido (por ejemplo, un umbral superior menor que el inferior): el sistema indica el error y vuelve al paso 3.
  - Cualquier paso: si el superadministrador cancela, no se guarda ningún cambio.

## CU-04 · Administración de la cuenta de recepción de pagos
- **Actor principal:** Superadministrador
- **Actor secundario:** Sistema de cobros
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se configura la conexión simulada con Stripe y, opcionalmente, con Revolut, de donde el sistema lee los cobros.
- **Interesados:**
  - Superadministrador: quiere que los cobros se detecten solos.
  - Turbine: quiere que las credenciales estén protegidas.
- **Precondiciones:** El actor es superadministrador.
- **Garantía de éxito:** La conexión queda configurada y comprobada.
- **Garantía mínima:** Las credenciales nunca se muestran en claro. Si la conexión falla, se mantiene la anterior.
- **Disparador:** Hay que conectar o cambiar la cuenta de cobros.
- **Escenario principal de éxito:**
  1. El superadministrador indica que quiere configurar la conexión con el sistema de cobros.
  2. El superadministrador introduce los datos de conexión.
  3. El sistema comprueba la conexión con el Sistema de cobros.
  4. El sistema guarda la configuración y lo registra en la auditoría.
- **Extensiones:**
  - 3a. La conexión falla: el sistema informa del error, no activa la configuración y vuelve al paso 2.

## CU-05 · Revisión anual del rango de facturación
- **Actor principal:** Administrador
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se revisa la facturación anual de los clientes que el sistema propone cada año y, si procede, se cambia su rango de tarifa con fecha de efecto.
- **Interesados:**
  - Administrador: quiere aplicar la tarifa correcta a cada cliente.
  - Cliente: no quiere que se le cambien facturas pasadas.
  - Turbine: quiere que el cambio esté justificado y aprobado.
- **Precondiciones:** El cliente tiene una suscripción activa.
- **Garantía de éxito:** El nuevo rango queda aprobado y se aplica desde su fecha de efecto.
- **Garantía mínima:** Las facturas anteriores no se modifican, y el dato introducido queda registrado con su fuente.
- **Disparador:** El sistema avisa de los clientes que toca revisar este año (CU-35).
- **Escenario principal de éxito:**
  1. El administrador consulta la lista de clientes pendientes de revisión.
  2. El administrador selecciona un cliente.
  3. El sistema muestra su rango actual y su facturación anual registrada.
  4. El administrador introduce la facturación anual nueva, con su fuente y su fecha.
  5. El sistema calcula el rango que corresponde según la tabla de tarifas.
  6. El administrador confirma el cambio y su fecha de efecto.
  7. El sistema deja el cambio pendiente de aprobación (CU-30).
  8. Tras la aprobación, el sistema actualiza la tarifa de la suscripción desde la fecha de efecto.
- **Extensiones:**
  - 4a. El dato corrige un error detectado después: el administrador indica el motivo y el sistema conserva el valor anterior en el historial.
  - 5a. El rango no cambia: el sistema registra la revisión como hecha y termina.
  - 7a. La aprobación se rechaza: el rango no cambia y el sistema avisa al administrador con el comentario.

## CU-06 · Activación de la facturación recurrente
- **Actor principal:** Usuario
- **Actor secundario:** Servicio de mensajería
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se registra la suscripción de un cliente (fecha de inicio, plan, tarifa, cuota y vencimiento pactado). Si el alta es a mitad de mes, se emite una factura prorrateada por los días restantes.
- **Interesados:**
  - Usuario: quiere dejar el cobro mensual automatizado.
  - Cliente: quiere pagar solo los días que usa el servicio el primer mes.
- **Precondiciones:** El cliente está registrado y asignado al usuario.
- **Garantía de éxito:** La suscripción queda activa (o programada) y, si corresponde, se emite y envía la factura prorrateada.
- **Garantía mínima:** No se activa ninguna suscripción sin datos fiscales suficientes.
- **Disparador:** Un cliente contrata el servicio recurrente.
- **Escenario principal de éxito:**
  1. El usuario selecciona el cliente y elige activar su facturación recurrente.
  2. El usuario indica la fecha de inicio, el plan, la cuota mensual y el vencimiento pactado.
  3. El sistema comprueba que el cliente tiene los datos fiscales necesarios.
  4. El sistema calcula la tarifa según el rango y muestra un resumen, con el prorrateo si el alta es a mitad de mes.
  5. El usuario confirma.
  6. El sistema registra la suscripción como activa.
  7. El sistema emite la factura prorrateada por los días restantes del mes y la envía al cliente (CU-12).
  8. El sistema registra la operación. Desde el día 1 siguiente se aplica la cuota normal (CU-32).
- **Extensiones:**
  - 2a. La fecha de inicio es futura: la suscripción queda programada y se activa en esa fecha.
  - 2b. El vencimiento pactado está fuera de lo habitual: queda pendiente de aprobación (CU-30).
  - 3a. Faltan datos fiscales: el sistema bloquea la activación e indica cuáles faltan.
  - 4a. Se pide no prorratear: el usuario indica el motivo y la excepción queda pendiente de aprobación (CU-30).
  - 7a. El alta es el día 1: no hay prorrateo y se pasa al paso 8.

## CU-07 · Delegación temporal de clientes
- **Actor principal:** Administrador
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Los clientes de un usuario se delegan en otro durante un intervalo (por ejemplo, unas vacaciones), sin perder la asignación principal.
- **Interesados:**
  - Administrador: no quiere que ningún cliente quede desatendido.
  - Usuario ausente: quiere recuperar sus clientes al volver.
- **Precondiciones:** Los dos usuarios están activos.
- **Garantía de éxito:** Durante el intervalo, el sustituto ve y gestiona esos clientes y recibe sus avisos. Al terminar, la delegación caduca sola.
- **Garantía mínima:** La asignación principal no cambia.
- **Disparador:** Un usuario va a ausentarse.
- **Escenario principal de éxito:**
  1. El administrador selecciona al usuario que se ausenta.
  2. El administrador indica el usuario sustituto y el intervalo.
  3. El sistema comprueba que el sustituto está activo y que las fechas son válidas.
  4. El sistema registra la delegación y avisa a ambos usuarios (CU-35).
- **Extensiones:**
  - 2a. Solo se delegan algunos clientes: el administrador los selecciona.
  - 3a. El sustituto no está activo o las fechas no son válidas: el sistema informa y vuelve al paso 2.
  - Cualquier paso posterior: el administrador cancela la delegación antes de tiempo y el sistema la finaliza.

## CU-08 · Asignación de clientes a usuario
- **Actor principal:** Administrador
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se asigna, reasigna o desasigna el usuario responsable de un cliente.
- **Interesados:**
  - Administrador: quiere repartir el trabajo.
  - Nuevo responsable: quiere recibir las tareas pendientes.
  - Turbine: quiere conservar quién hizo cada acción.
- **Precondiciones:** El usuario destino está activo.
- **Garantía de éxito:** El cliente tiene un nuevo responsable, que hereda sus tareas pendientes.
- **Garantía mínima:** El histórico conserva quién hizo cada acción antes del cambio.
- **Disparador:** Llega un cliente nuevo, se reparte la cartera o un usuario se desactiva.
- **Escenario principal de éxito:**
  1. El administrador busca y filtra los clientes (CU-24).
  2. El administrador selecciona el cliente.
  3. El administrador elige el usuario responsable.
  4. El sistema muestra las tareas pendientes que se traspasarán.
  5. El administrador confirma.
  6. El sistema asigna el cliente, traspasa las tareas, avisa al nuevo responsable (CU-35) y lo registra en la auditoría.
- **Extensiones:**
  - 2a. Son muchos clientes: se hace como acción masiva (CU-37).
  - 3a. El administrador desasigna el cliente: queda sin responsable y aparece en la bandeja global del administrador.

## CU-09 · Visualización de pagos e impagos
- **Actor principal:** Usuario
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se consulta el estado de los cobros (facturas pagadas, pendientes o vencidas) con filtros.
- **Interesados:**
  - Usuario: quiere saber qué falta por cobrar de sus clientes.
  - Turbine: quiere que cada uno vea solo lo que le corresponde.
- **Precondiciones:** Ninguna adicional.
- **Garantía de éxito:** El usuario ve las facturas que cumplen los filtros, con sus totales.
- **Garantía mínima:** El usuario nunca ve facturas de clientes que no tiene asignados.
- **Disparador:** El usuario quiere revisar los cobros.
- **Escenario principal de éxito:**
  1. El usuario accede al listado de facturas.
  2. El sistema muestra las facturas visibles para él con su estado.
  3. El usuario aplica filtros por fecha, estado, tipo, cliente, responsable, país o moneda.
  4. El sistema muestra las facturas que cumplen los filtros, con sus totales.
  5. El usuario selecciona una factura.
  6. El sistema muestra su detalle y su historial de pagos y envíos.
- **Extensiones:**
  - 4a. No hay resultados: el sistema lo indica.
  - 6a. El usuario quiere el PDF: lo descarga (CU-25).

## CU-10 · Visualización de métricas globales
- **Actor principal:** Administrador
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se accede al cuadro de mando con lo facturado, cobrado y pendiente de toda la empresa, por periodo, con previsión de recurrentes y conversión a moneda base.
- **Interesados:**
  - Administrador: quiere conocer la situación financiera.
  - Dirección de Turbine: quiere que las cifras sean fiables.
- **Precondiciones:** El actor es administrador o superadministrador.
- **Garantía de éxito:** El administrador ve las cifras del periodo y puede llegar al detalle de cada una.
- **Garantía mínima:** Las cifras no se muestran a usuarios sin permiso.
- **Disparador:** El administrador quiere revisar la situación financiera.
- **Escenario principal de éxito:**
  1. El administrador accede al cuadro de mando.
  2. El sistema muestra lo facturado, cobrado y pendiente, y la previsión de recurrentes, en la moneda base.
  3. El administrador elige el periodo (semana, mes, trimestre o año).
  4. El sistema recalcula las cifras.
  5. El administrador selecciona una cifra.
  6. El sistema muestra las facturas que la componen.
- **Extensiones:**
  - 4a. No hay datos en el periodo: el sistema muestra las cifras a cero y lo indica.
  - 6a. El administrador quiere sacar el dato: genera un informe (CU-11).

## CU-11 · Generación y exportación de informes
- **Actor principal:** Administrador
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se generan informes filtrados y se exportan a CSV o PDF. Incluye el paquete mensual o trimestral para la gestoría.
- **Interesados:**
  - Administrador: quiere sacar datos sin pedirlos a nadie.
  - Gestoría: quiere recibir toda la documentación del periodo junta.
- **Precondiciones:** Ninguna adicional.
- **Garantía de éxito:** El informe queda generado y descargable.
- **Garantía mínima:** El informe solo contiene datos que el actor tiene permiso para ver.
- **Disparador:** El administrador necesita un informe o hay que enviar la documentación a la gestoría.
- **Escenario principal de éxito:**
  1. El administrador elige el tipo de informe.
  2. El administrador aplica los filtros y el periodo.
  3. El administrador elige el formato (CSV o PDF).
  4. El sistema genera el informe respetando los permisos.
  5. El administrador lo descarga.
- **Extensiones:**
  - 1a. Paquete para la gestoría:
    - el administrador elige el mes o el trimestre;
    - el sistema reúne en un paquete descargable las facturas emitidas, las rectificativas, los cobros y los PDF del periodo.
  - 4a. El informe es pesado: el sistema lo genera en segundo plano y avisa cuando está listo (CU-35).
  - 4b. No hay datos para los filtros elegidos: el sistema lo indica.

## CU-12 · Envío de la factura al cliente
- **Actor principal:** — (caso incluido en CU-31 y CU-32; lo inicia el caso que lo incluye)
- **Actor secundario:** Servicio de mensajería
- **Ámbito:** Aplicación de facturación
- **Nivel:** Subfunción
- **Descripción:** Se envía al cliente la factura con el PDF y los datos de soporte, por email o por WhatsApp si lo ha consentido.
- **Interesados:**
  - Cliente: quiere recibir la factura en su idioma y saber a quién llamar.
  - Responsable: quiere saber si el envío falla.
- **Precondiciones:** La factura está emitida y tiene su PDF.
- **Garantía de éxito:** La factura llega al cliente y el envío queda registrado.
- **Garantía mínima:** Cada intento queda registrado, y un reenvío nunca duplica la factura.
- **Disparador:** Se emite una factura (CU-31 o CU-32).
- **Escenario principal de éxito:**
  1. El sistema obtiene el contacto de facturación del cliente y su canal (email, o WhatsApp si ha dado su consentimiento).
  2. El sistema prepara el mensaje en el idioma del contacto con la plantilla configurada, el PDF y los datos de soporte.
  3. El sistema lo envía a través del Servicio de mensajería.
  4. El Servicio de mensajería confirma el envío.
  5. El sistema registra el envío en el historial de la factura.
- **Extensiones:**
  - 1a. El cliente no tiene un contacto válido: el sistema no envía y avisa al responsable (CU-35).
  - 1b. El cliente no ha consentido WhatsApp: se usa el email.
  - 4a. El envío rebota o falla: el sistema registra el intento y avisa al responsable. Este corrige el contacto y reenvía la misma factura, sin que se duplique.

## CU-13 · Control de pagos por notificación
- **Actor principal:** Sistema de cobros
- **Actor alternativo:** el Planificador lanza la consulta periódica 
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se recibe un cobro de Stripe (simulado) o Revolut, se asocia a su factura, esta se marca como pagada y se detienen sus recordatorios.
- **Interesados:**
  - Usuario responsable: quiere saber que su cliente ha pagado sin revisarlo a mano.
  - Cliente: no quiere recibir recordatorios después de pagar.
- **Precondiciones:** La conexión con el sistema de cobros está configurada (CU-04).
- **Garantía de éxito:** La factura queda pagada, sus recordatorios se detienen y se avisa al responsable.
- **Garantía mínima:** Un mismo cobro nunca se aplica dos veces. Los cobros dudosos no se asignan solos.
- **Disparador:** El sistema de cobros notifica un pago.
- **Escenario principal de éxito:**
  1. El Sistema de cobros notifica un cobro recibido.
  2. El sistema comprueba que ese evento no se ha procesado ya.
  3. El sistema busca la factura pendiente que corresponde (por referencia, cliente e importe).
  4. El sistema marca la factura como pagada y registra el cobro.
  5. El sistema detiene sus recordatorios.
  6. El sistema avisa al usuario asignado.
- **Extensiones:**
  - 1a. Consulta periódica: el Planificador lanza la consulta, el sistema pide los cobros recientes al Sistema de cobros y continúa en el paso 2 con cada uno.
  - 2a. El evento está duplicado: el sistema lo ignora y lo registra.
  - 3a. No hay una coincidencia clara, o el importe no cuadra: el cobro queda pendiente de revisión manual (CU-14) y se avisa al responsable.

## CU-14 · Registro manual de un pago
- **Actor principal:** Usuario
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se marca a mano una factura como pagada aportando el justificante, o se concilia un cobro que quedó pendiente de revisión.
- **Interesados:**
  - Usuario: quiere reflejar pagos que el sistema no detecta solo.
  - Cliente: no quiere recibir más recordatorios.
  - Gestoría: quiere justificantes de cada cobro.
- **Precondiciones:** La factura está pendiente o vencida.
- **Garantía de éxito:** La factura queda pagada con su justificante y se detienen sus recordatorios.
- **Garantía mínima:** Cada cambio queda trazado y puede deshacerse dejando constancia.
- **Disparador:** El cliente ha pagado por un medio que no se detecta solo, o hay un cobro por conciliar.
- **Escenario principal de éxito:**
  1. El usuario selecciona la factura.
  2. El usuario indica la fecha y el importe del pago y adjunta el justificante.
  3. El sistema valida los datos.
  4. El sistema marca la factura como pagada y registra el pago con el justificante.
  5. El sistema detiene sus recordatorios y avisa al usuario asignado (CU-35).
  6. El sistema lo registra en la auditoría.
- **Extensiones:**
  - 1a. Conciliar un cobro pendiente de revisión: el usuario selecciona el cobro, lo asocia a su factura y continúa en el paso 4.
  - 2a. No hay justificante: el sistema no permite continuar.
  - 3a. El importe no coincide con el de la factura: el sistema lo avisa y pide confirmación.
  - Cualquier paso posterior: se descubre un error y se deshace la conciliación con un motivo. La factura vuelve a quedar pendiente y queda constancia del cambio.

## CU-15 · Desactivación de cuentas
- **Actor principal:** Administrador (con permiso de gestión de usuarios)
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se revoca el acceso de un usuario interno sin borrar el registro de sus acciones.
- **Interesados:**
  - Turbine: quiere que nadie que haya salido pueda seguir entrando.
  - Auditoría: quiere conservar el historial.
- **Precondiciones:** La cuenta está activa.
- **Garantía de éxito:** La cuenta queda desactivada, con sus sesiones cerradas, y sus clientes quedan pendientes de reasignar.
- **Garantía mínima:** El historial de acciones del usuario se conserva.
- **Disparador:** Un empleado deja la empresa o cambia de funciones.
- **Escenario principal de éxito:**
  1. El administrador selecciona el usuario.
  2. El administrador indica que quiere desactivarlo y el motivo.
  3. El sistema muestra sus clientes asignados y sus tareas pendientes.
  4. El administrador confirma.
  5. El sistema desactiva la cuenta, invalida sus sesiones activas y lo registra en la auditoría.
  6. El sistema deja sus clientes pendientes de reasignar (CU-08) y avisa al administrador.
- **Extensiones:**
  - 1a. El administrador intenta desactivar su propia cuenta o la del último superadministrador: el sistema lo impide.
  - 3a. El administrador prefiere reasignar los clientes antes: lo hace (CU-08) y vuelve al paso 4.

## CU-16 · Borrado lógico de clientes y facturas
- **Actor principal:** Administrador
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se marca un cliente como eliminado sin borrar sus datos, que se conservan lo que exija la ley. Solo se borran de verdad las facturas en borrador nunca emitidas.
- **Interesados:**
  - Turbine: quiere cumplir la normativa fiscal y de protección de datos y mantener el histórico comercial.
- **Precondiciones:** Ninguna adicional.
- **Garantía de éxito:** El cliente deja de aparecer en los listados ordinarios y sus datos se conservan.
- **Garantía mínima:** Ninguna factura emitida se borra, y todo borrado deja traza.
- **Disparador:** Un cliente ya no debe aparecer en la operativa diaria.
- **Escenario principal de éxito:**
  1. El administrador selecciona el cliente.
  2. El administrador indica que quiere eliminarlo y el motivo.
  3. El sistema comprueba que el cliente no tiene una suscripción activa.
  4. El administrador confirma.
  5. El sistema marca el cliente como eliminado, conserva sus datos y facturas y lo registra en la auditoría.
- **Extensiones:**
  - 1a. Se quiere borrar una factura en borrador nunca emitida: el sistema la elimina dejando traza.
  - 1b. Se quiere borrar una factura emitida: el sistema no lo permite e indica que se rectifique o anule.
  - 3a. El cliente tiene una suscripción activa: el sistema pide tramitar antes la baja.

## CU-17 · Consulta de facturas pendientes
- **Actor principal:** Usuario
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario (también incluido en CU-19)
- **Descripción:** Se consulta de forma organizada si un cliente tiene deudas pendientes.
- **Interesados:**
  - Usuario: quiere saber qué debe el cliente, por ejemplo antes de tramitar su baja.
- **Precondiciones:** El cliente es visible para el usuario.
- **Garantía de éxito:** El usuario ve las facturas pendientes y vencidas y el total adeudado.
- **Garantía mínima:** No se muestran datos de clientes no asignados.
- **Disparador:** El usuario necesita conocer la deuda de un cliente.
- **Escenario principal de éxito:**
  1. El usuario selecciona el cliente.
  2. El usuario pide ver sus facturas pendientes.
  3. El sistema muestra las facturas pendientes y vencidas, con su importe, su vencimiento y los días de retraso, y el total adeudado.
- **Extensiones:**
  - 3a. No hay deudas: el sistema indica que el cliente está al corriente.

## CU-18 · Modificación de facturas y datos
- **Actor principal:** Usuario
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se edita una factura en borrador. Una factura emitida no se modifica en silencio: los cambios relevantes se hacen con una rectificativa, y en un periodo cerrado no se permiten cambios ordinarios.
- **Interesados:**
  - Usuario: quiere corregir errores antes de emitir.
  - Turbine y Hacienda: quieren que lo emitido no cambie sin rastro.
- **Precondiciones:** La factura es visible para el usuario.
- **Garantía de éxito:** La factura queda modificada, con el PDF regenerado y el cambio registrado.
- **Garantía mínima:** Ninguna factura emitida cambia sus datos fiscales ni sus importes sin rectificativa.
- **Disparador:** El usuario detecta un dato que corregir.
- **Escenario principal de éxito:**
  1. El usuario selecciona la factura.
  2. El sistema comprueba su estado y su periodo.
  3. El usuario modifica los datos.
  4. El sistema valida los datos y muestra una vista previa.
  5. El usuario confirma.
  6. El sistema guarda los cambios, regenera el PDF y registra quién cambió qué y cuándo.
- **Extensiones:**
  - 2a. La factura está emitida: el sistema solo permite cambios que no afecten a sus datos fiscales ni a sus importes, y los deja trazados. Para el resto, indica que se haga una rectificativa.
  - 2b. El periodo está cerrado: el sistema impide el cambio e indica que hace falta reabrirlo.
  - 4a. Un importe manual supera el umbral: queda pendiente de aprobación.

## CU-19 · Alta, baja y edición de clientes
- **Actor principal:** Usuario
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se registran o modifican los datos fiscales de un cliente, sus etiquetas y sus contactos, se reactiva un cliente que vuelve o se tramita su baja.
- **Interesados:**
  - Usuario: quiere tener los datos de sus clientes al día.
  - Cliente: quiere que su baja se haga cuando la pide.
  - Turbine: quiere evitar duplicados y seguir cobrando la deuda.
- **Precondiciones:** Para editar o dar de baja, el cliente está asignado al usuario.
- **Garantía de éxito:** El cliente queda registrado o actualizado, o dado de baja desde la fecha efectiva.
- **Garantía mínima:** Todo cambio queda trazado, y una baja no anula la deuda pendiente.
- **Disparador:** Llega un cliente nuevo, cambian sus datos o pide la baja.
- **Escenario principal de éxito (alta):**
  1. El usuario indica que quiere dar de alta un cliente.
  2. El usuario introduce los datos fiscales (razón social, NIF, dirección, ubicación, sector y rango), las etiquetas y los contactos (nombre, email, teléfono, función, idioma y consentimientos).
  3. El sistema valida los datos y busca duplicados por NIF.
  4. El sistema registra el cliente y lo asigna al usuario que lo crea.
  5. El sistema lo registra en la auditoría.
- **Extensiones:**
  - 1a. Edición: el usuario selecciona un cliente y modifica sus datos. El sistema los valida y los guarda trazando el cambio.
  - 1b. Baja (también a partir de una tarea creada por CU-28):
    - el usuario indica la baja y su fecha efectiva;
    - el sistema cancela la suscripción desde esa fecha y consulta las facturas pendientes;
    - si hay deuda, el sistema la muestra, la baja se registra igualmente y el flujo de impagos continúa.
  - 3a. Hay datos no válidos: el sistema indica cuáles y vuelve al paso 2.
  - 3b. Ya existe un cliente con ese NIF: el sistema avisa y muestra el existente. El usuario cancela o continúa indicando el motivo.
  - 3c. El cliente existente estaba dado de baja: el sistema ofrece reactivarlo conservando su histórico.

## CU-20 · Importación masiva de clientes
- **Actor principal:** Administrador
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se cargan clientes desde un fichero CSV, con mapeo de columnas, previsualización de errores y aviso de NIF duplicados.
- **Interesados:**
  - Administrador: quiere migrar la cartera sin teclearla.
  - Turbine: no quiere datos erróneos ni duplicados.
- **Precondiciones:** Ninguna adicional.
- **Garantía de éxito:** Los clientes válidos quedan importados y se muestra un resumen.
- **Garantía mínima:** No se importa nada sin confirmación, y las filas erróneas no se importan.
- **Disparador:** Hay que cargar una base de datos de clientes.
- **Escenario principal de éxito:**
  1. El administrador sube el fichero CSV.
  2. El sistema lo lee y propone el mapeo de columnas.
  3. El administrador ajusta el mapeo.
  4. El sistema valida cada fila y previsualiza los errores y los NIF duplicados.
  5. El administrador confirma.
  6. El sistema importa las filas válidas, muestra el resumen (importadas, erróneas y duplicadas) y lo registra.
- **Extensiones:**
  - 2a. El fichero no tiene un formato válido: el sistema lo rechaza e informa.
  - 4a. Todas las filas son erróneas: el sistema no permite confirmar.
  - 5a. El administrador cancela: no se importa nada.
 
  ## CU-21 · Gestión del flujo de impagos automatizado
- **Actor principal:** Planificador (tiempo)
- **Actor secundario:** Servicio de mensajería
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se envían recordatorios amistosos de las facturas pendientes:
  - el día 3 y el día 5, o según el vencimiento pactado y el calendario laboral;
  - después del vencimiento, cada 2–3 días hasta el pago, sin cancelar ni sancionar al cliente.
- **Interesados:**
  - Turbine: quiere cobrar sin perseguir a mano.
  - Cliente: quiere recordatorios amables y no sanciones.
  - Responsable: quiere saber cómo va cada seguimiento.
- **Precondiciones:** El calendario de recordatorios está configurado (CU-03).
- **Garantía de éxito:** Cada factura que lo necesita recibe su recordatorio y su responsable queda avisado.
- **Garantía mínima:** No se envían recordatorios de facturas pagadas o en disputa, y cada ejecución queda registrada.
- **Disparador:** Llega la hora programada de cada día laborable.
- **Escenario principal de éxito:**
  1. El Planificador lanza el proceso diario.
  2. El sistema busca las facturas pendientes a las que toca un recordatorio hoy.
  3. Para cada factura, el sistema envía un recordatorio amistoso al contacto, con el PDF y los datos de soporte (Servicio de mensajería).
  4. El sistema registra el envío en la factura.
  5. El sistema avisa al usuario asignado al cliente (CU-35).
  6. El sistema registra el resultado de la ejecución.
- **Extensiones:**
  - 2a. La factura está en disputa o con el seguimiento pausado (CU-34): el sistema la omite.
  - 2b. Hoy no es laborable: el envío pasa al siguiente día laborable.
  - 3a. El envío falla: el sistema lo registra, avisa al responsable y sigue con el resto.

## CU-22 · Auditoría y copias de seguridad
- **Actor principal:** Superadministrador
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se consultan el registro de operaciones, el registro de ejecución de los procesos automáticos y la pantalla de salud, y se gestionan las copias de seguridad y su restauración.
- **Interesados:**
  - Superadministrador: quiere saber quién hizo qué y si el sistema funciona.
  - Turbine: quiere poder recuperarse ante una pérdida de datos.
- **Precondiciones:** El actor es superadministrador.
- **Garantía de éxito:** El superadministrador obtiene la información que busca, o la copia se descarga o restaura.
- **Garantía mínima:** El registro de auditoría no se puede modificar, y toda restauración queda registrada.
- **Disparador:** Hay que revisar una operación, el estado del sistema o una copia.
- **Escenario principal de éxito:**
  1. El superadministrador accede a la auditoría.
  2. El superadministrador filtra por usuario, tipo de operación, entidad o fechas.
  3. El sistema muestra las operaciones con quién las hizo, cuándo, qué cambió y los valores anterior y nuevo.
  4. El superadministrador consulta el detalle de una operación.
- **Extensiones:**
  - 1a. El superadministrador consulta los procesos automáticos: el sistema muestra la última ejecución de cada uno, su resultado y sus errores, y el estado de salud.
  - 1b. Copias de seguridad:
    - el sistema lista las copias;
    - el superadministrador descarga una o pide restaurarla;
    - el sistema pide confirmación, la restaura y lo registra.
  - 3a. No hay resultados: el sistema lo indica.

## CU-23 · Rectificación y anulación de facturas
- **Actor principal:** Administrador
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se crea una factura rectificativa vinculada a la original, se anula una factura cuando legalmente procede o se declara una deuda incobrable.
- **Interesados:**
  - Administrador: quiere corregir lo emitido.
  - Hacienda y gestoría: quieren la numeración correlativa y trazabilidad.
  - Cliente: quiere que se corrija lo que se le cobró mal.
- **Precondiciones:** La factura está emitida.
- **Garantía de éxito:** La rectificativa queda emitida y vinculada, o la factura queda anulada o declarada incobrable.
- **Garantía mínima:** La factura original conserva su número y nunca se borra.
- **Disparador:** Se detecta un error en una factura emitida o una deuda que no se cobrará.
- **Escenario principal de éxito (rectificativa):**
  1. El administrador selecciona la factura emitida.
  2. El administrador elige crear una rectificativa.
  3. El sistema crea un borrador vinculado a la original, copiando sus líneas.
  4. El administrador indica los cambios y el motivo.
  5. El sistema calcula la diferencia y muestra una vista previa.
  6. El administrador confirma.
  7. El sistema solicita la aprobación que corresponda según el importe (CU-30).
  8. Tras la aprobación, el sistema emite la rectificativa con el número de su serie y su PDF, y actualiza el saldo de la original.
- **Extensiones:**
  - 1a. La factura está en borrador: no hace falta rectificativa, porque se edita directamente (CU-18).
  - 2a. Anulación:
    - el administrador indica el motivo;
    - el sistema comprueba que legalmente procede;
    - el sistema anula la factura conservando su número y detiene sus recordatorios.
  - 2b. Deuda incobrable:
    - el administrador indica el motivo y la solicitud queda pendiente de aprobación (CU-30);
    - tras la aprobación, el sistema marca la factura como incobrable y detiene sus recordatorios.
  - 7a. El importe no requiere aprobación: el sistema pasa directamente al paso 8.
  - 7b. La aprobación se rechaza: el borrador no se emite y se avisa al administrador.

## CU-24 · Búsqueda y consulta de clientes
- **Actor principal:** Usuario
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se buscan clientes y facturas por nombre, NIF, contacto, email o número de factura, con filtros y accesos a favoritos y recientes, y se consulta la ficha del cliente.
- **Interesados:**
  - Usuario: quiere encontrar rápido lo que busca.
- **Precondiciones:** Ninguna adicional.
- **Garantía de éxito:** El usuario ve la ficha del cliente con su saldo, su historial y su línea temporal.
- **Garantía mínima:** Los resultados solo incluyen lo que el usuario puede ver.
- **Disparador:** El usuario necesita información de un cliente o una factura.
- **Escenario principal de éxito:**
  1. El usuario introduce un texto de búsqueda.
  2. El sistema muestra los clientes y las facturas que coinciden.
  3. El usuario aplica filtros por ubicación, sector, estado o etiquetas.
  4. El usuario selecciona un cliente.
  5. El sistema muestra su ficha: datos, contactos, saldo, historial de facturas y pagos y línea temporal de eventos.
- **Extensiones:**
  - 1a. El usuario accede desde sus favoritos o recientes y continúa en el paso 5.
  - 2a. No hay resultados: el sistema lo indica.
  - 5a. El usuario marca el cliente como favorito.

## CU-25 · Descarga de factura en PDF
- **Actor principal:** Usuario
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se descarga el PDF de una factura, idéntico al enviado al cliente y al guardado.
- **Interesados:**
  - Usuario: quiere reenviar o archivar la factura.
  - Cliente: quiere que su copia coincida con la de Turbine.
- **Precondiciones:** La factura está emitida y es visible para el usuario.
- **Garantía de éxito:** El usuario obtiene el PDF original.
- **Garantía mínima:** El PDF no se regenera con datos distintos de los emitidos.
- **Disparador:** El usuario necesita el PDF de una factura.
- **Escenario principal de éxito:**
  1. El usuario selecciona la factura.
  2. El usuario pide descargar el PDF.
  3. El sistema entrega el PDF guardado.
- **Extensiones:**
  - 3a. El PDF no está disponible: el sistema informa del error y lo registra.

## CU-26 · Generación de copias de seguridad
- **Actor principal:** Planificador (tiempo)
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se generan periódicamente copias de seguridad de los datos y los movimientos del sistema.
- **Interesados:**
  - Turbine: no quiere perder datos de facturación.
- **Precondiciones:** La periodicidad de las copias está configurada.
- **Garantía de éxito:** Hay una copia nueva, completa y registrada.
- **Garantía mínima:** Si la copia falla, se avisa y se conservan las anteriores.
- **Disparador:** Llega la hora programada.
- **Escenario principal de éxito:**
  1. El Planificador lanza la copia.
  2. El sistema genera la copia de los datos y los movimientos.
  3. El sistema comprueba que la copia está completa.
  4. El sistema la guarda y registra la ejecución.
- **Extensiones:**
  - 2a, 3a. La copia falla o está incompleta: el sistema registra el error y avisa al superadministrador (CU-35).
    
## CU-27 · Pausa y reactivación de suscripción
- **Actor principal:** Usuario
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se pausa una suscripción con motivo y fechas. Durante la pausa no se generan mensualidades, pero las facturas antiguas siguen su curso.
- **Interesados:**
  - Cliente: no quiere pagar mientras no usa el servicio.
  - Turbine: quiere seguir cobrando lo ya facturado.
- **Precondiciones:** El cliente tiene una suscripción activa.
- **Garantía de éxito:** La suscripción queda pausada entre las fechas indicadas y se avisa al responsable para reactivarla.
- **Garantía mínima:** Las facturas anteriores a la pausa siguen su flujo de cobro.
- **Disparador:** El cliente pide pausar el servicio.
- **Escenario principal de éxito:**
  1. El usuario selecciona la suscripción del cliente.
  2. El usuario indica el motivo, la fecha de inicio y la fecha prevista de reactivación.
  3. El sistema valida las fechas.
  4. El usuario confirma.
  5. El sistema registra la pausa. Desde la fecha de inicio, la generación mensual la omite (CU-32).
  6. Al llegar la fecha prevista, el sistema avisa al responsable (CU-35).
  7. El usuario reactiva la suscripción y el sistema vuelve a incluirla en la generación mensual.
- **Extensiones:**
  - 3a. Las fechas no son válidas: el sistema informa y vuelve al paso 2.
  - 7a. El cliente no quiere volver: el usuario tramita la baja (CU-19).
  - Cualquier paso posterior al 5: el usuario reactiva la suscripción antes de tiempo y continúa en el paso 7.

## CU-28 · Notificación de baja por API
- **Actor principal:** Sistema externo
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Otro sistema de Turbine notifica que un cliente solicita la baja, y la aplicación crea una tarea para su responsable.
- **Interesados:**
  - Turbine: quiere que las bajas pedidas por otros canales no se pierdan.
  - Responsable: quiere enterarse a tiempo.
- **Precondiciones:** El sistema externo tiene credenciales de acceso a la API.
- **Garantía de éxito:** Queda creada una tarea de baja para el responsable, que la tramita en CU-19.
- **Garantía mínima:** Las peticiones no autenticadas se rechazan y todas quedan registradas.
- **Disparador:** Un cliente pide la baja fuera de la aplicación.
- **Escenario principal de éxito:**
  1. El Sistema externo envía la solicitud de baja con el identificador del cliente y la fecha.
  2. El sistema autentica la petición.
  3. El sistema localiza el cliente.
  4. El sistema crea la tarea de baja y avisa al responsable (CU-35).
  5. El sistema confirma la recepción al Sistema externo.
- **Extensiones:**
  - 2a. La petición no está autenticada: el sistema la rechaza y la registra.
  - 3a. El cliente no existe: el sistema responde con un error.
  - 4a. Ya existe una tarea de baja para ese cliente: el sistema no la duplica.

## CU-29 · Solicitud de excepción comercial
- **Actor principal:** Usuario
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Se solicita para un cliente un descuento temporal, un mes gratuito o un precio pactado, con fecha de fin y sujeto a aprobación.
- **Interesados:**
  - Usuario: quiere cerrar un acuerdo con el cliente.
  - Turbine: quiere controlar los descuentos.
- **Precondiciones:** El cliente tiene una suscripción.
- **Garantía de éxito:** La excepción aprobada se aplica en la generación mensual hasta su fecha de fin y luego deja de aplicarse sola.
- **Garantía mínima:** Ninguna excepción se aplica sin aprobación.
- **Disparador:** Se ha negociado una condición especial con el cliente.
- **Escenario principal de éxito:**
  1. El usuario selecciona el cliente.
  2. El usuario indica el tipo de excepción, su valor, el motivo y la fecha de fin.
  3. El sistema valida los datos.
  4. El sistema deja la excepción pendiente de aprobación (CU-30).
  5. Tras la aprobación, el sistema la aplica en la generación mensual (CU-32) hasta su fecha de fin.
- **Extensiones:**
  - 3a. Falta la fecha de fin o el valor no es válido: el sistema informa y vuelve al paso 2.
  - 4a. La aprobación se rechaza: el sistema avisa al solicitante con el comentario.

## CU-30 · Gestión de aprobaciones
- **Actor principal:** Administrador
- **Ámbito:** Aplicación de facturación
- **Nivel:** Objetivo de usuario
- **Descripción:** Desde una bandeja se aprueban o rechazan con comentario las operaciones sensibles:
  - importes manuales y descuentos;
  - vencimientos fuera de lo habitual;
  - facturas de importe alto;
  - cambios de rango;
  - rectificativas e incobrables.
- **Interesados:**
  - Solicitante: quiere una respuesta rápida.
  - Turbine: quiere que nadie apruebe sus propias operaciones.
- **Precondiciones:** Hay solicitudes pendientes para el actor.
- **Garantía de éxito:** La solicitud queda aprobada o rechazada, la operación se aplica si procede y se avisa al solicitante.
- **Garantía mínima:** Ninguna operación sensible se aplica sin una aprobación registrada.
- **Disparador:** Llega una solicitud de aprobación (CU-05, CU-06, CU-18, CU-23, CU-29 o CU-31).
- **Escenario principal de éxito:**
  1. El administrador accede a la bandeja de aprobaciones.
  2. El sistema muestra las solicitudes pendientes, con su solicitante, tipo, importe y motivo.
  3. El administrador selecciona una solicitud.
  4. El administrador la aprueba o la rechaza con un comentario.
  5. El sistema registra la decisión, aplica la operación si se ha aprobado y avisa al solicitante (CU-35).
- **Extensiones:**
  - 2a. La solicitud supera el umbral superior: solo puede aprobarla un superadministrador.
  - 3a. La solicitud es del propio administrador: el sistema no le permite resolverla.
  - 4a. El administrador la reasigna a un suplente: el sistema la mueve a la bandeja del suplente y le avisa.

---------------
## Desactualizados

| # | Caso de uso | Actor | Descripción | 
|---|---|---|---| 
| 1 | Login | Superadministrador, administrador, usuario | Permite acceder de forma segura al sistema validando credenciales y comprobando que el correo electrónico pertenezca a un dominio corporativo válido de la empresa. |
| 2 | Registro de usuarios internos| Superadministrador | Permite registrar un nuevo usuario interno en la aplicación asignándole su rol correspondiente para el control de accesos. |
| 3 | Configuración global del sistema | Superadministrador | 	Modifica los parámetros globales: plazo estándar de pago (5 días), calendario de recordatorios (días 3 y 5 y, después, cada 2–3 días), datos de soporte que aparecen en las notificaciones y plantilla corporativa del PDF. Solo puede hacerlo este rol. |
| 4 | Administración de la cuenta de recepción de pagos | Superadministrador | 	Configura la conexión simulada con Stripe y, opcionalmente, con la cuenta de Revolut, de las que el sistema lee los cobros. |
| 5 | Revisión anual del rango de facturación | Superadministrador, administrador | Revisa los niveles de facturación anual de los clientes para actualizar importes y notificar cambios de rango o plan de suscripción. Así como administra y ajusta las condiciones o excepciones extraordinarias aplicadas sobre los cobros y pagos previstos. |
| 6 | Activación de la facturación recurrente | Superadministrador, administrador | Marca un cliente como recurrente e indica su cuota mensual. Si el alta es a mitad de mes, se emite una factura puntual prorrateada por los días restantes y la cuota normal desde el día 1 siguiente. |
| 7 | Asignación de grupos de usuarios a administradores | Superadministrador | Organiza y distribuye a los usuarios disponibles en grupos que serán regulados por un administrador de forma optimizada a la carga de trabajo. |
| 8 | Asignación de clientes a usuario | Superadministrador | Organiza y distribuye a los usuarios disponibles en los grupos para que cada usuario tenga asignado a 1 cliente o mas. |
| 9 | Visualización de pagos e impagos | Superadministrador, administrador, usuario | Consulta el estado financiero de los cobros, identificando facturas pagadas, pendientes o vencidas según la asignación de clientes. |
| 10 | Visualización de métricas globales | Superadministrador, administrador | Accede al cuadro de mando con lo facturado, lo cobrado y lo pendiente de toda la empresa, con desglose semanal, mensual, trimestral y anual. Solo para administración. |
| 11 | Visualización de métricas por equipos | Superadministrador, administrador,  | Accede a cuadros de mando protegidos con desgloses semanales, mensuales, trimestrales y anuales de ingresos del equipo de usuarios que tenga asignado. |
| 12 | Envío de la factura al cliente | Servicio de mensajería | Envía automáticamente al cliente, por email o WhatsApp, la factura con el PDF y los datos de soporte (email y teléfono). |
| 13 | Control de pagos por notificación |  Superadministrador, administrador, usuario, sistema de cobros | Consulta periódicamente los cobros en Stripe (simulado) o Revolut, los asocia a las facturas pendientes, las marca como pagadas, detiene sus recordatorios y avisa al usuario asignado. |
| 14 | Registro manual de un pago | Superadministrador, administrador, usuario | Marca a mano una factura como pagada o cambia su estado. Se detienen sus recordatorios y se avisa al usuario asignado. |
| 15 | Borrado de cuentas | Superadministrador | Elimina o revoca el acceso de usuarios internos de la aplicación por motivos de seguridad o cambios organizativos. |
| 16 | Borrado lógico de clientes y facturas | Superadministrador, administrador | Marca un cliente como eliminado sin borrar sus datos, que se conservan el máximo que permita la ley. Cumple la normativa fiscal y de protección de datos y mantiene el histórico comercial. Las facturas no se borran: se anulan. |
| 17 | Consulta de facturas pendientes |  Superadministrador, administrador, usuario | Permite  consultar de forma automatizada y organizada si un cliente tiene deudas pendientes para gestionar bloqueos de baja. |   
| 18 | Modificación de facturas y datos | Superadministrador, administrador, usuario  | Edita una factura mientras su trimestre natural no esté cerrado (por ejemplo, una factura del 15 de marzo no se puede tocar desde el 1 de abril). Regenera el PDF y registra el cambio. Los datos del cliente se editan en el caso 19 y no tienen esta restricción. |
| 19 | Alta, baja y edición de clientes | Superadministrador, administrador | Registra o modifica en cualquier momento los datos fiscales de la empresa (razón social, NIF, dirección, ubicación, sector, rango de facturación) y los de la persona de contacto (nombre, email, teléfono). Al pedir la baja se cancela de inmediato la recurrencia y se comprueba si hay facturas pendientes (caso 17). Sin deudas, la baja es inmediata. Con deudas, queda como "baja solicitada", se avisa al cliente, el flujo de impago continúa y la baja se completa cuando se paga la última factura. |
| 20 | Importación masiva de clientes | Superadministrador | Carga de forma ágil bases de datos masivas de clientes mediante ficheros estructurados en formato CSV. |
| 21 | Gestión del flujo de impagos automatizado | Servicio de mensajería | Envía al cliente un recordatorio el día 3 y otro el día 5. Si la factura vence sin pagarse, envía recordatorios amistosos cada 2–3 días hasta el pago, sin cancelar ni sancionar al cliente. En cada seguimiento avisa al usuario asignado al cliente (caso 28), no siempre al administrador. |
| 22 | Auditoría y copias de seguridad | Superadministrador | Consulta el registro exhaustivo de operaciones del sistema y gestiona los respaldos periódicos de seguridad y datos. |
| 23 | Anulación de facturas | Superadministrador, administrador | Anula una factura errónea mientras su trimestre esté abierto. Conserva su número y detiene sus recordatorios. Las facturas no se borran porque la numeración debe ser correlativa. |
| 24 | Búsqueda y consulta de clientes | Superadministrador, administrador, usuario | Busca y filtra clientes por ubicación, sector y estado, y consulta su ficha con el historial de facturas y pagos. |
| 25 | Descarga de factura en PDF | Superadministrador, administrador, usuario | Descarga el PDF de una factura, que es idéntico al enviado al cliente y al guardado. |
| 26 | Generación de copias de seguridad | Sistema interno | Genera periódicamente copias de seguridad de los datos y movimientos del sistema. |
-----
