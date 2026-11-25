---
name: xubio-papeles-rivadavia
description: "Gestiona en Xubio los presupuestos pendientes de Carpintería Rivadavia: aprobar, marcar Facturado, crear el comprobante de venta, hacer el remito e imprimir el Remito Valorizado tipo 4."
---

# Papeles de Carpintería Rivadavia en Xubio

## Objetivo y orden requerido

Procesar el presupuesto pendiente de aprobar que indique el usuario y seguir el orden del módulo administrativo:

1. Aprobar el presupuesto: **Pendiente de Aprobar → Aprobado**.
2. Cambiarlo a **Facturado** y verificar el estado.
3. Crear, como operación independiente, el comprobante de venta con los datos del presupuesto.
4. Crear el remito de venta desde los datos del comprobante.
5. Generar e imprimir/entregar el **Remito Valorizado**, con tipoimpresion=4.

El comprobante debe conservar cliente, productos, cantidades, precios, descuentos, impuestos y demás condiciones confirmadas del presupuesto. No afirmar que el comprobante está asociado nativamente al presupuesto si Xubio lo crea como operación independiente.

## Reglas de seguridad

- Usar la API y el cliente Xubio configurados para el proyecto activo. Revisar el catálogo operativo y el playbook antes de escribir. No inventar endpoints, campos, IDs, puntos de venta, talonarios, depósitos ni condiciones.
- Mantener las credenciales en la configuración local. Nunca mostrar, pedir, copiar a la respuesta ni registrar client_secret, tokens, cookies o sessionidlocal.
- Resolver en vivo el registro exacto de **Carpintería Rivadavia** y los presupuestos en estado **Pendiente de Aprobar**. No identificar al cliente solo por coincidencia parcial.
- Si hay un presupuesto, preparar ese. Si hay varios, mostrar número, fecha, estado, total e ítems y esperar que la persona autorizada elija por número. No modificar los no seleccionados.
- Antes de la primera escritura, mostrar una vista previa completa y pedir confirmación inequívoca del flujo entero: aprobación, cambio a Facturado, comprobante independiente, remito e impresión valorizada tipo 4.
- Ejecutar y verificar cada transición antes de pasar a la siguiente. No asumir que HTTP 200 significa éxito funcional.
- Evitar duplicados: antes de reintentar una creación, consultar si el objeto ya existe. Usar externalId idempotente cuando el cliente API lo soporte; registrar IDs y números devueltos.
- No alterar cantidades, precios o impuestos sin instrucción expresa. Informar cualquier dato incompleto o inconsistencia antes de confirmar.
- Si la creación del comprobante falla después de que el presupuesto quedó Facturado, informar el estado parcial con claridad. No revertir ni repetir escrituras automáticamente; verificar antes de cualquier intento posterior.
- Nunca afirmar éxito si una etapa quedó incompleta. Distinguir PDF generado de impresión física efectivamente enviada.

## Procedimiento

### 1. Revisar la operación

- Consultar el cliente exacto y sus presupuestos pendientes de aprobar.
- Leer el detalle completo de cada candidato: renglones, precios, descuentos, IVA, total, condición de pago y referencias disponibles.
- Si no hay pendientes, informarlo y no crear un presupuesto nuevo.
- Si hay varios, esperar que la persona autorizada elija el número. Si hay uno, preparar la vista previa.

### 2. Mostrar y confirmar

La vista previa debe incluir cliente y presupuesto seleccionado, número/fecha/estado/ID, todos los ítems con cantidades y valores, subtotales, impuestos y total, y este orden: **Pendiente de Aprobar → Aprobado → Facturado → comprobante de venta independiente → remito → Remito Valorizado tipo 4**. Indicar también cualquier dato operativo requerido que no se pueda resolver inequívocamente.

Esperar una confirmación inequívoca antes de modificar Xubio.

### 3. Aprobar el presupuesto

- Cambiar de Pendiente de Aprobar a Aprobado mediante la transición admitida por Xubio.
- Consultar nuevamente y verificar que figure Aprobado.
- Si la transición falla, detenerse; no continuar con el presupuesto ni crear documentos.

### 4. Dejar el presupuesto Facturado

- Cambiar el estado de Aprobado a Facturado mediante la transición admitida por Xubio.
- Consultar nuevamente y verificar que figure Facturado antes de crear el comprobante.
- Si la transición falla, detenerse; no crear el comprobante ni el remito.
- No dejarlo como estado final Pendiente de Aprobar o Aprobado.

### 5. Crear el comprobante de venta independiente

- Crear el comprobante usando el cliente y los datos del presupuesto aprobado como fuente, aunque el presupuesto ya figure Facturado.
- Resolver en vivo los campos que Xubio requiera: punto de venta, talonario, depósito, lista de precios, vendedor, centro de costo, provincia y condición de pago. No copiar IDs de pruebas ni de otros clientes.
- Respetar la configuración del cliente. No agregar cobranzas si la condición de pago no lo requiere.
- Consultar el comprobante recién creado y verificar cliente, ítems, importes, impuestos, número y estado.
- Si falla, conservar el estado real del presupuesto y reportar el bloqueo. Antes de reintentar, comprobar que no se haya creado ya el comprobante.

### 6. Crear e imprimir el remito valorizado

- Crear el remito desde los datos del comprobante verificado, usando el flujo soportado por el cliente API del proyecto.
- Consultar el catálogo/playbook para obtener los campos y el siguiente número de remito. La referencia documentada usa el formato X-0001-########; verificar el número actual antes de crear.
- Si no hay vendedor o comisión aplicable, omitir vendedorId y comisionVendedor; no enviar un vendedor vacío o sin comisión.
- Consultar el remito creado y verificar cliente, renglones, cantidades e importes.
- Solicitar el PDF con tipoimpresion=4. El tipo 2 corresponde al Remito Preimpreso y no cumple este pedido.
- Verificar que el PDF muestre valores unitarios y pertenezca a la operación elegida. Enviar a impresión solo si el entorno ofrece esa función; si no, entregar el PDF e indicar que está listo para imprimir.

## Verificación y respuesta

Al terminar, informar presupuesto (número/ID y estado Facturado), comprobante (número/ID y verificación), remito (número/ID y verificación) y el PDF valorizado, aclarando si se imprimió efectivamente o solo se generó.

Si el proceso queda incompleto, enumerar las etapas completadas, el último estado verificado y el error funcional de Xubio. No exponer secretos.

## Referencia técnica

Consultar [references/referencias-xubio.md](references/referencias-xubio.md) junto con el catálogo y la configuración vigentes del proyecto.

## Integración con Asistente_Codex

Esta skill es la guía operativa para las respuestas de Codex. En el proyecto Asistente_Codex, delegar las escrituras y la impresión al flujo existente de `app/tool_agent.py` y `app/administration/service.py`, que es el ejecutor de Xubio. Usar su vista previa y confirmación como puerta obligatoria; no iniciar una segunda secuencia de escrituras directamente contra la API.

