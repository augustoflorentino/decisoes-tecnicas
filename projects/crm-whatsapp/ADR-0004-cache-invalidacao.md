# ADR-0004: Cache-aside com invalidacao por evento

- **Status:** Aceita
- **Tags:** cache, redis, performance

## Contexto

CRM mostra lista de conversas com ultima mensagem, status, contato. Query envolve joins entre tabelas de mensagens e contatos. Em volume alto, fica lento.

Cache obvio, mas invalidacao e o problema. Mensagem nova chega a qualquer momento via webhook.

## Decisao

Padrao cache-aside com invalidacao explicita por evento.

**Leitura:**
1. Busca no Redis
2. Se existe, retorna
3. Se nao, busca no banco, salva no Redis com TTL, retorna

**Invalidacao:**
Webhook service publica evento no Redis pub/sub quando mensagem chega. CRM service escuta e invalida cache da conversa especifica.

```
Webhook -> Redis PUBLISH conversa:{id}:nova_msg
CRM     -> Redis SUBSCRIBE conversa:*:nova_msg -> DEL cache:conversa:{id}
```

## Justificativa

**Write-through (escreve no cache junto com banco)**
- Webhook teria que conhecer estrutura do cache do CRM
- Acoplamento entre servicos
- Cache pode ficar inconsistente se logica mudar

**TTL curto sem invalidacao**
- Simples, mas usuario ve mensagem com delay
- Inaceitavel pra CRM de atendimento em tempo real

**Cache-aside com invalidacao por evento**
- Servicos desacoplados (webhook so publica evento generico)
- CRM decide o que invalidar
- Consistencia em < 50ms na pratica

Trade-off: complexidade de pub/sub. Redis ja ta no stack, custo marginal.

## Consequencias

- Lista de conversas responde em < 20ms (cache hit)
- Mensagem nova aparece em < 1 segundo
- Menos load no banco de mensagens
- Painel de metricas mostra cache hit rate (~85%)
