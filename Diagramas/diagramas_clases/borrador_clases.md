# Borrador: Esquema de Clases del Sistema (Turbine)

* **Orientado a Objetos (no Modelo ER):** Cada clase incluye tanto estado (atributos privados `-`) como comportamiento (métodos públicos `+` extraídos de los Casos de Uso).
* **Sin claves foráneas (FK):** Las relaciones entre clases se modelan mediante referencias de objetos, listas y multiplicidades UML.

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
  * `- puedeGestionarUsuarios : bool` *(Permiso del Administrador, CU-02/CU-13)*
  * `- activo : bool`
  * `- intentosFallidos : int`
  * `- bloqueadoHasta : DateTime` *(Bloqueo temporal tras varios intentos, CU-01)*
  * `- canalesPreferidos : List<CanalNotificacion>`
* **Métodos:**
  * `+ iniciarSesion(correo : String, pass : String) : bool` *(CU-01)*
  * `+ cerrarSesion() : void` *(CU-01)*
  * `+ recuperarPassword() : void` *(CU-01)*
  * `+ enviarInvitacion() : bool` *(CU-02)*
  * `+ reactivarCuenta() : void` *(CU-02)*
  * `+ desactivarCuenta(motivo : String) : void` *(CU-13)*
  * `+ configurarCanales(canales : List<CanalNotificacion>) : void` *(CU-32)*

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
  * `+ estaVigente(fecha : Date) : bool` *(CU-07, la delegación caduca sola)*

### Clase `Notificacion`
Avisos internos en la pantalla de trabajo diario.
* **Atributos:**
  * `- idNotificacion : int`
  * `- mensaje : String`
  * `- fechaCreacion : DateTime`
  * `- estado : EstadoNotificacion` *(Enum: NO_LEIDA, LEIDA, ATENDIDA)*
  * `- canal : CanalNotificacion` *(Enum: INTERNO, EMAIL, WHATSAPP)*
* **Métodos:**
  * `+ marcarComoLeida() : void` *(CU-32)*
  * `+ marcarComoAtendida() : void` *(CU-32)*
  * `+ enviar() : bool` *(CU-32)*

---

## 2. Bloque de Configuración, Auditoría y Datos
*(Extraído de CU-03, CU-04, CU-20, CU-34)*

### Clase `ConfiguracionSistema`
Parámetros globales modificables por el Superadministrador.
* **Atributos:**
  * `- plazoPagoDefecto : int`
  * `- diasRecordatorio : List<int>` *(ej. días 3 y 5, CU-19)*
  * `- impuestoDefecto : double`
  * `- monedaBase : String`
  * `- umbralAprobacionInferior : double`
  * `- umbralAprobacionSuperior : double`
  * `- periodicidadCopias : int`
  * `- datosEmisor : String`
  * `- datosSoporte : String`
* **Métodos:**
  * `+ actualizarParametros() : bool` *(CU-03)*
  * `+ validarUmbrales() : bool` *(CU-03)*
  * `+ configurarPasarelaCobro(proveedor : String, credenciales : String) : bool` *(CU-04)*

### Clase `RegistroAuditoria`
Trazabilidad inmutable de todas las operaciones críticas del sistema (no tiene métodos de modificación).
* **Atributos:**
  * `- idRegistro : int`
  * `- fechaHora : DateTime`
  * `- tipoOperacion : String`
  * `- entidadAfectada : String`
  * `- valorAnterior : String`
  * `- valorNuevo : String`
* **Métodos:**
  * `{static} + registrarEvento(u : Usuario, tipoOperacion : String, entidad : String, valorAnterior : String, valorNuevo : String) : RegistroAuditoria` *(CU-03, CU-20)*
  * `{static} + consultar(usuario : Usuario, tipoOperacion : String, entidad : String, desde : Date, hasta : Date) : List<RegistroAuditoria>` *(CU-20)*

### Clase `PeriodoContable`
Control de cierre mensual o trimestral para evitar modificaciones en facturas declaradas.
* **Atributos:**
  * `- idPeriodo : String` *(ej. "2026-03" o "2026-Q3")*
  * `- tipo : TipoPeriodo` *(Enum: MENSUAL, TRIMESTRAL)*
  * `- fechaInicio : Date`
  * `- fechaFin : Date`
  * `- cerrado : bool`
  * `- fechaCierre : DateTime`
  * `- motivoReapertura : String`
