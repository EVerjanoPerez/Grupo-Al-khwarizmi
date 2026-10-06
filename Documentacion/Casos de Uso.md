# Casos de uso

Relación de casos de uso y actores que pueden realizarlos.

| # | Caso de uso | Actor | Descripción | 
|---|---|---|---| 
| 1 | Login | Todos | Permite acceder de forma segura al sistema validando credenciales y comprobando que el correo electrónico pertenezca a un dominio corporativo válido de la empresa. |
| 2 | Registro de usuarios internos| Administrador, Superadministrador | Permite registrar un nuevo usuario interno en la aplicación asignándole su rol correspondiente para el control de accesos. |
| 3 | Configuración global del sistema | Superadministrador | Modifica parámetros globales y ajustes profundos de la plataforma, como alterar el plazo estándar de pago de los 5 días. |
| 4 | Administración de la cuenta de recepción de pagos | Superadministrador | Configura y gestiona las cuentas bancarias o pasarelas simuladas (Stripe, Revolut) del lado de la empresa. |
| 5 | Administración de alteraciones en el pago | Superadministrador | Revisa los niveles de facturación anual de los clientes para actualizar importes y notificar cambios de rango o plan de suscripción. Así como administra y ajusta las condiciones o excepciones extraordinarias aplicadas sobre los cobros y pagos previstos. |
| 6 | Configuración de pagos (retrasar, adelantar, cambiar, etc.) | Superadministrador | Administra y ajusta las condiciones o excepciones extraordinarias aplicadas sobre los cobros y pagos previstos. |
| 7 | Creación de cuentas | Superadministrador | Habilita accesos exclusivos con privilegios de configuración dependientes del rol. |
| 8 | Asignación de grupos de usuarios a administradores | Superadministrador | Organiza y distribuye a los usuarios disponibles en grupos que serán regulados por un administrador de forma optimizada a la carga de trabajo. |
| 9 | Visualización de pagos e impagos | Usuario, administrador, superadministrador | Consulta el estado financiero de los cobros, identificando facturas pagadas, pendientes o vencidas según la asignación de clientes. |
| 10 | Visualización de métricas globales | Superadministrador | Accede a cuadros de mando protegidos con desgloses semanales, mensuales, trimestrales y anuales de ingresos y proyecciones contables de toda la empresa. |
| 11 | Visualización de métricas por equipos | Administrador, superadministrador | Accede a cuadros de mando protegidos con desgloses semanales, mensuales, trimestrales y anuales de ingresos del equipo de usuarios que tenga asignado. |
| 12 | Visualización de métricas por grupos de clientes | Usuarios, administrador, superadministrador | Accede a cuadros de mando con desgloses semanales, mensuales, trimestrales y anuales de ingresos de los clientes que tengan asignados. |
| 13 | Control de pagos por notificación |  Usuarios, administrador, superadministrador, sistema | Verifica de forma automatizada o manual la confirmación de abonos recibidos desde las pasarelas simuladas. |
| 14 | Generación de facturas | Usuarios, administrador, superadministrador | Emite facturas puntuales o programa recurrentes automáticas (día 1) con cálculo de prorrateo y formato PDF idéntico y consistente. |
| 15 | Borrado de cuentas | Superadministrador | Elimina o revoca el acceso de usuarios internos de la aplicación por motivos de seguridad o cambios organizativos. |
| 16 | Borrado lógico de clientes y facturas | Superadministrador | Aplica un borrado lógico que garantiza el cumplimiento de la ley fiscal y de protección de datos mientras retiene el histórico comercial. |
| 17 | Consulta de facturas pendientes |  Usuarios, administrador, superadministrador | Permite  consultar de forma automatizada y organizada si un cliente tiene deudas pendientes para gestionar bloqueos de baja. |   
| 18 | Modificación de facturas y datos | Administrador, superadministrador | Edita datos de clientes o facturas siempre y cuando el trimestre natural correspondiente no esté cerrado ni vencido. |
| 19 | Alta, baja y edición de clientes | Superadministrador | Registra o modifica los datos corporativos, fiscales y de contacto de la entidad y persona física asociada. |
| 20 | Importación masiva de clientes | Superadministrador | Carga de forma ágil bases de datos masivas de clientes mediante ficheros estructurados en formato CSV. |
| 21 | Gestión del flujo de impagos automatizado | Sistema |  Envía recordatorios periódicos (cada 2-3 días) ante facturas vencidas y notifica internamente al administrador asignado |
| 22 | Auditoría y copias de seguridad | Superadministrador | Consulta el registro exhaustivo de operaciones del sistema y gestiona los respaldos periódicos de seguridad y datos. |

