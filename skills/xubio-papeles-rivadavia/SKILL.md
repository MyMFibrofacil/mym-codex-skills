---
name: xubio-papeles-rivadavia
description: "Gestiona en Xubio los papeles de Carpintería Rivadavia desde presupuestos pendientes; revisar y elegir el presupuesto, aprobarlo, crear el comprobante de venta con sus datos, dejar el presupuesto Facturado, crear el remito e imprimir el Remito Valorizado tipo 4. Usar ante pedidos de papeles de Rivadavia."
---

# Papeles de Carpintería Rivadavia en Xubio

## Objetivo y resultado requerido

Procesar el presupuesto pendiente de aprobar que indique el usuario y completar el circuito:

1. Aprobar el presupuesto.
2. Crear un comprobante de venta separado, cargado con los datos del presupuesto.
3. Cambiar el presupuesto a **Facturado**.
4. Crear el remito desde el comprobante de venta.
5. Generar e imprimir/entregar el **Remito Valorizado**, tipoimpresion=4.

El comprobante puede crearse como una operación independiente en Xubio. Debe conservar los productos, cantidades, precios, descuentos, impuestos, cliente y demás condiciones del presupuesto elegido. No dar por finalizado el trabajo hasta verificar el comprobante, el estado Facturado del presupuesto, el remito y el PDF valorizado.

## Reglas de seguridad

- Usar la API y el cliente Xubio configurado para el proyecto activo. Revisar primero su catálogo/playbook operativo si el método necesario no está disponible. No inventar endpoints, campos, IDs, puntos de venta, talonarios, depósitos ni condiciones.
- Mantener credenciales en la configuración local existente. Nunca mostrar, pedir, copiar a la respuesta ni registrar client_secret, tokens, cookies o sessionidlocal.
- Resolver en vivo el registro exacto de **Carpintería Rivadavia** y el presupuesto antes de escribir. No identificar al cliente solo por una coincidencia parcial.
- Buscar presupuestos en estado **Pendiente de Aprobar**. Si hay uno, preparar ese. Si hay más de uno, mostrar para cada uno número, fecha, estado, total e ítems, y esperar que el usuario elija por número. No modificar los no seleccionados.
- Antes de la primera escritura, mostrar una vista previa completa y pedir confirmación inequívoca. Incluir explícitamente aprobación, creación independiente del comprobante, estado final Facturado, creación del remito e impresión valorizada tipo 4.
- Ejecutar las transiciones de estado una por una y verificar cada resultado con una consulta posterior. No asumir que HTTP 200 significa éxito funcional.
- Evitar duplicados: antes de reintentar una creación, consultar si el objeto ya existe. Usar externalId idempotente cuando el cliente API lo soporte; registrar los IDs y números devueltos.
- No alterar cantidades, precios o impuestos del presupuesto sin una instrucción expresa. Informar cualquier dato incompleto o inconsistencia antes de confirmar.
- Nunca informar éxito si una etapa quedó incompleta. Si falla una etapa, identificarla y reportar con precisión qué objetos sí se crearon y qué estado tiene el presupuesto. No marcar el presupuesto Facturado si no existe y no se verificó el comprobante correspondiente.
- Las escrituras son secuenciales. Tras aprobar, continuar hasta crear y verificar el comprobante y dejar el presupuesto Facturado. Si una etapa intermedia falla, detener nuevas escrituras no seguras y comunicar el estado parcial; no ocultar que el presupuesto quedó temporalmente Aprobado ni afirmar que el flujo terminó.

## Procedimiento

### 1. Revisar la operación

- Consultar en Xubio el cliente exacto y sus presupuestos pendientes de aprobar.
- Consultar el detalle completo de cada presupuesto candidato, incluyendo renglones, precios, descuentos, IVA, total, condición de pago y referencias disponibles.
- Si no hay presupuestos pendientes, informarlo y no crear un presupuesto nuevo.
- Si hay varios, presentar la lista y esperar que Pablo elija el número. Si hay uno, continuar con la vista previa.

