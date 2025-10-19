# ADR-0001: ClickHouse para armazenamento de eventos

- **Status:** Aceita
- **Tags:** banco, analytics, time-series

## Contexto

Plataforma vai receber eventos de tracking de produtos SaaS: page views, clicks, conversoes, erros. Volume esperado: 100M eventos/dia, crescendo.

Queries tipicas:
- "Quantos usuarios ativos ontem por plano?"
- "Qual a taxa de conversao da ultima semana por fonte de trafego?"
- "Mostre o grafico de erros por hora nos ultimos 30 dias"

Caracteristicas: append-only, queries agregadas, range de tempo, alta cardinalidade em algumas dimensoes.

## Decisao

ClickHouse como banco principal de eventos.

Estrutura:
```sql
CREATE TABLE events (
    tenant_id UInt64,
    event_type LowCardinality(String),
    timestamp DateTime64(3),
    user_id String,
    properties String,  -- JSON
    ...
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (tenant_id, event_type, timestamp)
```

## Justificativa

**PostgreSQL com TimescaleDB**
- Familiar, SQL padrao
- Mas performance degrada com volume alto
- Compressao inferior
- Nao foi feito pra analytics

**Elasticsearch**
- Bom pra busca full-text
- Queries agregadas sao lentas
- Custo de storage alto
- Schema management problematico

**ClickHouse**
- Otimizado pra queries analiticas (columnar)
- Compressao excelente (~10x)
- Escala horizontal
- SQL familiar
- Particionamento nativo por tempo

**Druid/Pinot**
- Mais complexo de operar
- Menos documentacao
- Overkill pro volume inicial

Trade-off: ClickHouse nao e transacional. Updates sao caros. Aceitavel porque eventos sao imutaveis.

## Consequencias

- Queries que levavam 30s no Postgres rodam em 200ms
- Storage de 1TB de eventos comprimido pra ~100GB
- Particionamento por mes facilita retencao (drop partition)
- Materialized views pra agregacoes frequentes
- Curva de aprendizado das peculiaridades do ClickHouse
