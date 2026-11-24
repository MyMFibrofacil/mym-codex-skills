# Referencias de Xubio para la skill

Base de API: https://xubio.com/API/1.1

## Recursos utilizados

- Presupuestos: POST/GET/PUT /presupuestoBean.
- Cambio de estado: PUT /presupuestoBean/{id}/estado.
- Comprobantes de venta: POST/GET/PUT /comprobanteVentaBean.
- Remitos de venta: POST/GET/PUT /remitoVentaBean.
- Impresión PDF: /imprimirPDF.

## Estados y flujo del módulo

Estados anotados: -3 Pendiente de Aprobar, -2 Aprobado, -5 Facturado y -4 Remitido. Confirmar los IDs y las transiciones disponibles en Xubio antes de operar.

Seguir el orden operativo del módulo: cambiar el presupuesto de Pendiente de Aprobar a Aprobado y verificarlo; cambiarlo de Aprobado a Facturado y verificarlo; después crear el comprobante de venta independiente con los datos confirmados del presupuesto y verificarlo. Si la creación del comprobante falla, informar que el presupuesto ya quedó Facturado; no revertir el estado ni reintentar a ciegas.

El cambio de estado a Facturado no crea por sí mismo un comprobante de venta. Para emitir el comprobante se requiere POST /comprobanteVentaBean.

## Remito valorizado

Solicitar el PDF con tipoimpresion=4 para el Remito Valorizado. Tipo 2 corresponde al Remito Preimpreso.

## Seguridad y consistencia

- Resolver en vivo IDs de cliente, presupuesto, comprobante, productos, depósitos, listas de precios y numeración; nunca asumirlos.
- Crear el remito desde los datos del comprobante verificado y confirmar su número antes de enviarlo a impresión.
- No hacer reintentos ciegos en escrituras: pueden duplicar comprobantes o remitos. Consultar si el recurso ya existe antes de repetir.
- No guardar credenciales en la skill ni en el repositorio. Configurarlas localmente y de forma segura en cada equipo.
