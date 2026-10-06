# Requisitos funcionales y no funcionales

**Proyecto:** Sistema de Administración Interna, Facturación y Cobros (aplicación independiente para la gestión de la startup).

## 1. Requisitos funcionales

### 1.1. Gestión de clientes y datos

- **Registro de clientes:** el sistema debe permitir el alta de nuevos clientes, registrando obligatoriamente tanto a la entidad corporativa (empresa) como a la persona física asociada y sus datos de contacto.
- **Datos mínimos:** empresa (razón social, NIF, dirección fiscal, ubicación, sector/temática, facturación anual o rango de plan, tipo de facturación recurrente o puntual, usuario de Turbine asignado) y contacto (nombre, email, teléfono).
- **Edición en cualquier momento:** los datos de clientes y contactos podrán modificarse en cualquier momento posterior al alta.
- **Importación masiva:** el sistema debe permitir importar bases de datos de clientes de forma ágil mediante ficheros en formato CSV (deseable).
- **Búsqueda, filtrado y visualización:** permitir buscar y filtrar clientes y empresas según su ubicación, su temática de trabajo/sector y su estado actual en el sistema.
- **Estado del cliente:** cada cliente tendrá un estado (activo, baja solicitada, baja) que un usuario autorizado podrá cambiar manualmente.
- **Gestión de bajas de clientes:**
  - **Bajas voluntarias:** un cliente puede solicitar la baja de los servicios en cualquier momento.
  - **Bloqueo por impago:** si el cliente tiene facturas pendientes de pago o en vigor, el sistema debe impedir la baja efectiva hasta que se regularice la situación, manteniendo el flujo de impagos activo. El sistema notificará al cliente que la baja queda pendiente por ese motivo.
  - **Cese de recurrencia:** al registrarse la baja, el sistema cancelará inmediatamente la facturación recurrente futura, tenga o no deudas pendientes.
- **Borrado lógico y retención de datos:** el borrado de los datos de los clientes debe implementarse como un borrado lógico, garantizando el cumplimiento de la ley fiscal y de protección de datos, y permitiendo conservar la información histórica para futuras acciones comerciales o de marketing.
- **Historial del cliente:** desde la ficha de cliente se consultarán todas sus facturas y su historial de pagos.

### 1.2. Gestión de facturación

- **Tipos de facturación:** el sistema debe permitir generar tanto facturas puntuales como facturas recurrentes automatizadas (mensuales).
- **Formato único:** no se distingue entre factura estándar y simplificada. Existe un único formato de factura; la recurrente y la puntual son iguales, solo cambia que la recurrente se reemite cada mes.
- **Programación automática:** al dar de alta un cliente recurrente, el sistema programará su facturación sin intervención manual; las recurrentes se emitirán el día 1 de cada mes.
- **Prorrateo:** si el alta es a mitad de mes (p. ej. día 22), se emitirá ese día una factura proporcional por los días restantes y la tarifa normal desde el día 1 siguiente.
- **Importe:** el importe será editable al crear la factura. En las recurrentes se mantendrá fijo todo el año salvo cambio de rango.
- **Generación y almacenamiento:** generación automática del documento PDF de la factura y almacenamiento seguro de este y de toda la información asociada en la base de datos del sistema.
- **Modificación de facturas:** se podrán editar mientras no haya cerrado su trimestre natural; después quedarán bloqueadas (fecha 15/03 → no editable desde 01/04).
- **Ajustes y chequeos de tarifas anuales:** dado que el modelo de negocio cobra una suscripción mensual basada en el nivel de facturación anual del cliente, el sistema debe contemplar herramientas para realizar chequeos anuales y actualizar los importes correspondientes. El sistema avisará cuando un cliente deba cambiar de rango/plan.
- **Exclusiones regulatorias:** no se implementa Verifactu (lo gestiona el gestor de Turbine).
- **Contenido y diseño del PDF:** numeración correlativa, datos fiscales del emisor y del cliente, IVA, e imagen corporativa de Turbine.
- **Filtros de facturas:** por fecha de emisión, estado (pagada, impagada/vencida) y tipo (recurrente/puntual).
- **Descarga:** las facturas podrán descargarse en PDF por los usuarios con permiso.

### 1.3. Gestión de cobros, pasarela y seguimiento

- **Simulación de pasarela de pago:** en la fase actual no se requiere una pasarela de pago real integrada de forma nativa, pero el sistema debe simular la conexión e integración con herramientas como Stripe y cuentas bancarias (p. ej., Revolut) para verificar automáticamente si los pagos han sido abonados. Solo se contempla el lado de la empresa: no es necesario gestionar los pagos de los clientes a la empresa, únicamente comprobar si se han realizado o no.
- **Registro automático del pago:** al detectarse el pago, la factura pasará a "pagada" sin intervención manual.
- **Registro manual:** un usuario autorizado podrá marcar una factura como pagada o cambiar su estado.
- **Plazos de pago estándar:** el plazo predeterminado de vencimiento de las facturas es de 5 días naturales desde su emisión.
- **Estados de factura:** pendiente, pagada, vencida/impagada, anulada.

### 1.4. Notificaciones y flujo de impagos

