# Analytics Eventos

Plataforma de analytics para produtos SaaS. Ingestao de eventos, dashboards em tempo real, retencao configuravel.

## Stack

- Go (ingestao)
- ClickHouse (time-series)
- Kafka (buffer de eventos)
- Redis (agregacoes real-time)
- PostgreSQL (metadata)
- Grafana (visualizacao)

## ADRs

- [ADR-0001 - ClickHouse para armazenamento de eventos](ADR-0001-clickhouse.md)
- [ADR-0002 - Agregacoes pre-computadas em janelas fixas](ADR-0002-agregacoes.md)
- [ADR-0003 - Schema flexivel com colunas dinamicas](ADR-0003-schema-flexivel.md)
- [ADR-0004 - Retencao em camadas com downsampling](ADR-0004-retencao.md)

## Trade-offs

- [TRD-0001 - Kafka vs ingestao direta no ClickHouse](TRD-0001-buffer-ingestao.md) → ADR implícita
- [TRD-0002 - Agregacoes pre-computadas vs query-time](TRD-0002-agregacoes-tempo.md) → ADR-0002
- [TRD-0003 - Multi-tenancy por database vs por coluna](TRD-0003-multi-tenancy.md)
