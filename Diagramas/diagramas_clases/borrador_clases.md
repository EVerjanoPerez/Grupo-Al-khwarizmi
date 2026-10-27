# Borrador: Esquema de Clases del Sistema (Turbine)

* **Orientado a Objetos (no Modelo ER):** Cada clase incluye tanto estado (atributos privados `-`) como comportamiento (métodos públicos `+` extraídos de los Casos de Uso).
* **Sin claves foráneas (FK):** Las relaciones entre tablas se modelan mediante referencias de objetos, listas y multiplicidades UML.

---

## 1. Bloque de Usuarios, Accesos y Trabajo Diario
*(Extraído de CU-01, CU-02, CU-07, CU-13, CU-32)*

### Clase `Usuario`
Representa a los empleados internos de Turbine (Usuario, Administrador, Superadministrador).
* **Atributos:**
  * `- idUsuario : int`
  * `- nombre : String`
  * `- correoCorporativo : String`
  * `- passwordHash : String`
  * `- rol : RolUsuario` *(Enum: USUARIO, ADMIN, SUPERADMIN)*
  * `- activo : bool`
  * `- intentosFallidos : int`
* **Métodos:**
  * `+ iniciarSesion(correo : String, pass : String) : bool` *(CU-01)*
  * `+ cerrarSesion() : void` *(CU-01)*
  * `+ recuperarPassword() : void` *(CU-01)*
  * `+ desactivarCuenta(motivo : String) : void` *(CU-13)*

### Clase `DelegacionTemporal`
Permite delegar los clientes de un usuario a un sustituto durante una ausencia.
* **Atributos:**
  * `- idDelegacion : int`
  * `- fechaInicio : Date`
  * `- fechaFin : Date`
  * `- activa : bool`
* **Métodos:**
  * `+ activarDelegacion() : void` *(CU-07)*
  * `+ cancelarDelegacion() : void` *(CU-07)*

### Clase `Notificacion`
Avisos internos en la pantalla de trabajo diario.
* **Atributos:**
  * `- idNotificacion : int`
  * `- mensaje : String`
  * `- fechaCreacion : DateTime`
  * `- leida : bool`
  * `- canal : CanalNotificacion` *(Enum: INTERNO, EMAIL, WHATSAPP)*
* **Métodos:**
  * `+ marcarComoLeida() : void` *(CU-32)*
  * `+ enviar() : bool` *(CU-32)*

---

## 2. Bloque de Configuración, Auditoría y Datos
*(Extraído de CU-03, CU-04, CU-20, CU-24, CU-34)*

### Clase `ConfiguracionSistema`
Parámetros globales modificables por el Superadministrador.
* **Atributos:**
  * `- plazoPagoDefecto : int`
  * `- umbralAprobacionInferior : double`
  * `- umbralAprobacionSuperior : double`
  * `- monedaBase : String`
  * `{static} - IVA_DEFECTO : double`
* **Métodos:**
  * `+ actualizarParametros() : bool` *(CU-03)*
  * `+ validarUmbrales() : bool` *(CU-03)*
  * `+ configurarPasarelaCobro(credenciales : String) : bool` *(CU-04)*

### Clase `RegistroAuditoria`
Trazabilidad inmutable de todas las operaciones críticas del sistema.
* **Atributos:**
  * `- idRegistro : int`
  * `- fechaHora : DateTime`
  * `- tipoOperacion : String`
  * `- entidadAfectada : String`
  * `- valorAnterior : String`
  * `- valorNuevo : String`
* **Métodos:**
  * `+ registrarEvento(u : Usuario, accion : String) : void` *(CU-20)*

### Clase `PeriodoContable`
Control de cierre mensual o trimestral para evitar modificaciones en facturas declaradas.
* **Atributos:**
  * `- idPeriodo : String` *(ej. "2026-Q3")*
  * `- cerrado : bool`
  * `- motivoReapertura : String`
* **Métodos:**
  * `+ cerrarPeriodo(admin : Usuario) : bool` *(CU-34)*
  * `+ reabrirPeriodo(superadmin : Usuario, motivo : String) : bool` *(CU-34)*

---

## 3. Bloque de Clientes y Suscripciones
*(Extraído de CU-05, CU-06, CU-08, CU-14, CU-17, CU-25, CU-26, CU-35, CU-36)*

