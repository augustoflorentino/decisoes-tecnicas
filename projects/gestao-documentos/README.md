# Gestao Documentos

Sistema de gestao documental com upload, processamento, OCR e assinatura digital.

## Stack

- NestJS
- PostgreSQL
- MinIO (storage)
- Redis (filas)
- Tesseract (OCR)
- ICP-Brasil (assinatura)

## ADRs

- [ADR-0001 - Storage em MinIO com referencia no banco](ADR-0001-storage.md)
- [ADR-0002 - Processamento assincrono com status tracking](ADR-0002-processamento-async.md)
- [ADR-0003 - Versionamento de documentos com imutabilidade](ADR-0003-versionamento.md)

## Trade-offs

- [TRD-0001 - OCR local vs servico externo](TRD-0001-ocr.md)
- [TRD-0002 - Assinatura digital ICP-Brasil vs assinatura simples](TRD-0002-assinatura.md)
