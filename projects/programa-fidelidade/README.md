# Programa Fidelidade

Sistema de fidelizacao com acumulo de pontos, niveis, campanhas e resgates.

## Stack

- NestJS
- PostgreSQL
- Redis (cache de saldo)
- Bull (processamento de campanhas)

## ADRs

- [ADR-0001 - Ledger de pontos com double-entry](ADR-0001-ledger.md)
- [ADR-0002 - Niveis calculados vs armazenados](ADR-0002-niveis.md)
- [ADR-0003 - Expiracao de pontos com janela de graca](ADR-0003-expiracao.md)

## Trade-offs

- [TRD-0001 - Pontos como moeda vs pontos como score](TRD-0001-modelo-pontos.md) → ADR-0001
- [TRD-0002 - Campanhas em engine de regras vs hardcoded](TRD-0002-campanhas.md)
