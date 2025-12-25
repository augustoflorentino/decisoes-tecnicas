# Integracao ERP

Plataforma de integracao com ERPs (SAP, TOTVS, Sankhya). Sincronizacao de produtos, pedidos, estoque e financeiro.

## Stack

- NestJS
- PostgreSQL
- Redis (cache + filas)
- Bull (sync jobs)

## ADRs

- [ADR-0001 - Mapeamento de campos com transformacao bidirecional](ADR-0001-mapeamento.md)
- [ADR-0002 - Fila de sincronizacao com retry e dead letter](ADR-0002-fila-sync.md)
- [ADR-0003 - Resolucao de conflitos com last-write-wins qualificado](ADR-0003-conflitos.md)

## Trade-offs

- [TRD-0001 - Integracao direta vs iPaaS](TRD-0001-arquitetura.md)
- [TRD-0002 - Sync real-time vs batch periodico](TRD-0002-frequencia-sync.md)
