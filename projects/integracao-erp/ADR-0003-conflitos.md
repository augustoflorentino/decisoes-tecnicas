# ADR-0003: Resolucao de conflitos com last-write-wins qualificado

- **Status:** Aceita
- **Tags:** conflito, sync, consistencia

## Contexto

Mesmo registro editado nos dois sistemas entre syncs.

Exemplo:
- 10:00 - Produto no ERP: preco = 100
- 10:00 - Produto no nosso: preco = 100
- 10:05 - Usuario no ERP altera pra 110
- 10:07 - Usuario no nosso altera pra 95
- 10:10 - Sync roda. Qual preco vale?

## Decisao

Last-write-wins qualificado por campo, com log de conflito.

```sql
sync_conflicts (
  id UUID,
  entity_type TEXT,
  entity_id UUID,
  field TEXT,
  our_value JSONB,
  erp_value JSONB,
  our_updated_at TIMESTAMPTZ,
  erp_updated_at TIMESTAMPTZ,
  resolution TEXT,  -- 'ours', 'erp', 'manual'
  resolved_at TIMESTAMPTZ
)
```

Regras por tipo de campo:
- Preco: ERP e fonte de verdade (last-write do ERP ganha)
- Estoque: ERP e fonte de verdade
- Descricao marketing: nosso sistema e fonte de verdade
- Status ativo: mais restritivo ganha (se qualquer um desativou, desativa)

Conflito detectado: registra, aplica regra, notifica se necessario.

## Justificativa

**Last-write-wins simples (timestamp)**
- Facil de implementar
- Mas campo errado pode ganhar
- Clock skew entre sistemas

**Manual sempre (bloqueia ate resolver)**
- Consistencia garantida
- Mas operador vira gargalo
- UX ruim

**Fonte de verdade por campo**
- Regra de negocio define quem manda
- Automatico na maioria dos casos
- Manual so quando ambiguo

## Consequencias

- Conflitos de preco: ERP sempre ganha (fiscalidade)
- Conflitos de descricao: nosso sistema ganha (marketing)
- Log de conflitos pra auditoria
- Dashboard mostra conflitos do dia
