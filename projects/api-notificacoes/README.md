# API Notificacoes

Servico centralizado de notificacoes para multiplos produtos. Unifica envio de email, SMS, push e WhatsApp.

## Stack

- NestJS
- PostgreSQL
- Redis (filas + rate limiting)
- Bull (processamento async)
- SendGrid (email)
- Twilio (SMS)
- Firebase (push)

## ADRs

- [ADR-0001 - Fila por canal com prioridade](ADR-0001-fila-por-canal.md)
- [ADR-0002 - Template engine com fallback por canal](ADR-0002-templates.md)
- [ADR-0003 - Deduplicacao de notificacoes por janela](ADR-0003-deduplicacao.md)

## Trade-offs

- [TRD-0001 - Providers proprios vs servico unificado](TRD-0001-unificacao.md)
- [TRD-0002 - Retry por canal vs retry global](TRD-0002-retry.md)
