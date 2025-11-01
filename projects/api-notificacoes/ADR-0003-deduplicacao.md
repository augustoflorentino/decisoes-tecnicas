# ADR-0003: Deduplicacao de notificacoes por janela

- **Status:** Aceita
- **Tags:** deduplicacao, spam, experiencia

## Contexto

Bug no sistema de pedidos dispara 5 webhooks pro mesmo evento. Cada um gera notificacao. Cliente recebe 5 SMS iguais. Reclama. Custo de SMS x5.

Mesmo cenario legitimo: usuario clica 3 vezes no botao de reenviar codigo. Recebe 3 OTPs, so o ultimo vale.

## Decisao

Deduplicacao por chave composta em janela temporal.

```
Chave: {notification_type}:{recipient}:{content_hash}
TTL: configuravel por tipo (default 5 minutos)
```

Fluxo:
1. Notificacao chega
2. Gera chave de deduplicacao
3. Verifica se existe no Redis
4. Se existe, descarta (log de duplicata)
5. Se nao existe, processa e salva chave com TTL

Configuracao por tipo:
- OTP: janela de 30 segundos, content_hash inclui codigo
- Atualizacao de pedido: janela de 5 minutos
- Marketing: janela de 24 horas

## Justificativa

**Sem deduplicacao**
- Simples
- Mas spam acidental acontece
- Custo desnecessario

**Deduplicacao por ID externo**
- Depende do caller enviar ID unico
- Se nao enviar, nao funciona
- Nao pega duplicatas legitimas (retry do caller)

**Deduplicacao por conteudo + janela**
- Funciona mesmo se caller nao colaborar
- Janela configuravel por caso de uso
- Content hash pega duplicatas exatas

Trade-off: notificacao legitima identica dentro da janela e descartada. Raro e aceitavel.

## Consequencias

- Zero spam por bug de integracao
- Custo de SMS/WhatsApp controlado
- Log de duplicatas ajuda debug de integracoes problematicas
- Caller pode forcar envio com flag skip_dedup se necessario