* **Métodos:**
  * `+ obtenerAvisosPendientes() : List<String>` *(CU-34, borradores o cobros sin conciliar)*
  * `+ cerrarPeriodo(admin : Usuario) : bool` *(CU-34)*
  * `+ reabrirPeriodo(superadmin : Usuario, motivo : String) : bool` *(CU-34)*
  * `+ contieneFecha(fecha : Date) : bool` *(CU-16)*

---

## 3. Bloque de Clientes y Suscripciones
*(Extraído de CU-05, CU-06, CU-08, CU-10, CU-14, CU-15, CU-17, CU-25, CU-26, CU-29, CU-35, CU-36)*

### Clase `Cliente`
Entidad principal sobre la que gira la facturación.
* **Atributos:**
  * `- idCliente : int`
  * `- razonSocial : String`
  * `- nif : String`
  * `- direccionFiscal : String`
  * `- pais : String`
  * `- sector : String`
  * `- etiquetas : List<String>`
  * `- rangoTarifa : String`
  * `- facturacionAnual : double`
  * `- estado : EstadoCliente` *(Enum: ACTIVO, BAJA)*
  * `- fechaBaja : Date`
  * `- eliminadoLogico : bool`
* **Métodos:**
  * `+ darDeAlta() : bool` *(CU-36)*
  * `+ reactivar() : void` *(CU-36, cliente que vuelve)*
  * `+ modificarDatosFiscales() : void` *(CU-17)*
  * `+ asignarResponsable(u : Usuario) : void` *(CU-08)*
  * `+ desasignarResponsable() : void` *(CU-08)*
  * `+ registrarFacturacionAnual(importe : double, fuente : String, fecha : Date) : void` *(CU-05)*
  * `+ calcularRango() : String` *(CU-05)*
  * `+ obtenerFacturasPendientes() : List<Factura>` *(CU-15)*
  * `+ calcularDeudaTotal() : double` *(CU-15)*
  * `+ darDeBaja(fechaEfectiva : Date) : void` *(CU-17)*
  * `+ borradoLogico(motivo : String) : void` *(CU-14)*

### Clase `ContactoCliente`
Persona de contacto dentro de la empresa cliente (separado para cumplir protección de datos CU-35).
* **Atributos:**
  * `- idContacto : int`
  * `- nombre : String`
  * `- email : String`
  * `- telefono : String`
  * `- funcion : String` *(ej. contacto de facturación, CU-10)*
  * `- idioma : String`
  * `- consienteWhatsApp : bool`
* **Métodos:**
  * `+ obtenerCanalEnvio() : CanalNotificacion` *(CU-10)*
  * `+ anonimizarDatos() : void` *(CU-35)*
  * `+ eliminarDatos() : void` *(CU-35)*

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
  * `- motivoPausa : String`
  * `- fechaInicioPausa : Date`
  * `- fechaReactivacionPrevista : Date`
* **Métodos:**
  * `+ activar() : void` *(CU-06)*
  * `+ calcularProrrateo(fechaAlta : Date) : double` *(CU-06)*
  * `+ pausar(fechaInicio : Date, fechaReactivacion : Date, motivo : String) : void` *(CU-25)*
  * `+ reactivar() : void` *(CU-25)*
  * `+ cancelar(fechaEfectiva : Date) : void` *(CU-17)*
  * `+ actualizarTarifa(nuevoRango : String, fechaEfecto : Date) : void` *(CU-05)*
  * `+ aplicarExcepcion(descuento : double, fechaFin : Date) : void` *(CU-26)*
  * `+ calcularImporteMensual(mes : Date) : double` *(CU-29)*
  * `+ generarFacturaMensual(mes : Date) : Factura` *(CU-29)*

---

## 4. Bloque de Facturación, Presupuestos y Aprobaciones
*(Extraído de CU-10, CU-11, CU-12, CU-14, CU-15, CU-16, CU-19, CU-21, CU-23, CU-27, CU-28, CU-29, CU-30)*

### Clase `Presupuesto`
Propuesta comercial previa que puede convertirse en borrador de factura.
* **Atributos:**
  * `- idPresupuesto : String`
  * `- fechaCreacion : Date`
  * `- estado : EstadoPresupuesto` *(Enum: PENDIENTE, ENVIADO, ACEPTADO, RECHAZADO)*
  * `- documentoEvidencia : String`
