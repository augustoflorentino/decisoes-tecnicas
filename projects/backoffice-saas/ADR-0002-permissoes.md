# ADR-0002: Permissoes por recurso e acao

- **Status:** Aceita
- **Tags:** permissoes, seguranca, rbac

## Contexto

Backoffice usado por varios perfis:
- Suporte: ve usuarios, nao edita pagamentos
- Financeiro: ve e edita pagamentos, nao ve dados sensiveis
- Admin: tudo
- Auditor: ve tudo, nao edita nada

Sistema simples de roles (admin/user) nao cobre esses casos.

## Decisao

Permissoes granulares: recurso + acao.

```
users:view
users:edit
users:delete
payments:view
payments:refund
logs:view
```

Roles agrupam permissoes:
```
role:support = [users:view, orders:view, orders:edit]
role:finance = [payments:*, reports:view]
role:admin = [*:*]
```

Usuario pode ter role + permissoes extras (ou revogadas).

Verificacao em dois pontos:
1. API: middleware checa permissao antes de processar
2. UI: esconde botoes/menus que usuario nao pode usar

## Justificativa

**Roles fixas (admin, editor, viewer)**
- Simples
- Mas nao cobre casos intermediarios
- Criacao de roles especificas prolifera

**ACL por registro (user X pode ver order Y)**
- Granularidade maxima
- Mas complexidade alta
- Performance de checagem ruim

**RBAC com permissoes granulares**
- Flexibilidade boa
- Performance ok (cache de permissoes)
- Modelo mental claro (recurso:acao)

## Consequencias

- Novo perfil criado em minutos (combinacao de permissoes)
- UI reflete permissoes (sem botao que vai dar 403)
- Cache de permissoes no Redis (checagem em < 1ms)
- Auditoria sabe exatamente o que cada role podia fazer