### 2. Mostrar y confirmar

La vista previa debe indicar:

- Cliente y presupuesto seleccionado (número, fecha, estado e ID interno si es útil).
- Todos los ítems con cantidad y valores; subtotales, impuestos y total.
- Cambio de estado: Pendiente de Aprobar → Aprobado → Facturado.
- Comprobante de venta independiente, creado con los datos del presupuesto.
- Remito derivado del comprobante y salida impresa/PDF como Remito Valorizado tipo 4.
- Cualquier dato operativo adicional que Xubio requiera y que no se pueda resolver de forma inequívoca.

Esperar una confirmación inequívoca antes de modificar Xubio.

### 3. Aprobar el presupuesto

- Cambiar el estado de Pendiente de Aprobar a Aprobado mediante la transición admitida por Xubio.
- Consultar el presupuesto y verificar que el estado sea Aprobado antes de continuar.
- Si la transición falla, detenerse; no crear el comprobante ni el remito.

### 4. Crear el comprobante de venta independiente

- Crear el comprobante de venta usando la entidad y los datos del presupuesto aprobado como fuente.
- No crear otro presupuesto ni declarar que el comprobante está técnicamente asociado al presupuesto si la API solo permite una creación independiente.
- Resolver desde Xubio los campos operativos requeridos por la empresa, como punto de venta, talonario, depósito, lista de precios, vendedor, centro de costo, provincia y condición de pago. No copiar valores de pruebas o de otros clientes.
- Respetar la configuración del cliente y el catálogo vigente. No agregar cobranzas si la condición de pago no lo requiere.
- Consultar el comprobante recién creado y verificar cliente, ítems, importes, impuestos, número y estado.

### 5. Dejar el presupuesto Facturado

- Solo después de verificar el comprobante, avanzar el presupuesto de Aprobado a Facturado mediante la transición admitida.
- Consultar el presupuesto y verificar que figure Facturado.
- No dejarlo como estado final Pendiente de Aprobar o Aprobado.
- Si el comprobante no se pudo crear o verificar, no marcar el presupuesto Facturado. Reportar el estado parcial y resolver el bloqueo antes de considerar finalizado el trámite.

### 6. Crear y emitir el remito valorizado

- Crear el remito de venta desde los datos del comprobante de venta verificado, usando el flujo soportado por el cliente API del proyecto.
- Consultar el catálogo/playbook del proyecto para obtener los campos y el siguiente número de remito. La referencia operativa documentada usa el formato X-0001-########; verificar el número actual antes de crear.
- Si no hay vendedor o comisión aplicable, omitir vendedorId y comisionVendedor; no enviar un vendedor vacío o sin comisión.
- Consultar el remito creado y verificar cliente, renglones, cantidades e importes.
- Solicitar el PDF mediante el mecanismo API ya configurado. Para Remito Valorizado usar tipoimpresion=4. El tipo 2 corresponde al Remito Preimpreso y no satisface este pedido.
- Verificar que el PDF muestre valores unitarios y que sea el remito del comprobante/presupuesto elegido. Enviar a impresión si el entorno ofrece una función de impresión autorizada; de lo contrario, entregar el PDF y aclarar que queda listo para imprimir.

## Verificación y respuesta

Al terminar, informar:

- Presupuesto: número, ID si está disponible y estado **Facturado**.
- Comprobante de venta: número, ID y resultado de verificación.
- Remito: número, ID y resultado de verificación.
- Remito Valorizado: PDF adjunto/enlace y si se envió efectivamente a imprimir.

Si el proceso queda incompleto, enumerar las etapas completadas, el último estado verificado y el error funcional de Xubio. No exponer secretos ni afirmar impresión física cuando solo se generó el PDF.

## Referencia técnica

Consultar [referencias-xubio.md](references/referencias-xubio.md) para los endpoints y decisiones técnicas documentadas. Esta referencia no reemplaza el catálogo operativo ni la configuración vigentes del proyecto.
