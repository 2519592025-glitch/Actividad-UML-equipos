# Caso 2: Plataforma E-Commerce Omnicanal ("GlobalMarket")

## 1. Integrante
- **Nombre:** Anderson Ernesto Castillo Navarrete

## 2. Descripción de Entidades y Relaciones
- **Pedido (Order) y Línea de Detalle (OrderDetail):** Relación de **Composición**. Las líneas de detalle no existen de manera independiente sin un pedido asignado. Si el pedido se elimina, las líneas de detalle se destruyen automáticamente.
- **Línea de Detalle y Artículo (Item):** Relación de **Asociación / Agregación**. Las líneas de detalle hacen referencia a artículos del catálogo, pero los artículos existen independientemente del pedido.
- **Cliente y Pedido:** Un cliente puede realizar múltiples pedidos (1 a N).
- **Herencia de Clientes:** `Cliente` es una clase base que se especializa en `ClienteCorporativo` y `ClienteIndividual`.
- **Herencia de Pagos:** `Pago` es una clase abstracta/base que se especializa mediante herencia en `PagoTarjeta`, `TransferenciaBancaria` y `PagoCripto`.
- **Envío (Shipment):** Relación de **Asociación** entre el Pedido y el proveedor logístico externo.

## 3. Justificación Técnica: Composición vs Agregación
Se definió una relación de **Composición** entre `Pedido` y `LíneaDeDetalle` porque la vida útil de las líneas depende estrictamente de la existencia del pedido general. Por el contrario, la relación con `Artículo` es una **Agregación/Asociación**, dado que la eliminación de un pedido no destruye el producto del inventario general del catálogo.
