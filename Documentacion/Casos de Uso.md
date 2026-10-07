# Casos de uso

## Actores 

| Actor | Tipo | Descripción |
|---|---|---|
| Lector | Primario | Empleado de Turbine. Crea, consulta y edita solo los clientes y facturas que tiene asignados. |
| Administrador | Primario | Puede hacer todo lo del Lector sobre todos los clientes. Además gestiona usuarios, asigna clientes y consulta el dashboard. |
| Superadministrador | Primario | Puede hacer todo lo del Administrador. Además configura el sistema y consulta la auditoría y las copias de seguridad. |
| Sistema de cobros | Secundario | Stripe (simulado) y, opcionalmente, Revolut. |
| Sistema interno | Primarío | Servidor donde se aloja la aplicación. |
| Servicio de mensajería | Secundario | Email y WhatsApp. |

*TODO: Actor tiempo*: Hay algunos casos de usos que pueden depender del tiempo, en ese caso es posible hacer un actor que sea tiempo, pero me gustaría que se decidiera con el equipo

## Tabla de casos de uso

Relación de casos de uso y actores que pueden realizarlos.

| # | Caso de uso | Actor | Descripción | 
|---|---|---|---| 
| 1 | Login | Superadministrador, administrador, lector | Permite acceder de forma segura al sistema validando credenciales y comprobando que el correo electrónico pertenezca a un dominio corporativo válido de la empresa. |
| 2 | Registro de usuarios internos| Superadministrador | Permite registrar un nuevo usuario interno en la aplicación asignándole su rol correspondiente para el control de accesos. |
| 3 | Configuración global del sistema | Superadministrador | 	Modifica los parámetros globales: plazo estándar de pago (5 días), calendario de recordatorios (días 3 y 5 y, después, cada 2–3 días), datos de soporte que aparecen en las notificaciones y plantilla corporativa del PDF. Solo puede hacerlo este rol. |
| 4 | Administración de la cuenta de recepción de pagos | Superadministrador | 	Configura la conexión simulada con Stripe y, opcionalmente, con la cuenta de Revolut, de las que el sistema lee los cobros. |
| 5 | Revisión anual del rango de facturación | Superadministrador, administrador | Revisa los niveles de facturación anual de los clientes para actualizar importes y notificar cambios de rango o plan de suscripción. Así como administra y ajusta las condiciones o excepciones extraordinarias aplicadas sobre los cobros y pagos previstos. |
| 6 | Activación de la facturación recurrente | Superadministrador, administrador | Marca un cliente como recurrente e indica su cuota mensual. Si el alta es a mitad de mes, se emite una factura puntual prorrateada por los días restantes y la cuota normal desde el día 1 siguiente. |
| 7 | Asignación de grupos de usuarios a administradores | Superadministrador | Organiza y distribuye a los usuarios disponibles en grupos que serán regulados por un administrador de forma optimizada a la carga de trabajo. |
| 8 | Asignación de clientes a lectores | Superadministrador | Organiza y distribuye a los usuarios disponibles en los grupos para que cada lector tenga asignado a 1 cliente o mas. |
| 9 | Visualización de pagos e impagos | Superadministrador, administrador, lector | Consulta el estado financiero de los cobros, identificando facturas pagadas, pendientes o vencidas según la asignación de clientes. |
| 10 | Visualización de métricas globales | Superadministrador, administrador | Accede al cuadro de mando con lo facturado, lo cobrado y lo pendiente de toda la empresa, con desglose semanal, mensual, trimestral y anual. Solo para administración. |
| 11 | Visualización de métricas por equipos | Superadministrador, administrador,  | Accede a cuadros de mando protegidos con desgloses semanales, mensuales, trimestrales y anuales de ingresos del equipo de usuarios que tenga asignado. |
| 12 | Envío de la factura al cliente | Servicio de mensajería | Envía automáticamente al cliente, por email o WhatsApp, la factura con el PDF y los datos de soporte (email y teléfono). |
| 13 | Control de pagos por notificación |  Superadministrador, administrador, lector, sistema de cobros | Consulta periódicamente los cobros en Stripe (simulado) o Revolut, los asocia a las facturas pendientes, las marca como pagadas, detiene sus recordatorios y avisa al usuario asignado. |
| 14 | Registro manual de un pago | Superadministrador, administrador, lector | Marca a mano una factura como pagada o cambia su estado. Se detienen sus recordatorios y se avisa al usuario asignado. |
| 15 | Borrado de cuentas | Superadministrador | Elimina o revoca el acceso de usuarios internos de la aplicación por motivos de seguridad o cambios organizativos. |
| 16 | Borrado lógico de clientes y facturas | Superadministrador, administrador | Marca un cliente como eliminado sin borrar sus datos, que se conservan el máximo que permita la ley. Cumple la normativa fiscal y de protección de datos y mantiene el histórico comercial. Las facturas no se borran: se anulan. |
| 17 | Consulta de facturas pendientes |  Superadministrador, administrador, lector | Permite  consultar de forma automatizada y organizada si un cliente tiene deudas pendientes para gestionar bloqueos de baja. |   
| 18 | Modificación de facturas y datos | Superadministrador, administrador, lector  | Edita una factura mientras su trimestre natural no esté cerrado (por ejemplo, una factura del 15 de marzo no se puede tocar desde el 1 de abril). Regenera el PDF y registra el cambio. Los datos del cliente se editan en el caso 19 y no tienen esta restricción. |
| 19 | Alta, baja y edición de clientes | Superadministrador, administrador | Registra o modifica en cualquier momento los datos fiscales de la empresa (razón social, NIF, dirección, ubicación, sector, rango de facturación) y los de la persona de contacto (nombre, email, teléfono). Al pedir la baja se cancela de inmediato la recurrencia y se comprueba si hay facturas pendientes (caso 17). Sin deudas, la baja es inmediata. Con deudas, queda como "baja solicitada", se avisa al cliente, el flujo de impago continúa y la baja se completa cuando se paga la última factura. |
| 20 | Importación masiva de clientes | Superadministrador | Carga de forma ágil bases de datos masivas de clientes mediante ficheros estructurados en formato CSV. |
| 21 | Gestión del flujo de impagos automatizado | Servicio de mensajería | Envía al cliente un recordatorio el día 3 y otro el día 5. Si la factura vence sin pagarse, envía recordatorios amistosos cada 2–3 días hasta el pago, sin cancelar ni sancionar al cliente. En cada seguimiento avisa al usuario asignado al cliente (caso 28), no siempre al administrador. |
| 22 | Auditoría y copias de seguridad | Superadministrador | Consulta el registro exhaustivo de operaciones del sistema y gestiona los respaldos periódicos de seguridad y datos. |
| 23 | Anulación de facturas | Superadministrador, administrador | Anula una factura errónea mientras su trimestre esté abierto. Conserva su número y detiene sus recordatorios. Las facturas no se borran porque la numeración debe ser correlativa. |
| 24 | Búsqueda y consulta de clientes | Superadministrador, administrador, lector | Busca y filtra clientes por ubicación, sector y estado, y consulta su ficha con el historial de facturas y pagos. |
| 25 | Descarga de factura en PDF | Superadministrador, administrador, lector | Descarga el PDF de una factura, que es idéntico al enviado al cliente y al guardado. |
| 26 | Generación de copias de seguridad | Sistema interno | Genera periódicamente copias de seguridad de los datos y movimientos del sistema. |