- **Emisión y avisos:** notificar al cliente de forma automatizada en el momento en que se genera su nueva factura.
- **Recordatorios preventivos:** enviar notificaciones de recordatorio de pago al cliente de manera anticipada (por ejemplo, un recordatorio el día 3 y otro el día 5, coincidiendo con la fecha de vencimiento).
- **Flujo de impagos automatizado:** si una factura no se abona tras cumplirse el plazo establecido, el sistema debe iniciar un flujo de impagos, enviando recordatorios periódicos al cliente (cada 2 o 3 días) hasta que se efectúe el pago.
- **Fin del flujo:** el flujo se detendrá al detectarse el pago o al cambiar manualmente el estado.
- **Notificaciones internas:** se avisará al usuario de Turbine asignado a ese cliente (administrador o lector) de cada pago confirmado y de cada seguimiento de impago. No es necesario enviarle la factura, que queda guardada.
- ***Notificaciones externas:** el sistema enviará las notificaciones automáticamente por email o WhatsApp. La aplicación no tendrá chat propio ni acceso para clientes.
- **Datos de soporte:** todas las notificaciones al cliente incluirán el email y teléfono de soporte de Turbine.
  

### 1.5. Autenticación, roles, permisos y seguridad

- **Inicio de sesión verificado:** acceso seguro mediante email y contraseña. El sistema debe validar que el correo electrónico introducido sea un email corporativo válido (rechazando correos personales genéricos o de compañías ajenas).
- **Sin auto-registro:** solo el administrador da de alta usuarios internos; los clientes no pueden registrarse.
- **Jerarquía y control de roles:** el sistema debe estructurarse en tres niveles diferenciados de acceso:
  1. **Superadministrador:** cuenta con privilegios de configuración profunda del sistema (capacidad para modificar parámetros globales, como el plazo estándar de pago de 5 días). Esta configuración es exclusiva de este rol.
  2. **Administrador:** posee acceso global y total para dar de alta clientes, generar facturas, gestionar usuarios de la aplicación y consultar todos los informes financieros de la empresa.
  3. **Lector (edición limitada):** consultar, descargar, crear y editar (CRUD) solo en los clientes asignados.
- **Asignación de clientes:** el administrador decide qué clientes ve cada lector, con granularidad por cliente.
- **Protección de datos por rol:** la visualización de datos personales (nombre y contacto) y fiscales (razón social, NIF, dirección) se restringirá según rol y asignación. (Se eliminan los "datos bancarios", que el cliente no mencionó.)
- **Extensibilidad de roles:** el modelo de permisos permitirá añadir funciones de administrador u otros roles más adelante.

### 1.6. Informes financieros (reporting)

- **Cuadros de mando y métricas:** mostrar resúmenes e informes financieros con desglose de datos a nivel semanal, mensual, trimestral y anual (incluyendo sumatorios de lo facturado, estado de ingresos y proyecciones contables). Esta sección debe estar protegida y ser accesible únicamente para los roles de Administrador y Superadministrador.

### 1.7. Auditoría y copias de seguridad 

- **Auditoría:** registro de todas las operaciones (altas, facturas, pagos, cambios de estado, permisos concedidos y autor), consultable por el superadministrador.
- **Copias de seguridad:** copias periódicas de datos y movimientos, accesibles para el superadministrador.

### 1.8. API y documentación

- **API de consulta de pendientes:** permitirá saber si un cliente tiene facturas pendientes (para el proceso de baja que Turbine gestiona externamente).
- **API abierta:** diseñada para consultas futuras de sistemas externos o agentes IA de Turbine.
- **Documentación:** el sistema y la API estarán documentados.

## 2. Requisitos no funcionales

### 2.1. Usabilidad y experiencia de usuario (UX)

- **Extrema rapidez y simplicidad:** la gestión del software debe ser sumamente rápida y fluida. Acciones críticas como dar de alta un cliente, generar una factura recurrente, realizar un chequeo anual o comprobar el estado de un cobro no deben tomar más de 1 minuto, minimizando drásticamente el número de clics requeridos.
- **Diseño minimalista:** la interfaz debe ser limpia y directa, evitando la complejidad excesiva del software de contabilidad comercial tradicional (por ejemplo, Holded).

### 2.2. Portabilidad y accesibilidad

- **Aplicación web multiplataforma:** el sistema debe desarrollarse obligatoriamente como una aplicación web accesible de forma fluida desde cualquier dispositivo (optimizada para ordenadores, tablets y teléfonos móviles), permitiendo trabajar en movilidad.

### 2.3. Independencia tecnológica y arquitectura

- **Aislamiento del producto principal:** el sistema de facturación y administración interna debe ser una aplicación completamente independiente. No debe tocar, depender ni integrarse con el núcleo del producto principal de la compañía (el Brain Operating System).
- **Tecnología:** libre elección; base de datos asequible, preferiblemente libre.
- **Interoperabilidad:** la integración de cobros estará desacoplada para sustituir la simulación por Stripe real sin rediseño.

### 2.4. Consistencia de datos

- **Integridad documental:** el documento PDF de la factura generado automáticamente debe mantener una consistencia absoluta, siendo idéntico independientemente de la sección, dispositivo o plataforma desde donde se consulte, imprima o descargue.
- **Imagen corporativa:** el PDF respetará la estética corporativa de Turbine.

### 2.5. Fiabilidad 

- **Flujo de información sin pérdidas:** ningún dato ni notificación podrá dejar de llegar a la persona correcta (criterio de éxito del cliente).

### 2.6 Escalabilidad y rendimiento

- **Escalabilidad:** el sistema soportará miles de clientes y sus facturas sin rediseño.
- **Rendimiento:** respuesta ágil en cualquier dispositivo.

### 2.7 Seguridad, privacidad y cumplimiento legal

- **Privacidad desde el diseño:** datos fiscales y personales protegidos conforme al RGPD.
- **Autenticación robusta:** doble facto de autenticación; contraseñas almacenadas de forma segura.
- **Validez legal:** facturas conformes a la normativa española e inmutables tras el cierre trimestral.
- **Conservación:** datos de clientes conservados el máximo que permita la ley.

