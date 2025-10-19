# ADR-0002: Agregacoes pre-computadas em janelas fixas

- **Status:** Aceita
- **Tags:** analytics, performance, agregacao
- **Trade-off:** [TRD-0002 - Agregacoes pre-computadas vs query-time](TRD-0002-agregacoes-tempo.md)

## Contexto

Dashboard mostra metricas agregadas: usuarios ativos, eventos por tipo, conversoes. Cada refresh do dashboard dispara queries.

Problema: query sobre 30 dias de dados com 100M eventos e lenta, mesmo no ClickHouse. Usuario espera resposta instantanea.

## Decisao

Agregacoes pre-computadas em janelas fixas via materialized views.

Janelas:
- Por minuto (ultimas 24 horas)
- Por hora (ultimos 30 dias)
- Por dia (historico completo)

```sql
CREATE MATERIALIZED VIEW events_hourly
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(hour)
ORDER BY (tenant_id, event_type, hour)
AS SELECT
    tenant_id,
    event_type,
    toStartOfHour(timestamp) as hour,
    count() as count,
    uniqState(user_id) as users_state
FROM events
GROUP BY tenant_id, event_type, hour
```

Queries do dashboard usam a view agregada, nao a tabela raw.

## Justificativa

**Query-time aggregation sempre**
- Flexibilidade maxima (qualquer corte de dados)
- Mas lento pra dashboards
- Custo de CPU alto
- Nao escala com volume

**Cache de queries**
- Ajuda em repeticao exata
- Mas qualquer parametro diferente invalida
- TTL curto pra dados real-time

**Pre-agregacao em janelas**
- Dashboard instantaneo
- Custo de storage marginal (agregado e pequeno)
- Trade-off em flexibilidade (so cortes pre-definidos)

**Cubos OLAP (Druid style)**
- Flexibilidade maxima com performance
- Mas complexidade de operacao alta
- Overkill pro caso de uso

Trade-off aceito: queries ad-hoc sobre dados raw continuam lentas. Aceitavel porque 95% do uso e via dashboard com dimensoes conhecidas.

## Consequencias

- Dashboard carrega em < 500ms
- Materialized views ocupam ~5% do storage dos eventos raw
- Novas dimensoes requerem nova view (planejamento)
- Queries exploratórias vao na tabela raw (usuario sabe que e lento)
