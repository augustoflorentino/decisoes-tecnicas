# TRD-0002: RabbitMQ vs Kafka para eventos financeiros

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada na infra inicial)

## Contexto

Sistema precisa de mensageria pra:
- Notificar merchants sobre mudanca de status (webhooks)
- Processar conciliacao em background
- Sincronizar projecoes do event sourcing
- Comunicar entre servicos

Volume estimado: ~50k eventos/dia, picos de 500/segundo.

## Opcoes

### A: RabbitMQ

- **Pros:** Semantica de fila classica, acknowledgment por mensagem, dead letter nativo, roteamento flexivel
- **Contras:** Mensagens consumidas somem, replay impossivel, cluster complexo

### B: Kafka

- **Pros:** Log append-only, replay de eventos, retencao configuravel, throughput extremo
- **Contras:** Complexidade operacional, consumer groups, ordering por particao

### C: Redis Streams

- **Pros:** Ja temos Redis, consumer groups, simples
- **Contras:** Persistencia secundaria do Redis, nao e o foco do produto, limites de escala

### D: AWS SQS + SNS

- **Pros:** Gerenciado, escala infinita, integracao AWS
- **Contras:** Vendor lock-in, latencia maior, custo por mensagem escala mal

## Analise

Event sourcing sugere Kafka (eventos sao o log). Mas:

- Volume atual nao justifica complexidade do Kafka
- Eventos de transacao ja estao no banco (event store)
- Mensageria aqui e pra integracao, nao pra ser fonte de verdade
- RabbitMQ cobre o caso de uso com menos overhead

Kafka faz sentido se:
- Volume passar de 1M eventos/dia
- Precisar de replay de eventos de integracao
- Multiplos consumidores do mesmo evento

## Decisao

**RabbitMQ** para mensageria de integracao.

Eventos financeiros ficam no event store (Postgres). RabbitMQ distribui notificacoes derivadas.

```
Transacao capturada
  -> Event store (fonte verdade)
  -> RabbitMQ (notifica webhook worker, atualiza projecao, etc)
```

Se perder mensagem do RabbitMQ, pode reprocessar do event store.

## Riscos

- RabbitMQ nao escalar pro volume futuro - mitigacao: monitorar, migrar pra Kafka se necessario
- Mensagem perdida antes de processar - mitigacao: event store permite replay