### Clase `Cliente`
Entidad principal sobre la que gira la facturación.
* **Atributos:**
  * `- idCliente : int`
  * `- razonSocial : String`
  * `- nif : String`
  * `- direccionFiscal : String`
  * `- sector : String`
  * `- rangoTarifa : String`
  * `- eliminadoLogico : bool`
* **Métodos:**
  * `+ darDeAlta() : bool` *(CU-36)*
  * `+ modificarDatosFiscales() : void` *(CU-17)*
  * `+ asignarResponsable(u : Usuario) : void` *(CU-08)*
  * `+ actualizarRangoAnual(nuevaFacturacion : double) : void` *(CU-05)*
  * `+ darDeBaja(fechaEfectiva : Date) : void` *(CU-17)*
  * `+ borradoLogico(motivo : String) : void` *(CU-14)*

### Clase `ContactoCliente`
Persona de contacto dentro de la empresa cliente (separado para cumplir protección de datos CU-35).
* **Atributos:**
  * `- idContacto : int`
  * `- nombre : String`
  * `- email : String`
  * `- telefono : String`
  * `- idioma : String`
  * `- consienteWhatsApp : bool`
* **Métodos:**
  * `+ anonimizarDatos() : void` *(CU-35)*

### Clase `Suscripcion`
Contrato de facturación recurrente asociado a un cliente.
* **Atributos:**
  * `- idSuscripcion : int`
  * `- plan : String`
  * `- cuotaMensual : double`
  * `- diaVencimientoPactado : int`
  * `- fechaInicio : Date`
  * `- fechaFin : Date`
  * `- estado : EstadoSuscripcion` *(Enum: PROGRAMADA, ACTIVA, PAUSADA, CANCELADA)*
* **Métodos:**
  * `+ activar() : void` *(CU-06)*
  * `+ calcularProrrateo(fechaAlta : Date) : double` *(CU-06)*
  * `+ pausar(fechaInicio : Date, fechaReactivacion : Date, motivo : String) : void` *(CU-25)*
  * `+ reactivar() : void` *(CU-25)*
  * `+ aplicarExcepcion(descuento : double, fechaFin : Date) : void` *(CU-26)*

---

## 4. Bloque de Facturación, Presupuestos y Aprobaciones
*(Extraído de CU-10, CU-16, CU-21, CU-23, CU-27, CU-28, CU-29, CU-30)*

### Clase `Presupuesto`
Propuesta comercial previa que puede convertirse en borrador de factura.
* **Atributos:**
  * `- idPresupuesto : String`
  * `- fechaCreacion : Date`
  * `- estado : EstadoPresupuesto` *(Enum: PENDIENTE, ACEPTADO, RECHAZADO)*
  * `- documentoEvidencia : String`
* **Métodos:**
  * `+ calcularTotal() : double` *(CU-30)*
  * `+ aceptar(evidencia : String) : void` *(CU-30)*
  * `+ rechazar(evidencia : String) : void` *(CU-30)*
  * `+ convertirEnFactura() : Factura` *(CU-30)*

### Clase `Factura`
Documento contable (puntual, recurrente o rectificativa).
* **Atributos:**
  * `- numeroFactura : String`
  * `- serie : String`
  * `- fechaEmision : Date`
  * `- fechaVencimiento : Date`
  * `- baseImponible : double`
  * `- totalImpuestos : double`
  * `- importeTotal : double`
  * `- estado : EstadoFactura` *(Enum: BORRADOR, PENDIENTE_APROBACION, EMITIDA, PAGADA, VENCIDA, RECTIFICADA, ANULADA, INCOBRABLE)*
  * `- enDisputa : bool`
  * `- rutaPdf : String`
* **Métodos:**
  * `+ calcularTotales() : void` *(CU-28)*
  * `+ emitir() : bool` *(CU-28, CU-29)*
  * `+ generarPdf() : String` *(CU-23, CU-28)*
  * `+ enviarAlCliente() : bool` *(CU-10)*
  * `+ crearRectificativa(motivo : String) : Factura` *(CU-21)*
  * `+ anular(motivo : String) : bool` *(CU-21)*
  * `+ declararIncobrable(motivo : String) : void` *(CU-21)*

### Clase `LineaFactura`
Detalle de cada concepto facturado (Composición fuerte con `Factura` y `Presupuesto`).
* **Atributos:**
  * `- numeroLinea : int`
  * `- concepto : String`
  * `- cantidad : int`
  * `- precioUnitario : double`
  * `- porcentajeDescuento : double`
  * `- porcentajeImpuesto : double`
