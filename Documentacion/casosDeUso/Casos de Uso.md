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