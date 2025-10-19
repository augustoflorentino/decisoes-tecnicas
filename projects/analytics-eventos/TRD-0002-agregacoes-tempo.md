# TRD-0002: Agregacoes pre-computadas vs query-time

- **Status:** Concluida
- **ADR resultante:** [ADR-0002 - Agregacoes pre-computadas em janelas fixas](ADR-0002-agregacoes.md)

## Contexto

Dashboard de analytics mostra metricas agregadas. Usuario seleciona periodo e dimensoes.

Exemplo: "Usuarios ativos por dia, ultimos 30 dias, filtrado por plano=premium"

Questao: computar na hora da query ou pre-computar?

## Opcoes

### A: Query-time puro

```sql
SELECT toDate(timestamp), uniqExact(user_id)
FROM events
WHERE tenant_id = ? AND timestamp > now() - 30 days AND plan = 'premium'
GROUP BY toDate(timestamp)
```

- **Pros:** Flexibilidade total, qualquer filtro funciona, dados sempre atualizados
- **Contras:** Lento com volume alto, custo de CPU, experiencia ruim

### B: Pre-agregacao total (cubo OLAP)

Pre-computa todas as combinacoes de dimensoes.

- **Pros:** Queries instantaneas, qualquer corte
- **Contras:** Explosao combinatoria de dimensoes, storage massivo, complexidade

### C: Pre-agregacao seletiva (materialized views)

Pre-computa apenas agregacoes mais usadas.

- **Pros:** Queries comuns sao rapidas, storage controlado
- **Contras:** Queries fora do padrao sao lentas, manutencao das views

### D: Cache de resultados

Cacheia resultado de queries frequentes.

- **Pros:** Simples, queries repetidas sao instantaneas
- **Contras:** Cache miss e lento, invalidacao complexa, parametros diferentes = miss

## Analise

Mapeamento de uso real:
- 85% das queries: ultimos 7/30 dias, agregado por dia/hora, 3-4 dimensoes conhecidas
- 10% das queries: periodos custom, mesmas dimensoes
- 5% das queries: exploratorio (qualquer coisa)

Pre-agregar pra 85% resolve a maioria. Os 15% restantes podem ser lentos.

Cubo OLAP com 10 dimensoes de 100 valores cada = 100^10 combinacoes = inviavel.

Materialized views pra top 10 queries cobre 85% do uso.

## Decisao

**Pre-agregacao seletiva** via materialized views.

Views criadas:
- events_by_day: count, usuarios unicos por dia/tenant/event_type
- events_by_hour: mesmo, por hora (ultimos 30 dias)
- conversions_by_source: funil por fonte de trafego

Queries exploratórias vao na tabela raw com warning de latencia.

## Riscos

- Dimensao nova popular nao tem view - mitigacao: monitorar queries lentas, criar view sob demanda
- View desatualizada - mitigacao: materialized view e real-time no ClickHouse
