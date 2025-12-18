# ADR-0002: Niveis calculados vs armazenados

- **Status:** Aceita
- **Tags:** niveis, gamificacao, cache

## Contexto

Programa tem niveis: Bronze, Prata, Ouro, Diamante. Nivel determina beneficios (multiplicador de pontos, acesso exclusivo).

Nivel baseado em:
- Pontos acumulados nos ultimos 12 meses
- Ou valor gasto nos ultimos 12 meses

Como determinar nivel do cliente?

## Decisao

Nivel armazenado com recalculo periodico.

```sql
customer_tiers (
  customer_id UUID,
  current_tier TEXT,
  qualifying_points BIGINT,  -- pontos que contam pro nivel
  tier_expires_at DATE,      -- quando nivel expira se nao renovar
  last_evaluated_at TIMESTAMPTZ
)

tier_rules (
  tier TEXT,
  min_points BIGINT,
  multiplier DECIMAL,
  benefits JSONB
)
```

Recalculo:
- Job noturno avalia todos os clientes
- Cliente com atividade recente: recalculo imediato
- Nivel so sobe imediatamente, descida espera fim do periodo

## Justificativa

**Calculo em tempo real**
- Sempre atualizado
- Mas query pesada (soma 12 meses de transacoes)
- Latencia em toda requisicao

**Nivel fixo (nunca muda)**
- Simples
- Mas cliente perde incentivo de subir
- Nao reflete comportamento recente

**Armazenado com recalculo periodico**
- Leitura instantanea (campo no banco)
- Atualizacao controlada
- Regra de "protecao" (nivel so cai no fim do periodo)

Trade-off: nivel pode estar 1 dia desatualizado. Aceitavel porque:
- Subida e instantanea (webhook de compra dispara recalculo)
- Descida tem periodo de graca (nao surpreende cliente)

## Consequencias

- Dashboard mostra nivel sem consulta pesada
- Cliente que atinge threshold sobe na hora
- Cliente inativo so desce no fim do ano
- Comunicacao antecipada: "faltam X pontos pra manter Ouro"
