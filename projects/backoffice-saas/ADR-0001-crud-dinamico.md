# ADR-0001: CRUD dinamico via metadata

- **Status:** Aceita
- **Tags:** crud, admin, produtividade

## Contexto

Cada produto precisa de backoffice pra operacoes internas: ver usuarios, editar configs, consultar logs. Hoje cada produto implementa do zero.

Problema: 80% das telas sao CRUD simples. Reimplementar tabela com filtro, paginacao, formulario de edicao toda vez e desperdicio.

## Decisao

CRUD gerado dinamicamente a partir de definicao de metadata.

```typescript
const entityConfig = {
  name: 'users',
  table: 'users',
  columns: [
    { name: 'id', type: 'uuid', primaryKey: true },
    { name: 'email', type: 'string', searchable: true },
    { name: 'plan', type: 'enum', options: ['free', 'pro'], filterable: true },
    { name: 'created_at', type: 'datetime', sortable: true }
  ],
  actions: ['view', 'edit', 'delete'],
  relations: [
    { name: 'orders', target: 'orders', type: 'one-to-many' }
  ]
}
```

Frontend renderiza tabela, filtros, formularios a partir do metadata. Backend gera queries dinamicamente.

## Justificativa

**CRUD manual por entidade**
- Controle total
- Mas tempo de desenvolvimento alto
- Inconsistencia entre telas

**Geradores de codigo (scaffolding)**
- Gera uma vez, depois customiza
- Mas atualizacao do gerador nao propaga
- Codigo gerado vira legacy

**CRUD dinamico via metadata**
- Define uma vez, renderiza automatico
- Atualizacao do framework beneficia todos
- Customizacao via hooks quando necessario

Trade-off: casos muito custom nao cabem no framework. Esses continuam manuais.

## Consequencias

- Nova entidade CRUD em 10 minutos (so metadata)
- 15 telas de admin geradas, 3 manuais (casos especiais)
- Bug no componente de tabela corrigido uma vez, todos ganham
- Onboarding rapido (padrao conhecido)