* **Métodos:**
  * `+ calcularTotal() : double` *(CU-30)*
  * `+ enviar() : bool` *(CU-30)*
  * `+ aceptar(evidencia : String) : void` *(CU-30)*
  * `+ rechazar(evidencia : String) : void` *(CU-30)*
  * `+ convertirEnFactura() : Factura` *(CU-30)*

### Clase `Factura`
Documento contable (puntual, recurrente o rectificativa).
* **Atributos:**
  * `- idFactura : int` *(un borrador aún no tiene número)*
  * `- numeroFactura : String` *(se asigna al emitir, CU-28)*
  * `- serie : String`
  * `- tipo : TipoFactura` *(Enum: PUNTUAL, RECURRENTE, RECTIFICATIVA)*
  * `- fechaEmision : Date`
  * `- fechaVencimiento : Date`
  * `- mesFacturado : String` *(solo recurrentes, evita duplicados en CU-29)*
  * `- moneda : String`
  * `- centroCoste : String`
  * `- baseImponible : double`
  * `- totalImpuestos : double`
  * `- importeTotal : double`
  * `- saldoPendiente : double`
  * `- estado : EstadoFactura` *(Enum: BORRADOR, PENDIENTE_APROBACION, EMITIDA, PAGADA, VENCIDA, RECTIFICADA, ANULADA, INCOBRABLE)*
  * `- enDisputa : bool`
  * `- rutaPdf : String`
* **Métodos:**
  * `+ calcularTotales() : void` *(CU-28)*
  * `+ modificarBorrador() : bool` *(CU-16)*
  * `+ eliminarBorrador() : bool` *(CU-14, solo si nunca se emitió)*
  * `+ emitir() : bool` *(CU-06, CU-28, CU-29)*
  * `+ generarPdf() : String` *(CU-16, CU-28)*
  * `+ obtenerPdf() : String` *(CU-23, devuelve el PDF guardado, no lo regenera)*
  * `+ enviarAlCliente() : bool` *(CU-10)*
  * `+ marcarComoPagada(c : Cobro) : void` *(CU-11, CU-12)*
  * `+ calcularDiasRetraso() : int` *(CU-15)*
  * `+ requiereRecordatorio(fecha : Date) : bool` *(CU-19)*
  * `+ crearRectificativa(motivo : String) : Factura` *(CU-21)*
  * `+ anular(motivo : String) : bool` *(CU-21)*
  * `+ declararIncobrable(motivo : String) : void` *(CU-21)*

### Clase `LineaFactura`
Detalle de cada concepto facturado. Cada línea pertenece **o** a una `Factura` **o** a un `Presupuesto` (composición con restricción `{xor}`); al convertir un presupuesto o crear una rectificativa, las líneas se copian.
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
  * `- tipoOperacion : TipoAprobacion` *(Enum: IMPORTE_MANUAL, DESCUENTO, EXCEPCION_COMERCIAL, NO_PRORRATEO, VENCIMIENTO_ESPECIAL, FACTURA_IMPORTE_ALTO, CAMBIO_RANGO, RECTIFICATIVA, INCOBRABLE)*
  * `- importeAfectado : double`
  * `- motivoSolicitud : String`
  * `- comentarioResolucion : String`
  * `- fechaSolicitud : DateTime`
  * `- fechaResolucion : DateTime`
  * `- estado : EstadoAprobacion` *(Enum: PENDIENTE, APROBADA, RECHAZADA)*
* **Métodos:**
  * `+ requiereSuperadmin() : bool` *(CU-27, supera el umbral superior)*
  * `+ aprobar(aprobador : Usuario, comentario : String) : bool` *(CU-27, el aprobador no puede ser el solicitante)*
  * `+ rechazar(aprobador : Usuario, comentario : String) : void` *(CU-27)*
  * `+ reasignar(suplente : Usuario) : void` *(CU-27)*

---

## 5. Bloque de Cobros, Impagos e Incidencias
*(Extraído de CU-11, CU-12, CU-19, CU-31)*

### Clase `Cobro`
Registro de un pago recibido (automático por Stripe/Revolut o registrado manualmente).
* **Atributos:**
  * `- idCobro : int`
  * `- referenciaExterna : String` *(evita aplicar dos veces el mismo evento, CU-11)*
  * `- fechaCobro : Date`
  * `- importe : double`
  * `- origen : OrigenCobro` *(Enum: STRIPE, REVOLUT, MANUAL)*
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
  * `- canal : CanalNotificacion`
  * `- enviadoConExito : bool`
* **Métodos:**
  * `+ enviarRecordatorio() : bool` *(CU-19)*

