# Gateway Pagamentos

Orquestrador de pagamentos multi-adquirente para e-commerces. Abstrai integracao com Cielo, Rede, Stone, PagSeguro.

## Stack

- Go (API principal)
- NestJS (backoffice)
- PostgreSQL
- Redis (cache + idempotencia)
- RabbitMQ (eventos)
- Elasticsearch (auditoria)

## ADRs

- [ADR-0001 - Idempotencia com chave composta e janela temporal](ADR-0001-idempotencia.md)
- [ADR-0002 - Roteamento dinamico de adquirente](ADR-0002-roteamento-adquirente.md)
- [ADR-0003 - Event sourcing para transacoes financeiras](ADR-0003-event-sourcing.md)
- [ADR-0004 - Conciliacao assincrona com tolerancia a divergencia](ADR-0004-conciliacao.md)

## Trade-offs

- [TRD-0001 - Go vs Node para API de pagamentos](TRD-0001-linguagem-api.md) → ADR implícita na stack
- [TRD-0002 - RabbitMQ vs Kafka para eventos financeiros](TRD-0002-mensageria.md)
- [TRD-0003 - Estrategia de fallback entre adquirentes](TRD-0003-fallback-adquirente.md) → ADR-0002
