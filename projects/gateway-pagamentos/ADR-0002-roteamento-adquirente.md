# ADR-0002: Roteamento dinamico de adquirente

- **Status:** Aceita
- **Tags:** pagamentos, roteamento, disponibilidade
- **Trade-off:** [TRD-0003 - Estrategia de fallback entre adquirentes](TRD-0003-fallback-adquirente.md)

## Contexto

Sistema integra com 4 adquirentes: Cielo, Rede, Stone, PagSeguro. Cada merchant tem contrato com 1 ou mais.

Problemas a resolver:
- Adquirente X fora do ar, transacao tem que passar por outro
- Merchant quer rotear por custo (taxa menor primeiro)
- Algumas bandeiras so funcionam em alguns adquirentes
- Taxa de aprovacao varia por adquirente/bandeira/valor

## Decisao

Engine de roteamento com regras em camadas:

```
1. Filtro de elegibilidade (bandeira suportada, merchant habilitado)
2. Ordenacao por prioridade (custo, taxa aprovacao, regra custom)
3. Fallback automatico se primeiro falhar
```

Configuracao por merchant em JSON:

```json
{
  "rules": [
    {"acquirer": "stone", "priority": 1, "conditions": {"brand": ["visa", "master"]}},
    {"acquirer": "cielo", "priority": 2, "conditions": {"brand": ["*"]}}
  ],
  "fallback": {"enabled": true, "max_attempts": 2}
}
```

Health check continuo dos adquirentes. Se um cair, removido temporariamente do pool.

## Justificativa

**Roteamento fixo (sempre o mesmo adquirente)**
- Simples
- Mas se cair, para tudo
- Nao otimiza custo

**Round-robin**
- Distribui carga
- Mas ignora custo e taxa de aprovacao
- Transacao pode ir pro adquirente errado

**Roteamento por regras + fallback**
- Merchant configura prioridade
- Sistema respeita mas tem autonomia pra fallback
- Metricas de cada adquirente alimentam decisao

Trade-off: complexidade de configuracao. Mitigado com defaults sensiveis e UI de configuracao.

## Consequencias

- Uptime efetivo maior que qualquer adquirente individual
- Merchants economizam com roteamento por custo
- Dashboard mostra performance por adquirente (ajuda renegociar contrato)
- Fallback transparente - cliente final nao percebe