* **Métodos:**
  * `+ calcularSubtotal() : double` *(CU-28)*

### Clase `SolicitudAprobacion`
Bandeja de control para operaciones que superan umbrales o requieren validación de un Administrador.
* **Atributos:**
  * `- idSolicitud : int`
  * `- tipoOperacion : String` *(ej. Cambio rango, Excepción, Factura alta, Incobrable)*
  * `- importeAfectado : double`
  * `- motivoSolicitud : String`
  * `- comentarioResolucion : String`
  * `- estado : EstadoAprobacion` *(Enum: PENDIENTE, APROBADA, RECHAZADA)*
* **Métodos:**
  * `+ aprobar(aprobador : Usuario, comentario : String) : bool` *(CU-27)*
  * `+ rechazar(aprobador : Usuario, comentario : String) : void` *(CU-27)*

---

## 5. Bloque de Cobros, Impagos e Incidencias
*(Extraído de CU-09, CU-11, CU-12, CU-15, CU-19, CU-31)*

### Clase `Cobro`
Registro de un pago recibido (automático por Stripe/Revolut o registrado manualmente).
* **Atributos:**
  * `- idCobro : int`
  * `- referenciaExterna : String`
  * `- fechaCobro : Date`
  * `- importe : double`
  * `- origen : String` *(Stripe, Revolut, Manual)*
  * `- justificanteAdjunto : String`
  * `- conciliado : bool`
* **Métodos:**
  * `+ procesarCobroAutomatico() : bool` *(CU-11)*
  * `+ conciliarManualmente(f : Factura, justificante : String) : bool` *(CU-12)*
  * `+ deshacerConciliacion(motivo : String) : void` *(CU-12)*

### Clase `RecordatorioImpago`
Avisos amistosos enviados automáticamente cuando una factura está próxima a vencer o vencida.
* **Atributos:**
  * `- idRecordatorio : int`
  * `- fechaEnvio : DateTime`
  * `- numeroIntento : int`
  * `- enviadoConExito : bool`
* **Métodos:**
  * `+ enviarRecordatorio() : bool` *(CU-19)*

### Clase `Incidencia`
Ticket interno o disputa abierta sobre un cliente o una factura (detiene los recordatorios).
* **Atributos:**
  * `- idIncidencia : int`
  * `- descripcion : String`
  * `- prioridad : String`
  * `- esDisputaFactura : bool`
  * `- fechaRevision : Date`
  * `- resuelta : bool`
* **Métodos:**
  * `+ abrirIncidencia() : void` *(CU-31)*
  * `+ pausarSeguimientoImpago(f : Factura) : void` *(CU-19, CU-31)*
  * `+ agregarComentario(u : Usuario, texto : String) : void` *(CU-31, CU-32)*
  * `+ resolverYCerrar(resolucion : String) : void` *(CU-31)*

---

## 6. Resumen de Relaciones Principales (UML)
1. **Composición (`*--`, rombo negro - ciclo de vida dependiente):**
   * `Factura "1" *-- "1..*" LineaFactura` *(Si se borra un borrador de factura, desaparecen sus líneas).*
   * `Presupuesto "1" *-- "1..*" LineaFactura`
   * `Cliente "1" *-- "1..*" ContactoCliente`
2. **Agregación / Asociación (`o--` / `-->` - ciclo de vida independiente):**
   * `Usuario "1" --> "0..*" Cliente : -clientesAsignados` *(Un empleado gestiona varios clientes).*
   * `Cliente "1" --> "0..*" Suscripcion : -suscripciones`
   * `Cliente "1" --> "0..*" Factura : -facturas`
   * `Factura "1" --> "0..*" Cobro : -cobros`
   * `Factura "1" --> "0..*" RecordatorioImpago : -recordatorios`
   * `Factura "0..1" <-- "0..1" Factura : -facturaOriginal / -rectificativa` *(Asociación reflexiva para el CU-21).*
3. **Dependencia (`..>`, línea discontinua - relación de uso):**
   * `Factura ..> PeriodoContable : <<uses>>` *(Comprueba si el periodo está abierto antes de editar, CU-16/CU-34).*
   * `Factura ..> ConfiguracionSistema : <<uses>>` *(Lee los impuestos y series vigentes al emitirse, CU-03/CU-28).*
