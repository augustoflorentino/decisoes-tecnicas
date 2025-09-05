# ADR-0003: Rate limiting por instancia com sliding window

- **Status:** Aceita
- **Tags:** whatsapp, rate-limit, redis

## Contexto

WhatsApp bane numeros que enviam muitas mensagens em curto periodo. Nao existe documentacao oficial de limites, mas empiricamente:

- ~60 msgs/minuto por numero e seguro
- Acima disso, risco de ban temporario
- Muito acima, ban permanente

Sistema vai ter multiplas instancias (numeros) de clientes diferentes. Cada um com seu limite.

## Decisao

Rate limiting com sliding window log no Redis, por instancia.

```
Chave: ratelimit:{instance_id}
Tipo: sorted set
Score: timestamp do envio
```

Antes de enviar:
1. Remove entradas mais velhas que 60s
2. Conta entradas restantes
3. Se < 60, envia e adiciona timestamp
4. Se >= 60, calcula quando proximo slot abre e agenda

## Justificativa

**Fixed window (contador por minuto)**
- Problema de borda: 60 msgs no segundo 59, mais 60 no segundo 61 = 120 em 2 segundos
- WhatsApp ve burst, bane

**Token bucket**
- Mais complexo de implementar
- Permite burst controlado, mas queremos evitar burst completamente

**Sliding window log**
- Janela real de 60 segundos
- Sem problema de borda
- Redis sorted set e eficiente pra isso
- Facil de debugar (ve exatamente quando cada msg foi)

Trade-off: usa mais memoria que contador simples. Aceitavel pro volume (~1KB por instancia ativa).

## Consequencias

- Zero bans por rate limit desde implementacao
- Mensagens de campanha distribuidas naturalmente ao longo do tempo
- Metricas de "fila de espera" por instancia disponiveis
- Sender service e unico ponto de saida - rate limit centralizado
