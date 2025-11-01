# ADR-0001: Fila por canal com prioridade

- **Status:** Aceita
- **Tags:** filas, notificacoes, escalabilidade

## Contexto

Sistema envia notificacoes por 4 canais: email, SMS, push, WhatsApp. Cada canal tem caracteristicas diferentes:

- Email: alto volume, tolerante a delay
- SMS: custo alto, precisa ser rapido
- Push: volume massivo, delivery best-effort
- WhatsApp: rate limit agressivo do provider

Fila unica trava se um canal estiver lento.

## Decisao

Fila separada por canal + niveis de prioridade.

```
queue:email:high
queue:email:normal
queue:email:low
queue:sms:high
queue:sms:normal
...
```

Workers por canal, quantidade proporcional ao volume.

Prioridade definida pelo tipo de notificacao:
- High: OTP, confirmacao de pagamento, alertas criticos
- Normal: atualizacoes de pedido, lembretes
- Low: marketing, newsletters

## Justificativa

**Fila unica**
- Simples
- Mas canal lento bloqueia os outros
- Nao permite escalar por canal

**Fila por produto/cliente**
- Isolamento entre clientes
- Mas mesmo cliente tem canais diferentes
- Muitas filas pra gerenciar

**Fila por canal + prioridade**
- Escala independente por canal
- OTP passa na frente de marketing
- Rate limit de WhatsApp nao afeta email

## Consequencias

- OTP chega em < 5 segundos (fila high, poucos itens)
- Newsletter pode demorar horas (fila low, alto volume)
- Workers de WhatsApp limitados pra respeitar rate limit
- Dashboard mostra lag por canal/prioridade
