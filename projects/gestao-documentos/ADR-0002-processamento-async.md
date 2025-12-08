# ADR-0002: Processamento assincrono com status tracking

- **Status:** Aceita
- **Tags:** processamento, async, filas

## Contexto

Upload de documento dispara processamentos:
- Extracao de texto (OCR)
- Geracao de thumbnail
- Validacao de assinatura (se PDF assinado)
- Indexacao pra busca

Alguns demoram segundos, outros minutos. Usuario nao pode esperar.

## Decisao

Upload retorna imediatamente. Processamento via fila com status tracking.

```sql
document_jobs (
  id UUID,
  document_id UUID,
  job_type TEXT,  -- 'ocr', 'thumbnail', 'index'
  status TEXT,    -- 'pending', 'processing', 'completed', 'failed'
  result JSONB,
  error TEXT,
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ
)
```

Fluxo:
1. Upload completo -> cria documento com status=uploaded
2. Enfileira jobs de processamento
3. Workers processam async
4. Frontend consulta status via polling ou WebSocket
5. Documento fica status=ready quando todos jobs terminam

UI mostra progresso:
```
[✓] Upload
[✓] Gerando preview
[◌] Extraindo texto
[ ] Indexando
```

## Justificativa

**Processamento sincrono**
- Simples
- Mas timeout em arquivos grandes
- Experiencia ruim (usuario espera minutos)

**Fire-and-forget (sem tracking)**
- Rapido pra usuario
- Mas sem visibilidade do progresso
- Erro silencioso

**Async com status tracking**
- Usuario recebe feedback imediato
- Progresso visivel
- Retry de jobs falhos
- Dashboard de jobs pra ops

## Consequencias

- Upload de 50MB retorna em 2 segundos
- OCR de documento longo roda em background
- Retry automatico de jobs falhos (3x)
- Dashboard mostra fila de processamento
