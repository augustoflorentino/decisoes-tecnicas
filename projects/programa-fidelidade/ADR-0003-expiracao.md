# ADR-0003: Expiracao de pontos com janela de graca

- **Status:** Aceita
- **Tags:** expiracao, pontos, ux

## Contexto

Pontos precisam expirar (contabilidade, incentivo a uso). Regra comum: pontos expiram 12 meses apos acumulo.

Problemas:
- Cliente com 10k pontos perde tudo de uma vez
- Suporte recebe reclamacao "nao sabia que ia expirar"
- Cliente some por 13 meses, volta, pontos ja foram

## Decisao

Expiracao por lote com janela de graca e notificacoes.

```sql
point_batches (
  id UUID,
  customer_id UUID,
  points BIGINT,
  earned_at TIMESTAMPTZ,
  expires_at TIMESTAMPTZ,
  grace_until TIMESTAMPTZ,  -- 30 dias apos expires_at
  status TEXT  -- 'active', 'expiring', 'expired', 'used'
)
```

Regras:
- Pontos expiram 12 meses apos acumulo
- Janela de graca: 30 dias apos expiracao
- Durante graca: pontos nao podem ser acumulados mas podem ser resgatados
- Apos graca: expiracao definitiva

Notificacoes:
- 30 dias antes: "X pontos vao expirar"
- 7 dias antes: "ultima chance"
- No dia: "pontos em periodo de graca"

Consumo usa FIFO (pontos mais antigos primeiro).

## Justificativa

**Expiracao unica (todos os pontos juntos)**
- Simples
- Mas perda grande de uma vez
- UX ruim

**Sem expiracao**
- Cliente feliz
- Mas passivo contabil cresce indefinidamente
- Sem incentivo a uso

**Expiracao por lote com graca**
- Perdas pequenas e graduais
- Tempo pra reagir
- Ainda incentiva uso

## Consequencias

- Cliente recebe avisos antecipados
- Perda maxima = pontos de um mes (nao todos)
- Periodo de graca reduz reclamacoes
- Relatorio de pontos a expirar pra marketing (campanha de resgate)
