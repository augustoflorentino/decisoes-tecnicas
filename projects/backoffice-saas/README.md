# Backoffice SaaS

Framework de backoffice reutilizavel para produtos internos. CRUD dinamico, permissoes granulares, auditoria.

## Stack

- NestJS
- Angular
- PostgreSQL
- Redis (cache de permissoes)

## ADRs

- [ADR-0001 - CRUD dinamico via metadata](ADR-0001-crud-dinamico.md)
- [ADR-0002 - Permissoes por recurso e acao](ADR-0002-permissoes.md)
- [ADR-0003 - Audit log imutavel com contexto](ADR-0003-auditoria.md)

## Trade-offs

- [TRD-0001 - Framework proprio vs admin generico](TRD-0001-build-vs-buy.md)
- [TRD-0002 - Permissoes em banco vs em codigo](TRD-0002-permissoes-storage.md)
