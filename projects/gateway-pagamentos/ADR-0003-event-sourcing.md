# ADR-0003: Event sourcing para transacoes financeiras

- **Status:** Aceita
- **Tags:** pagamentos, auditoria, event-sourcing

## Contexto

Transacao financeira passa por varios estados: criada, autorizada, capturada, estornada, cancelada. Regulatorio exige audit trail completo. Disputas de chargeback precisam de historico detalhado.

UPDATE em tabela perde historico. Trigger de auditoria e gambiarra. Precisa de solucao first-class.

## Decisao

Event sourcing para o dominio de transacoes.

Estado atual e derivado da sequencia de eventos. Eventos sao imutaveis.

```
Evento 1: TransactionCreated {id, amount, merchant, card_hash, timestamp}
Evento 2: TransactionAuthorized {id, acquirer, auth_code, timestamp}
Evento 3: TransactionCaptured {id, captured_amount, timestamp}
```

Tabela de eventos append-only. Projecao materializada pra queries.

```sql
-- eventos (append-only)
transaction_events (
  event_id, transaction_id, event_type, payload, timestamp
)

-- projecao (reconstruivel)
transactions (
  id, status, amount, merchant_id, ...
)
```

## Justificativa

**CRUD tradicional + tabela de auditoria**
- Auditoria e cidada de segunda classe
- Facil esquecer de logar algo
- Reconstruir estado em ponto no tempo e dificil

**Soft delete + campos de historico**
- Nao escala pra multiplas transicoes
- Campos nullable proliferam
- Logica de "qual era o estado em X" fica complexa

**Event sourcing**
- Eventos sao a fonte de verdade
- Auditoria e automatica (eventos sao o log)
- Replay pra debug ou reconstrucao
- Projecoes podem ser recriadas se precisar de estrutura diferente

Trade-off: complexidade de implementacao. Queries simples ficam mais verbosas. Aceitavel porque auditoria perfeita e requisito regulatorio.

## Consequencias

- Auditoria completa sem esforco adicional
- Disputas de chargeback resolvidas em minutos (timeline completa)
- Possivel criar projecoes novas retroativamente
- Replay de eventos pra debug de bugs em producao
- Elasticsearch indexa eventos pra busca full-text
