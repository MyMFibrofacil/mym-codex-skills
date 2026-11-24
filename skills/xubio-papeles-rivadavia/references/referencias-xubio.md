# Referencias de Xubio para la skill

Base de API: https://xubio.com/API/1.1

## Recursos utilizados

- Presupuestos: POST/GET/PUT /presupuestoBean.
- Comprobantes de venta: POST/GET/PUT /comprobanteVentaBean.
- Remitos de venta: POST/GET/PUT /remitoVentaBean.
- Impresión PDF: /imprimirPDF.

## Estados y flujo

Los identificadores anotados para estados de presupuesto son -3 (Pendiente de Aprobar), -2 (Aprobado), -5 y -4. Confirmar los valores disponibles en vivo antes de operar; objetivo final: estado Facturado. Crear el comprobante de venta como operación independiente desde el presupuesto aprobado, conservar los datos confirmados y luego verificar que el presupuesto figure Facturado.

## Remito valorizado

Solicitar TipoImpresion=4 para el remito valorizado. Tipo 2 corresponde al remito preimpreso.

## Seguridad y consistencia

- Resolver IDs de cliente, presupuesto, comprobante, productos, depósitos y listas de precios consultando Xubio; nunca asumirlos.
- No hacer reintentos ciegos en escrituras: pueden duplicar comprobantes o remitos. Verificar antes de repetir.
- No guardar credenciales en la skill ni en el repositorio. Configurarlas localmente y de forma segura en cada equipo.
