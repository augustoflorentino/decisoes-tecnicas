# TRD-0003: Estrategia de autenticacao multi-role

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada na stack inicial)

## Contexto

Tres tipos de usuario: vendedor, cliente, admin. Cada um acessa partes diferentes do sistema. Vendedor ve seus produtos, cliente ve catalogo, admin ve tudo.

Problema: como separar permissoes sem complicar demais? Tem que ser seguro mas sem virar burocracia de enterprise.

Restricoes:
- Time pequeno, nao da pra manter sistema complexo de permissoes
- Precisa funcionar em mobile (app futuro)
- Vendedor pode ter funcionarios com acesso limitado

## Opcoes

### A: JWT com roles no token
- **Pros:** Stateless, facil de validar, funciona bem com mobile, escala horizontal trivial
- **Contras:** Revogacao de token e complicada, token grande se tiver muita claim, refresh token precisa de cuidado

### B: Session tradicional (Redis)
- **Pros:** Revogacao instantanea, session data flexivel, familiar
- **Contras:** Lookup no Redis em todo request, problema se Redis cair, CORS mais chato

### C: JWT + blacklist no Redis
- **Pros:** Melhor dos dois mundos, revogacao funciona, ainda e stateless pro caso comum
- **Contras:** Complexidade adicional, blacklist pode crescer, precisa limpar periodicamente

## Decisao

**Opcao A: JWT com roles no token**, aceitando as limitacoes.

Token estruturado:
```json
{
  "sub": "user_id",
  "type": "vendedor|cliente|admin",
  "store_id": "se vendedor",
  "permissions": ["products:write", "orders:read"],
  "exp": "15min"
}
```

Refresh token com duracao longa (7 dias) armazenado em httpOnly cookie. Access token curto (15min) mitiga problema de revogacao - no pior caso, acesso indevido dura 15min.

Para casos criticos (vendedor demitido, conta comprometida), endpoint de logout forca troca do refresh token no proximo request.

## Riscos

- Token vazado fica valido por 15min - mitigacao: tempo curto, monitorar padroes anomalos
- Permissions no token podem ficar desatualizadas - mitigacao: refresh token atualiza claims, role changes forcam re-login
- Funcionario de vendedor com escopo errado - mitigacao: permissions granulares checadas no backend, nunca confiar so no token

## Notas

Guard no NestJS checa role + permissions. Decorator customizado pra simplificar:

```typescript
@RequiresRole('vendedor')
@RequiresPermission('products:write')
updateProduct() { ... }
```

Permissions sao aditivas. Vendedor owner tem todas, funcionario tem subset.
