# TRD-0003: Multi-tenancy por database vs por coluna

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada no schema inicial)

## Contexto

Plataforma SaaS com multiplos clientes (tenants). Cada um envia eventos proprios e so pode ver os seus.

Questao: como isolar dados entre tenants?

## Opcoes

### A: Database por tenant

```
tenant_1.events
tenant_2.events
tenant_3.events
```

- **Pros:** Isolamento total, facil deletar tenant (drop database), sem risco de vazamento
- **Contras:** Milhares de databases, schema migration em todos, queries cross-tenant impossiveis

### B: Tabela por tenant

```
events_tenant_1
events_tenant_2
```

- **Pros:** Isolamento bom, drop table pra deletar
- **Contras:** Milhares de tabelas, mesmos problemas de schema migration

### C: Coluna de tenant (row-level)

```sql
events (tenant_id, event_type, ...)
WHERE tenant_id = ?
```

- **Pros:** Uma tabela, schema migration simples, queries eficientes com indice
- **Contras:** Risco de bug vazar dados, delete tenant requer DELETE em massa

### D: Row-level security (RLS)

Coluna de tenant + politicas de acesso no banco.

- **Pros:** Isolamento forcado pelo banco, menos risco de bug
- **Contras:** ClickHouse nao tem RLS nativo, overhead de performance

## Analise

Volume esperado: 1000+ tenants, cada um com milhoes de eventos.

Database por tenant:
- 1000 databases = pesadelo operacional
- ClickHouse nao foi feito pra isso
- Backup/restore individual complexo

Coluna de tenant:
- Uma tabela grande, ClickHouse gosta disso
- Particionamento por (tenant_id, mes) isola fisicamente
- WHERE tenant_id sempre presente, enforced na API

Risco de vazamento e real. Mitigacoes:
- Middleware que injeta tenant_id em toda query
- Testes automatizados de isolamento
- Audit log de queries

## Decisao

**Coluna de tenant** (row-level) com enforced via aplicacao.

Schema:
```sql
ORDER BY (tenant_id, event_type, timestamp)
PARTITION BY (tenant_id % 100, toYYYYMM(timestamp))
```

Particao por tenant distribui dados. % 100 evita muitas particoes.

Toda query passa por middleware que adiciona `AND tenant_id = ?`. Impossivel esquecer.

## Riscos

- Bug no middleware vaza dados - mitigacao: code review rigido, testes de isolamento, query audit
- DELETE de tenant grande e lento - mitigacao: TTL por tenant, soft delete com filtro
- Tenant grande afeta outros - mitigacao: monitoramento de queries por tenant, throttling
