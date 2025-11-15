# ADR-0003: Audit log imutavel com contexto

- **Status:** Aceita
- **Tags:** auditoria, compliance, seguranca

## Contexto

Backoffice manipula dados sensiveis. Regulatorio exige saber quem fez o que e quando. Investigacao de incidente precisa de trilha completa.

Requisitos:
- Quem fez a acao (usuario, IP, sessao)
- O que foi feito (operacao, entidade, campos alterados)
- Quando (timestamp preciso)
- Valores antes e depois

## Decisao

Audit log em tabela append-only com contexto completo.

```sql
CREATE TABLE audit_log (
  id UUID,
  timestamp TIMESTAMPTZ,
  actor_id UUID,
  actor_email TEXT,
  actor_ip INET,
  session_id UUID,
  action TEXT,  -- 'users:update', 'payments:refund'
  resource_type TEXT,
  resource_id TEXT,
  changes JSONB,  -- {"email": {"old": "a@b.com", "new": "x@y.com"}}
  metadata JSONB  -- contexto extra (motivo, ticket relacionado)
)
```

Tabela sem UPDATE ou DELETE. Particao por mes pra performance.

Campos sensiveis mascarados no log:
```json
{"password": {"old": "[REDACTED]", "new": "[REDACTED]"}}
```

## Justificativa

**Log em arquivo**
- Simples
- Mas busca e dificil
- Rotacao pode perder dados

**Trigger de banco**
- Automatico
- Mas nao captura contexto (quem, IP)
- Performance impactada

**Log explicito na aplicacao**
- Controle total do que logar
- Contexto completo disponivel
- Mascaramento de dados sensiveis

Trade-off: precisa lembrar de logar. Mitigado com decorator automatico no CRUD dinamico.

## Consequencias

- Investigacao de incidente: "quem alterou X?" respondido em segundos
- Compliance: trilha completa pra auditores
- Decorator @Audited no controller gera log automatico
- Retencao de 5 anos (requisito legal)
- Elasticsearch indexa pra busca full-text
