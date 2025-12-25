# ADR-0002: Fila de sincronizacao com retry e dead letter

- **Status:** Aceita
- **Tags:** sync, filas, resiliencia

## Contexto

Sincronizacao com ERP falha por varios motivos:
- ERP fora do ar (manutencao noturna)
- Timeout (operacao pesada)
- Erro de validacao (dado invalido no ERP)
- Rate limit

Sistema precisa ser resiliente sem perder dados.

## Decisao

Fila de sincronizacao com estrategia de retry diferenciada.

```sql
sync_jobs (
  id UUID,
  entity_type TEXT,  -- 'product', 'order', 'inventory'
  entity_id UUID,
  direction TEXT,    -- 'to_erp', 'from_erp'
  payload JSONB,
  status TEXT,       -- 'pending', 'processing', 'completed', 'failed', 'dead'
  attempts INT,
  last_error TEXT,
  next_retry_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ
)
```

Estrategia de retry por tipo de erro:
- Timeout/5xx: retry com backoff (1min, 5min, 30min, 2h)
- Rate limit: retry apos janela informada pelo ERP
- Validacao (4xx): dead letter imediato (nao vai resolver com retry)
- Desconhecido: retry 3x, depois dead letter

Dead letter queue tem dashboard pra operador revisar e reprocessar manualmente.

## Justificativa

**Retry infinito**
- Nunca perde dado
- Mas fila cresce se ERP fica fora por dias
- Esconde problemas

**Fail fast (sem retry)**
- Simples
- Mas perde dado em falha transiente
- Usuario tem que reenviar manualmente

**Retry com limite + dead letter**
- Falhas transitorias resolvem sozinhas
- Falhas permanentes vao pra analise
- Fila nao cresce infinitamente

## Consequencias

- 95% dos erros transitorios resolvem com retry
- Dead letter tem < 1% dos jobs
- Dashboard de sync mostra status em tempo real
- Alerta quando dead letter cresce
