# ADR-0003: Soft delete com anonimizacao pra LGPD

- **Status:** Aceita
- **Tags:** lgpd, dados, compliance

## Contexto

Usuario pode pedir exclusao de conta. LGPD exige que dados pessoais sejam removidos. Mas:

- Pedidos antigos precisam existir pra contabilidade (5 anos minimo)
- Vendedor precisa do historico pra contestar disputas
- Metricas agregadas nao podem quebrar

Deletar fisicamente quebra integridade referencial e historico.

## Decisao

Soft delete + anonimizacao seletiva.

```
Usuario pede exclusao:
1. Flag deleted_at = now()
2. Job assincrono anonimiza dados pessoais:
   - nome → "Usuario Removido"
   - email → hash@removed.internal
   - telefone → null
   - endereco → null
3. Pedidos mantem: id, valores, status, timestamps
4. Avaliacoes mantem: nota, texto (se nao identificavel)
```

Retencao: dados anonimizados ficam 5 anos, depois hard delete.

## Justificativa

**Hard delete imediato**
- Quebra FK dos pedidos
- Perde historico de vendas
- Auditoria fica impossivel

**Soft delete sem anonimizar**
- Dado pessoal continua la, so escondido
- Nao atende LGPD de verdade
- Risco juridico

**Soft delete + anonimizacao**
- Atende LGPD (dado pessoal some)
- Mantem integridade referencial
- Historico preservado sem identificar pessoa

Trade-off: queries precisam filtrar deleted_at. Indice parcial ajuda, mas e mais uma coisa pra lembrar.

## Consequencias

- Compliant com LGPD desde o inicio
- Job de anonimizacao roda em ate 72h (exigencia legal)
- Relatorios precisam de logica pra "Usuario Removido"
- Backup tambem precisa respeitar (restaurar nao pode "ressuscitar" dados)