### Clase `Incidencia`
Ticket interno o disputa abierta sobre un cliente o una factura (detiene los recordatorios).
* **Atributos:**
  * `- idIncidencia : int`
  * `- descripcion : String`
  * `- prioridad : Prioridad` *(Enum: BAJA, MEDIA, ALTA)*
  * `- fechaApertura : DateTime`
  * `- esDisputaFactura : bool`
  * `- fechaRevision : Date`
  * `- resolucion : String`
  * `- resuelta : bool`
* **Métodos:**
  * `+ abrirIncidencia() : void` *(CU-31)*
  * `+ pausarSeguimientoImpago(motivo : String, fechaRevision : Date) : void` *(CU-19, CU-31)*
  * `+ reanudarSeguimientoImpago() : void` *(CU-31, si la factura sigue pendiente al cerrar)*
  * `+ agregarComentario(u : Usuario, texto : String) : void` *(CU-31, CU-32)*
  * `+ resolverYCerrar(resolucion : String) : void` *(CU-31)*

---

## 6. Resumen de Relaciones Principales (UML)
1. **Composición (`*--`, rombo negro - ciclo de vida dependiente):**
   * `Factura "0..1" *-- "1..*" LineaFactura` *(Si se borra un borrador de factura, desaparecen sus líneas).*
   * `Presupuesto "0..1" *-- "1..*" LineaFactura` *(`{xor}` con la anterior: cada línea pertenece a uno solo de los dos).*
   * `Cliente "1" *-- "1..*" ContactoCliente`
2. **Agregación / Asociación (`o--` / `-->` - ciclo de vida independiente):**
   * `Usuario "0..1" --> "0..*" Cliente : -clientesAsignados` *(Un empleado gestiona varios clientes; un cliente puede quedar sin responsable, CU-08/CU-13).*
   * `Usuario "0..*" --> "0..*" Cliente : -favoritos` *(CU-22).*
   * `Usuario "1" --> "0..*" Notificacion : -notificaciones` *(CU-32).*
   * `DelegacionTemporal "0..*" --> "1" Usuario : -titular`
   * `DelegacionTemporal "0..*" --> "1" Usuario : -sustituto`
   * `DelegacionTemporal "0..*" --> "0..*" Cliente : -clientesDelegados` *(Si está vacía, se delegan todos, CU-07).*
   * `Cliente "1" --> "0..*" Suscripcion : -suscripciones`
   * `Cliente "1" --> "0..*" Factura : -facturas`
   * `Cliente "1" --> "0..*" Presupuesto : -presupuestos` *(CU-30).*
   * `Suscripcion "0..1" --> "0..*" Factura : -facturasGeneradas` *(CU-06, CU-29).*
   * `Presupuesto "0..1" --> "0..1" Factura : -facturaGenerada` *(CU-30).*
   * `Factura "0..1" --> "0..*" Cobro : -cobros` *(Un cobro sin coincidencia clara queda sin factura hasta conciliarse, CU-11/CU-12).*
   * `Factura "1" --> "0..*" RecordatorioImpago : -recordatorios`
   * `Factura "0..1" <-- "0..*" Factura : -facturaOriginal` *(Asociación reflexiva: una original puede tener varias rectificativas, CU-21).*
   * `SolicitudAprobacion "0..*" --> "1" Usuario : -solicitante`
   * `SolicitudAprobacion "0..*" --> "0..1" Usuario : -aprobador` *(CU-27).*
   * `Incidencia "0..*" --> "0..1" Cliente` y `Incidencia "0..*" --> "0..1" Factura` *(`{xor}`: sobre un cliente o sobre una factura, CU-31).*
   * `Incidencia "0..*" --> "1" Usuario : -responsable` *(CU-31).*
   * `RegistroAuditoria "0..*" --> "0..1" Usuario : -autor` *(0..1 porque los procesos del Planificador no tienen usuario).*
3. **Dependencia (`..>`, línea discontinua - relación de uso):**
   * `Factura ..> PeriodoContable : <<uses>>` *(Comprueba si el periodo está abierto antes de editar, CU-16/CU-34).*
   * `Factura ..> ConfiguracionSistema : <<uses>>` *(Lee los impuestos y series vigentes al emitirse, CU-03/CU-28).*
   * `SolicitudAprobacion ..> ConfiguracionSistema : <<uses>>` *(Lee los umbrales de aprobación, CU-27).*
