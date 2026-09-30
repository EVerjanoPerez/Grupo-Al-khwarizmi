# Requisitos funcionales y no funcionales

**Proyecto:** Sistema de Administración Interna, Facturación y Cobros (aplicación independiente para la gestión de la startup).

## 1. Requisitos funcionales

### 1.1. Gestión de clientes y datos

- **Registro de clientes:** el sistema debe permitir el alta de nuevos clientes, registrando obligatoriamente tanto a la entidad corporativa (empresa) como a la persona física asociada y sus datos de contacto.
- **Importación masiva:** el sistema debe permitir importar bases de datos de clientes de forma ágil mediante ficheros en formato CSV (opcional, pero recomendable).
- **Búsqueda, filtrado y visualización:** permitir buscar y filtrar clientes y empresas según su ubicación, su temática de trabajo/sector y su estado actual en el sistema.
- **Gestión de bajas de clientes:**
  - **Bajas voluntarias:** un cliente puede solicitar la baja de los servicios en cualquier momento.
  - **Bloqueo por impago:** si el cliente tiene facturas pendientes de pago o en vigor, el sistema debe impedir la baja efectiva hasta que se regularice la situación, manteniendo el flujo de impagos activo.
  - **Cese de recurrencia:** una vez tramitada la baja de forma exitosa (sin deudas), el sistema debe detener automáticamente la generación de facturas recurrentes futuras.
- **Borrado lógico y retención de datos:** el borrado de los datos de los clientes debe implementarse como un borrado lógico, garantizando el cumplimiento de la ley fiscal y de protección de datos, y permitiendo conservar la información histórica para futuras acciones comerciales o de marketing.

### 1.2. Gestión de facturación

- **Tipos de facturación:** el sistema debe permitir generar tanto facturas puntuales como facturas recurrentes automatizadas (mensuales).
- **Tipos de documentos fiscales:** el sistema debe distinguir entre facturas estándar y facturas simplificadas, manteniendo un formato de PDF de factura consistente e idéntico en cualquier sección donde se visualice o descargue.
- **Generación y almacenamiento:** generación automática del documento PDF de la factura y almacenamiento seguro de este y de toda la información asociada en la base de datos del sistema.
- **Modificación de facturas:** permitir la edición de facturas siempre que estén dentro de su periodo de vigencia actual (por ejemplo, prohibiendo la modificación de facturas de un trimestre o periodo ya cerrado y vencido).
- **Facturación proporcional (prorrateo):** si un cliente se da de alta en una fecha distinta al inicio de mes (p. ej., el día 22), el sistema debe calcular y emitir automáticamente una factura proporcional por los días restantes de ese mes, programando la tarifa normal a partir del día 1 del mes siguiente.
- **Ajustes y chequeos de tarifas anuales:** dado que el modelo de negocio cobra una suscripción mensual basada en el nivel de facturación anual del cliente, el sistema debe contemplar herramientas para realizar chequeos anuales y actualizar los importes correspondientes.
- **Exclusiones regulatorias:** el sistema no requiere implementar el nuevo sistema de la Agencia Tributaria (Hacienda) conocido como VeriFactu.

### 1.3. Gestión de cobros, pasarela y seguimiento

- **Simulación de pasarela de pago:** en la fase actual no se requiere una pasarela de pago real integrada de forma nativa, pero el sistema debe simular la conexión e integración con herramientas como Stripe y cuentas bancarias (p. ej., Revolut) para verificar automáticamente si los pagos han sido abonados. Solo se contempla el lado de la empresa: no es necesario gestionar los pagos de los clientes a la empresa, únicamente comprobar si se han realizado o no.
- **Plazos de pago estándar:** el plazo predeterminado de vencimiento de las facturas es de 5 días naturales desde su emisión.
- **Filtros de facturas:** permitir la búsqueda y filtrado de facturas en función de su fecha de emisión y su estado actual (pagadas, impagadas, recurrentes).

### 1.4. Notificaciones y flujo de impagos

