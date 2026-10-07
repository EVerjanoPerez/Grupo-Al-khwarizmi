# Diagramas

En esta carpeta se almacenará el código correspondiente a los diagramas UML de casos de uso y, posteriormente, a los diagramas de secuencia. Cualquier información adicional debe quedar reflejada en este readme.

---

### 1. prueba.puml

Este documento contiene un diagrama de **ejemplo** realizado por el profesor en clase.

### 2. ExplicacionClases-Clase0710

Segundo diagra de clase hecho por el profesor a modo de **ejemplo**.

---

## División del trabajo diagrama UML

#### Francis — Usuarios y acceso

1. Login — Acceso y autenticación de usuarios.
2. Registro de usuarios internos — Gestión del alta de nuevos usuarios.
7. Asignación de grupos de usuarios a administradores — Organización de usuarios por administradores.
8. Asignación de clientes a lectores — Distribución de clientes entre lectores.
15. Borrado de cuentas — Gestión de bajas de usuarios.

Todo está relacionado con usuarios, roles, permisos y asignaciones.

#### Marcondes — Administración y configuración

3. Configuración global del sistema — Configuración de parámetros generales.
4. Administración de la cuenta de recepción de pagos — Configuración de Stripe/Revolut.
5. Revisión anual del rango de facturación — Gestión de rangos y condiciones de cobro.
6. Activación de la facturación recurrente — Configuración de cuotas recurrentes.
10. Visualización de métricas globales — Consulta de información global de la empresa.

Casos centrados en la configuración y administración general del sistema.

#### Esperanza — Pagos e impagos

9. Visualización de pagos e impagos — Consulta del estado de los cobros.
13. Control de pagos por notificación — Detección y actualización automática de pagos.
14. Registro manual de un pago — Registro manual de pagos.
17. Consulta de facturas pendientes — Comprobación de deudas pendientes.
21. Gestión del flujo de impagos automatizado — Gestión de recordatorios de facturas vencidas.

Todos forman parte del ciclo de pagos, impagos y seguimiento de deudas.

#### Juan — Clientes y facturas

18. Modificación de facturas y datos — Edición de facturas dentro del plazo permitido.
19. Alta, baja y edición de clientes — Gestión completa de clientes.
20. Importación masiva de clientes — Alta de clientes mediante CSV.
23. Anulación de facturas — Anulación de facturas incorrectas.
24. Búsqueda y consulta de clientes — Búsqueda y consulta de información de clientes.

Se centra en la gestión operativa de clientes y facturas.

#### Alvaro — Documentos, métricas y sistema

11. Visualización de métricas por equipos — Consulta del rendimiento de equipos.
12. Envío de la factura al cliente — Envío de facturas mediante email/WhatsApp.
16. Borrado lógico de clientes y facturas — Eliminación lógica y conservación histórica.
22. Auditoría y copias de seguridad — Consulta de auditoría y gestión de respaldos.
25. Descarga de factura en PDF — Obtención del PDF de una factura.
26. Generación de copias de seguridad — Creación automática de respaldos.

Agrupa los casos relacionados con documentos, métricas específicas, auditoría y funciones internas del sistema.