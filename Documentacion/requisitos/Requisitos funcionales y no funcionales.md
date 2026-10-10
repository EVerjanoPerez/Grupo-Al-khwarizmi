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
- FR6. El sistema permitirá registrar varias personas de contacto por empresa, cada una con su función (contratación, facturación, cobros), idioma preferido, canales y consentimientos.
    - Dependencias: FR5.
- FR6.1. Una persona podrá asociarse a varias empresas.
    - Dependencias: FR6.
- FR6.2. El sistema permitirá marcar un contacto como inactivo sin perder el histórico de los envíos que recibió.
    - Dependencias: FR6.
- FR6.3. El sistema avisará si se intenta eliminar o desactivar el único contacto de facturación de un cliente activo.
    - Dependencias: FR6.
- FR7. El sistema permitirá modificar los datos de clientes y contactos en cualquier momento. Los cambios de datos fiscales quedarán auditados.
    - Dependencias: FR5.
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