- **Emisión y avisos:** notificar al cliente de forma automatizada en el momento en que se genera su nueva factura.
- **Recordatorios preventivos:** enviar notificaciones de recordatorio de pago al cliente de manera anticipada (por ejemplo, un recordatorio el día 3 y otro el día 5, coincidiendo con la fecha de vencimiento).
- **Flujo de impagos automatizado:** si una factura no se abona tras cumplirse el plazo establecido, el sistema debe iniciar un flujo de impagos, enviando recordatorios periódicos al cliente (cada 2 o 3 días) hasta que se efectúe el pago.
- **Notificaciones internas:**
  - Enviar avisos automáticos al administrador cuando un cliente complete con éxito un pago.
  - Alertar al administrador sobre el estado de los impagos recurrentes y la evolución del cobro.
  - Todas las comunicaciones de soporte deben incluir los datos de contacto (email y teléfono de soporte) de la empresa.
- **Notificaciones externas:**
  - Todas las comunicaciones hacia el cliente deben enviarse mediante aplicaciones externas, como Gmail o WhatsApp, ya que esta aplicación es solo de gestión interna.

### 1.5. Autenticación, roles, permisos y seguridad

- **Inicio de sesión verificado:** acceso seguro mediante email y contraseña. El sistema debe validar que el correo electrónico introducido sea un email corporativo válido (rechazando correos personales genéricos o de compañías ajenas).
- **Jerarquía y control de roles:** el sistema debe estructurarse en tres niveles diferenciados de acceso:
  1. **Superadministrador:** cuenta con privilegios de configuración profunda del sistema (capacidad para modificar parámetros globales, como el plazo estándar de pago de 5 días).
  2. **Administrador:** posee acceso global y total para dar de alta clientes, generar facturas, gestionar usuarios de la aplicación y consultar todos los informes financieros de la empresa.
  3. **Lector (con edición limitada):** puede visualizar, descargar e incluso editar información, pero su ámbito de actuación y visualización está estrictamente limitado a los clientes o expedientes que tenga asignados.
- **Ocultación y protección de datos:** restringir la visualización de los datos personales (contacto de la persona) y fiscales (nombre, DNI/NIF, dirección postal y datos bancarios) de los clientes en función del rol y de los permisos del usuario.

### 1.6. Informes financieros (reporting)

- **Cuadros de mando y métricas:** mostrar resúmenes e informes financieros con desglose de datos a nivel semanal, mensual y anual (incluyendo sumatorios de lo facturado, estado de ingresos y proyecciones contables). Esta sección debe estar protegida y ser accesible únicamente para los roles de Administrador y Superadministrador.

## 2. Requisitos no funcionales

### 2.1. Usabilidad y experiencia de usuario (UX)

- **Extrema rapidez y simplicidad:** la gestión del software debe ser sumamente rápida y fluida. Acciones críticas como dar de alta un cliente, generar una factura recurrente o realizar un chequeo anual no deben tomar más de 1 minuto, minimizando drásticamente el número de clics requeridos.
- **Diseño minimalista:** la interfaz debe ser limpia y directa, evitando la complejidad excesiva del software de contabilidad comercial tradicional (por ejemplo, Holded).

### 2.2. Portabilidad y accesibilidad

- **Aplicación web multiplataforma:** el sistema debe desarrollarse obligatoriamente como una aplicación web accesible de forma fluida desde cualquier dispositivo (optimizada para ordenadores, tablets y teléfonos móviles), permitiendo trabajar en movilidad.

### 2.3. Independencia tecnológica y arquitectura

- **Aislamiento del producto principal:** el sistema de facturación y administración interna debe ser una aplicación completamente independiente. No debe tocar, depender ni integrarse con el núcleo del producto principal de la compañía (el Brain Operating System).

### 2.4. Consistencia de datos

- **Integridad documental:** el documento PDF de la factura generado automáticamente debe mantener una consistencia absoluta, siendo idéntico independientemente de la sección, dispositivo o plataforma desde donde se consulte, imprima o descargue.
