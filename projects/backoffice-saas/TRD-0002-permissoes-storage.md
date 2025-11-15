# TRD-0002: Permissoes em banco vs em codigo

- **Status:** Concluida
- **ADR resultante:** [ADR-0002 - Permissoes por recurso e acao](ADR-0002-permissoes.md)

## Contexto

Sistema de permissoes precisa de:
- Definicao de quais permissoes existem
- Atribuicao de permissoes a roles/usuarios
- Verificacao em runtime

Onde guardar a definicao das permissoes?

## Opcoes

### A: Hardcoded em constantes

```typescript
const PERMISSIONS = {
  USERS_VIEW: 'users:view',
  USERS_EDIT: 'users:edit',
}
```

- **Pros:** Type-safe, autocomplete, sem consulta ao banco
- **Contras:** Deploy pra adicionar permissao, nao da pra criar roles dinamicamente

### B: Banco de dados

```sql
permissions (id, name, description)
role_permissions (role_id, permission_id)
```

- **Pros:** Dinamico, admin pode criar roles/permissoes
- **Contras:** Typo em nome de permissao nao e pego em compile time, lookup em toda checagem

### C: Hibrido (definicao em codigo, atribuicao em banco)

Permissoes definidas em codigo (enum/const). Roles e atribuicoes em banco.

- **Pros:** Permissoes sao type-safe, roles sao dinamicas
- **Contras:** Nova permissao requer deploy (aceitavel)

## Analise

Permissoes novas sao raras (quando adiciona feature nova). Roles novas sao frequentes (cliente quer perfil custom).

Hibrido:
- `users:edit` em codigo - pego em compile se errar
- Role "Suporte N2" criada pelo admin - flexivel

Cache de permissoes do usuario no Redis. Lookup sem ir no banco.

## Decisao

**Hibrido**: permissoes em codigo, roles e atribuicoes em banco.

```typescript
// codigo - type safe
export const Permissions = {
  users: ['view', 'edit', 'delete'] as const,
  payments: ['view', 'refund'] as const,
}

// banco - dinamico
roles (id, name, permissions: text[])
user_roles (user_id, role_id)
```

Seed popula roles padrao. Admin cria novas combinando permissoes existentes.

## Riscos

- Permissao removida do codigo mas role ainda referencia - mitigacao: migration valida consistencia
- Cache desatualizado - mitigacao: invalidar cache quando role/usuario atualizado
