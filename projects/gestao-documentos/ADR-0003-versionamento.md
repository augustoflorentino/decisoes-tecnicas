# ADR-0003: Versionamento de documentos com imutabilidade

- **Status:** Aceita
- **Tags:** versionamento, imutabilidade, auditoria

## Contexto

Contrato precisa de revisao. Usuario faz upload de versao atualizada. Sistema precisa:
- Manter versao anterior acessivel
- Saber quem alterou quando
- Garantir que versao antiga nao foi modificada

Requisito legal: documento assinado nao pode ser alterado.

## Decisao

Documentos sao imutaveis. Nova versao cria novo registro linkado ao anterior.

```sql
documents (
  id UUID,
  parent_id UUID,     -- versao anterior (null se primeira)
  version INT,        -- 1, 2, 3...
  storage_key TEXT,
  checksum TEXT,
  is_latest BOOLEAN,
  ...
)
```

Regras:
- Upload de nova versao: cria novo documento com parent_id = documento atual, is_latest = true no novo, false no antigo
- Documento assinado: flag locked = true, nova versao bloqueada
- Acesso a versao especifica: por document_id
- Acesso a "ultima versao": WHERE parent chain, is_latest = true

Checksum calculado no upload. Verificado em cada download.

## Justificativa

**Overwrite (substitui arquivo)**
- Simples
- Mas perde historico
- Nao cumpre requisito legal

**Versionamento no storage (S3 versioning)**
- Storage cuida
- Mas metadata no banco desconectado
- Dificil saber quem fez qual versao

**Registro por versao (imutavel)**
- Historico completo no banco
- Auditoria trivial
- Integridade verificavel (checksum)

## Consequencias

- Timeline de versoes visivel na UI
- Qualquer versao pode ser baixada
- Impossivel alterar documento ja assinado
- Comparacao entre versoes possivel (diff de texto extraido)
- Storage cresce com versoes (aceitavel, documentos sao permanentes)
