# ADR-0001: Ledger de pontos com double-entry

- **Status:** Aceita
- **Tags:** pontos, financeiro, auditoria
- **Trade-off:** [TRD-0001 - Pontos como moeda vs score](TRD-0001-modelo-pontos.md)

## Contexto

Pontos sao acumulados, resgatados, expiram, podem ser estornados. Precisa de:
- Saldo atual correto
- Historico de movimentacoes
- Auditoria (de onde veio cada ponto)
- Consistencia (nao criar pontos do nada)

Modelo simples (saldo como campo) nao garante consistencia.

## Decisao

Ledger com partidas dobradas (double-entry bookkeeping).

```sql
point_accounts (
  id UUID,
  type TEXT,  -- 'customer', 'campaign', 'expiration', 'redemption'
  owner_id UUID,  -- customer_id ou campaign_id
  balance BIGINT  -- saldo atual (cache, derivado do ledger)
)

point_transactions (
  id UUID,
  debit_account_id UUID,
  credit_account_id UUID,
  amount BIGINT,
  description TEXT,
  reference_type TEXT,  -- 'purchase', 'redemption', 'expiration', 'adjustment'
  reference_id UUID,
  created_at TIMESTAMPTZ
)
```

Regras:
- Toda transacao: debita uma conta, credita outra
- Soma de todos debitos = soma de todos creditos (invariante)
- Saldo = creditos - debitos da conta
- Campo balance e cache (recalculavel)

Exemplo de acumulo:
```
Debit: campaign:blackfriday (origem dos pontos)
Credit: customer:123 (recebe pontos)
Amount: 1000
```

## Justificativa

**Campo saldo simples**
- UPDATE balance = balance + X
- Mas se transacao falhar no meio, saldo fica errado
- Sem historico de origem

**Log de eventos (event sourcing)**
- Historico completo
- Mas saldo precisa ser calculado sempre
- Sem garantia de balanco

**Double-entry ledger**
- Consistencia por design (debito = credito)
- Historico completo
- Auditoria trivial
- Padrao de mercado financeiro

## Consequencias

- Impossivel criar pontos sem origem
- Estorno e transacao inversa (nao delete)
- Relatorio de movimentacao trivial
- Reconciliacao: soma de saldos = 0
