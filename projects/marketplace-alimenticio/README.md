# Marketplace Alimenticio

Backend de marketplace B2C focado em reducao de desperdicio. Produtos proximos da validade com desconto.

## Stack

- NestJS
- Prisma + PostgreSQL
- Redis (cache + filas)
- S3 (imagens)

## ADRs

- [ADR-0001 - Consistencia de estoque em alta concorrencia](ADR-0001-consistencia-estoque.md)
- [ADR-0002 - Cache multi-camada com invalidacao seletiva](ADR-0002-cache-multi-camada.md)
- [ADR-0003 - Soft delete com anonimizacao pra LGPD](ADR-0003-soft-delete-lgpd.md)
- [ADR-0004 - Particionamento de tabelas de historico](ADR-0004-particionamento-historico.md)
- [ADR-0005 - Calculo de preco dinamico baseado em validade](ADR-0005-preco-dinamico-validade.md)

## Trade-offs

- [TRD-0001 - Escolha de sistema de mensageria](TRD-0001-mensageria.md)
- [TRD-0002 - Monolito modular vs Microservices](TRD-0002-arquitetura-deploy.md)
- [TRD-0003 - Estrategia de autenticacao multi-role](TRD-0003-autenticacao-multitenancy.md)
- [TRD-0004 - Estrategia de concorrencia em estoque](TRD-0004-estrategia-concorrencia-estoque.md) → ADR-0001
