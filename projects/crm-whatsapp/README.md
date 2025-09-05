# CRM WhatsApp

Plataforma de automacao WhatsApp: chatbot com IA, disparador de campanhas e CRM de conversas.

## Stack

- NestJS (microservicos)
- Angular (frontend)
- Prisma + PostgreSQL (2 instancias)
- Redis (cache + filas)
- Nginx + Traefik (gateways)
- Evolution API (integracao WhatsApp)

## ADRs

- [ADR-0001 - Separacao de bancos por criticidade de dados](ADR-0001-separacao-bancos.md)
- [ADR-0002 - BFF para agregacao de servicos](ADR-0002-bff-agregacao.md)
- [ADR-0003 - Rate limiting por instancia com sliding window](ADR-0003-rate-limiting.md)
- [ADR-0004 - Cache-aside com invalidacao por evento](ADR-0004-cache-invalidacao.md)

## Trade-offs

- [TRD-0001 - Microservicos com banco compartilhado](TRD-0001-banco-compartilhado.md)
- [TRD-0002 - Dual gateway externo e interno](TRD-0002-dual-gateway.md)
- [TRD-0003 - Estrategia de retry e dead letter queue](TRD-0003-retry-dlq.md)
