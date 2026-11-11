# Diseño Preliminar de Implementación (Fase 2)

Este documento describe el diseño previsto; RabbitMQ y sus consumidores aún no están implementados. La selección y comparación de protocolos se encuentra en [protocol-selection.md](protocol-selection.md).

## Flujo previsto

1. El cliente envía la orden al servicio de órdenes mediante REST. El servicio registra la orden como `PENDING` y responde con su identificador.
2. El servicio publica `order.created` en el exchange `orders.events`.
3. RabbitMQ enruta el evento a colas independientes de pagos e inventario. Cada consumidor procesa la orden y comunica su resultado.
4. El servicio de órdenes consolida los resultados y publica `order.confirmed` o `order.rejected`.
5. El consumidor de notificaciones procesa el resultado y avisa al cliente. El estado final permanece disponible mediante REST.

## Mensajes y entrega

Los eventos incluirán un identificador único, tipo y versión del evento, identificador de orden, fecha de creación y datos mínimos para el consumidor. Se evitarán datos de pago sensibles. Las colas serán durables y los mensajes importantes persistentes; se usarán confirmaciones de publicación y ACK manuales después de completar el procesamiento.

Como RabbitMQ puede volver a entregar mensajes, los consumidores deberán ser idempotentes. Se definirán reintentos limitados y una DLQ para mensajes que no puedan procesarse. Se evaluará el patrón *transactional outbox* para mantener consistente el registro de la orden con la publicación del evento.

## Decisiones

- Mantener REST para la interacción síncrona con el cliente y usar AMQP 0-9-1 mediante RabbitMQ para el procesamiento entre servicios.
- Usar Spring AMQP para integrar RabbitMQ con Spring Boot durante la Fase 2.
- Separar las colas por consumidor para permitir procesamiento, reintentos y escalado independientes.