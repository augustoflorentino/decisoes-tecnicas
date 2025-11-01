# TRD-0001: Providers proprios vs servico unificado

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao de criar o servico)

## Contexto

Temos 4 produtos internos. Cada um integra com providers de notificacao diretamente:
- Produto A: SendGrid + Firebase
- Produto B: Mailgun + OneSignal
- Produto C: SendGrid + Twilio
- Produto D: nenhum (quer adicionar)

Problemas:
- Credenciais espalhadas
- Logica de retry duplicada
- Sem visibilidade consolidada
- Produto D vai reinventar a roda

## Opcoes

### A: Manter cada produto com seus providers

- **Pros:** Autonomia, sem dependencia de servico central
- **Contras:** Duplicacao, inconsistencia, custo de manutencao

### B: Biblioteca compartilhada

- **Pros:** Reutiliza codigo, cada produto ainda e independente
- **Contras:** Versionamento de lib, deploy pra atualizar, ainda precisa de credenciais em cada lugar

### C: Servico centralizado de notificacoes

- **Pros:** Um lugar pra credenciais, metricas consolidadas, evolui uma vez
- **Contras:** Ponto unico de falha, dependencia entre times, latencia adicional

## Analise

Custo atual:
- 3 integracoes com SendGrid diferentes (codigo similar)
- 2 implementacoes de retry com bugs diferentes
- Nenhuma visibilidade de "quantas notificacoes enviamos no total"
- Produto D vai levar 2 sprints pra integrar do zero

Servico centralizado:
- Produto D integra em 2 dias (uma API)
- Metricas de todos os produtos em um dashboard
- Troca de provider (ex: SendGrid -> Resend) em um lugar

Risco de ponto unico de falha e real. Mitigacao: circuit breaker nos produtos, fallback pra envio direto em emergencia.

## Decisao

**Servico centralizado de notificacoes** (este projeto).

API simples:
```
POST /notifications
{
  "type": "order_update",
  "recipient": {"email": "...", "phone": "..."},
  "channels": ["email", "sms"],
  "data": {"order_id": "123", "status": "shipped"}
}
```

Produtos migram gradualmente. Providers antigos desligados apos migracao.

## Riscos

- Servico fora = ninguem envia - mitigacao: SLA alto, fallback local
- Gargalo de desenvolvimento - mitigacao: API estavel, mudancas internas nao afetam clientes
