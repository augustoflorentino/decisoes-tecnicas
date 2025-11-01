# TRD-0002: Retry por canal vs retry global

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada no design de filas)

## Contexto

Notificacao pedida pra 3 canais: email, SMS, push. Email falha (SendGrid fora). O que fazer?

Opcoes de retry:
1. Retry so o email, SMS e push ja foram
2. Retry a notificacao inteira (reenvia tudo)

## Opcoes

### A: Retry global (notificacao inteira)

- **Pros:** Simples, logica unica
- **Contras:** Reenvia canais que ja deram certo, spam

### B: Retry por canal (independente)

- **Pros:** So retenta o que falhou, sem spam
- **Contras:** Mais complexo, precisa trackear status por canal

### C: Retry por canal com compensacao

Se um canal critico falha apos N tentativas, tenta canal alternativo.

- **Pros:** Maximiza chance de entrega
- **Contras:** Pode entregar por canal nao preferido

## Analise

Cenario real:
- Notificacao importante: email (principal) + push (backup)
- Email falha, push funciona
- Retry global: push duplicado
- Retry por canal: so email retenta

Outro cenario:
- OTP so por SMS
- SMS falha (Twilio fora)
- Retry por canal: tenta 3x, falha
- Com compensacao: tenta WhatsApp como fallback

## Decisao

**Retry por canal** com fallback configuravel.

Cada canal tem:
- Retry proprio (3 tentativas, backoff exponencial)
- Status independente (sent, failed, pending)
- Fallback opcional (se SMS falhar, tenta WhatsApp)

Notificacao so e "failed" se todos os canais falharam.

Configuracao por tipo de notificacao:
```json
{
  "type": "otp",
  "channels": ["sms"],
  "fallback": {"sms": "whatsapp"},
  "retry": {"attempts": 3, "backoff": "exponential"}
}
```

## Riscos

- Fallback pra canal mais caro - mitigacao: alerta de custo, limite de fallback por dia
- Complexidade de status - mitigacao: estado bem definido, dashboard claro
