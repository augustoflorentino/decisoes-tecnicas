# TRD-0003: Estrategia de retry e dead letter queue

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada no design de filas)

## Contexto

Envio de mensagem pode falhar por varios motivos:
- Evolution API fora do ar
- Numero invalido
- WhatsApp do destinatario offline
- Rate limit do WhatsApp

Precisa de retry inteligente sem:
- Perder mensagem
- Spammar um numero com erro permanente
- Travar fila por causa de um item problematico

## Opcoes

### A: Retry infinito com backoff

- **Pros:** Nunca perde mensagem
- **Contras:** Fila cresce infinitamente se erro persistente, item problematico nunca sai

### B: Retry limitado, descarta depois

- **Pros:** Fila nao trava
- **Contras:** Perde mensagem, usuario nao sabe que falhou

### C: Retry limitado + Dead Letter Queue

- **Pros:** Fila principal flui, mensagens problematicas separadas pra analise
- **Contras:** Precisa de processo pra tratar DLQ

### D: Classificacao de erro + estrategia por tipo

```
Erro transitorio (timeout, 503) -> retry com backoff
Erro permanente (numero invalido) -> DLQ imediato
```

- **Pros:** Nao gasta retry em erro que nunca vai resolver
- **Contras:** Precisa classificar erros corretamente

## Analise

Maioria das falhas sao transitorias (Evolution reiniciando, rede instavel). Retry resolve.

Algumas sao permanentes (numero nao existe). Retry so atrasa o inevitavel.

Classificar erros da Evolution API:
- `400 Bad Request` -> permanente (payload errado)
- `404 Not Found` -> permanente (numero invalido)
- `429 Too Many Requests` -> transitorio (rate limit)
- `500/502/503/504` -> transitorio (infra)
- Timeout -> transitorio

## Decisao

**Opcao D: Classificacao de erro + DLQ**

```
Erro transitorio:
  - Tentativa 1: imediato
  - Tentativa 2: +5 segundos
  - Tentativa 3: +30 segundos
  - Tentativa 4: +5 minutos
  - Apos 4: DLQ

Erro permanente:
  - DLQ imediato
```

DLQ tem dashboard pra operador revisar. Pode reprocessar manualmente ou descartar.

## Riscos

- Classificar erro errado (transitorio como permanente) - mitigacao: log detalhado, errar pro lado de retry
- DLQ crescer sem ninguem olhar - mitigacao: alerta quando DLQ > 100 itens
- Usuario achar que mensagem foi enviada quando ta na DLQ - mitigacao: status "falha" visivel no CRM
