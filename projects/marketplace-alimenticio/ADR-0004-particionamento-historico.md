# ADR-0004: Particionamento de tabelas de historico

- **Status:** Aceita
- **Tags:** banco, performance, historico

## Contexto

Tabela de logs de preco vai crescer rapido. Cada alteracao de preco (automatica por validade) gera registro. Estimativa: ~500k linhas/dia.

Sem particionar desde o inicio, em poucos meses:
- Queries de historico vao degradar
- Backup vai demorar demais
- VACUUM vai travar a tabela

## Decisao

Particionamento por range de data (mes) no Postgres nativo desde o dia 1.

```sql
-- preco_historico particionado por mes
preco_historico_2025_08
preco_historico_2025_09
...
```

Retencao: particoes > 12 meses movidas pra cold storage (S3 + Athena), depois dropadas do Postgres.

## Justificativa

**Sharding por produto_id**
- Distribui bem, mas query de "todos os precos do mes" fica lenta
- Complexidade de roteamento
- Nao resolve o problema de volume total

**Tabela separada por mes (manual)**
- Funciona, mas aplicacao precisa saber qual tabela consultar
- Migracao de dados manual
- Propenso a erro

**Particionamento nativo**
- Postgres roteia automatico
- DROP PARTITION e instantaneo (sem VACUUM)
- Backup pode ser por particao
- Aplicacao nao muda (query normal)

Trade-off: schema migration mais chato. Criar particao nova todo mes via job automatizado.

## Consequencias

- Queries de historico rapidas (indice local na particao)
- Backup leve (so particoes recentes)
- Job mensal cria particao do proximo mes (cron no primeiro dia)
- Dados antigos acessiveis via Athena quando necessario
- Prisma nao entende particao nativamente - raw query pra DDL
