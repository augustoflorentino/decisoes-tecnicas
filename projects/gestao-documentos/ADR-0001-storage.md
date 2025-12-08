# ADR-0001: Storage em MinIO com referencia no banco

- **Status:** Aceita
- **Tags:** storage, arquivos, infraestrutura

## Contexto

Sistema armazena documentos: contratos, notas fiscais, comprovantes. Arquivos de 1KB a 50MB. Volume esperado: 500GB no primeiro ano.

Onde guardar os arquivos?

## Decisao

MinIO (S3-compatible) para arquivos. PostgreSQL para metadata e referencia.

```sql
documents (
  id UUID,
  name TEXT,
  mime_type TEXT,
  size_bytes BIGINT,
  storage_key TEXT,  -- 'documents/2025/01/abc123.pdf'
  checksum TEXT,     -- SHA256
  uploaded_at TIMESTAMPTZ,
  uploaded_by UUID
)
```

Organizacao no storage:
```
documents/
  {year}/
    {month}/
      {uuid}.{ext}
```

Acesso via presigned URLs com expiracao curta (15 min).

## Justificativa

**Arquivos no PostgreSQL (BYTEA)**
- Simples, transacional
- Mas banco incha, backup pesado
- Performance de leitura ruim pra arquivos grandes

**Filesystem local**
- Rapido, simples
- Mas nao escala horizontal
- Backup complicado
- Sem redundancia

**S3 (AWS)**
- Escala infinita, durabilidade alta
- Mas custo de egress, vendor lock-in
- Latencia pra operacoes frequentes

**MinIO (self-hosted S3)**
- API S3 compativel
- Controle total, sem custo de egress
- Pode migrar pra S3 real depois

## Consequencias

- Arquivos nao passam pelo backend (presigned URL direto pro MinIO)
- Backup de arquivos separado do banco
- Checksum garante integridade
- Migracao pra S3 e trocar endpoint
