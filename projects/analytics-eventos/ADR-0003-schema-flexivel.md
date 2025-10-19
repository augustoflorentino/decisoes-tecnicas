# ADR-0003: Schema flexivel com colunas dinamicas

- **Status:** Aceita
- **Tags:** schema, flexibilidade, eventos

## Contexto

Clientes enviam eventos com propriedades customizadas:

```json
{"event": "purchase", "properties": {"product_id": "123", "price": 99.90, "coupon": "PROMO10"}}
{"event": "signup", "properties": {"source": "google", "campaign": "black-friday"}}
```

Nao da pra prever quais propriedades cada cliente vai usar. Schema rigido quebra.

## Decisao

Colunas fixas para campos comuns + coluna JSON para propriedades dinamicas.

```sql
CREATE TABLE events (
    tenant_id UInt64,
    event_type LowCardinality(String),
    timestamp DateTime64(3),
    user_id String,
    session_id String,
    -- propriedades dinamicas
    properties String,  -- JSON
    -- colunas extraidas para queries frequentes
    property_source LowCardinality(String) MATERIALIZED JSONExtractString(properties, 'source'),
    property_value Float64 MATERIALIZED JSONExtractFloat(properties, 'value')
)
```

Colunas materializadas sao criadas sob demanda quando cliente usa muito uma propriedade.

## Justificativa

**Schema rigido (todas as colunas pre-definidas)**
- Performance otima
- Mas nao suporta propriedades custom
- Cada cliente novo requer migration

**Tudo em JSON (schemaless)**
- Flexibilidade total
- Mas queries em JSON sao lentas
- Sem type safety
- Compressao ruim

**Colunas dinamicas do ClickHouse (Map type)**
- Flexibilidade boa
- Mas Map nao suporta todos os tipos
- Performance inferior a colunas nativas

**Hibrido: fixo + JSON + materializacao**
- Campos comuns sao rapidos
- Propriedades custom funcionam (mais lentas)
- Promocao pra coluna nativa quando justifica
- Melhor dos mundos

Trade-off: gestao de quais propriedades viram coluna. Processo manual baseado em metricas de uso.

## Consequencias

- Clientes enviam qualquer propriedade sem coordenacao
- Queries em propriedades frequentes sao rapidas
- Queries em propriedades raras funcionam (JSONExtract)
- Script semanal identifica candidatos a promocao
- Colunas materializadas sao populadas retroativamente
