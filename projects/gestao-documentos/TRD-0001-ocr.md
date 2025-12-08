# TRD-0001: OCR local vs servico externo

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada no processamento)

## Contexto

Documentos escaneados precisam de OCR pra virar texto pesquisavel. Volume: ~1000 paginas/dia.

## Opcoes

### A: Tesseract (local, open source)

- **Pros:** Gratuito, sem limite, dados ficam internos
- **Contras:** Qualidade inferior em documentos complexos, precisa de GPU pra performance

### B: Google Vision API

- **Pros:** Qualidade excelente, handwriting, tabelas
- **Contras:** Custo por pagina ($1.50/1000 paginas), dados vao pra Google

### C: AWS Textract

- **Pros:** Qualidade boa, integracao AWS, extrai estrutura (tabelas, forms)
- **Contras:** Custo maior que Vision, vendor lock-in

### D: Azure Form Recognizer

- **Pros:** Bom pra formularios estruturados
- **Contras:** Custo, menos flexivel pra documentos livres

## Analise

Tipos de documento:
- 70%: notas fiscais, contratos (texto impresso limpo)
- 20%: documentos escaneados (qualidade variavel)
- 10%: formularios preenchidos a mao

Tesseract resolve 70% bem. Os 30% restantes precisam de qualidade melhor.

Custo Google Vision: 1000 paginas/dia = 30k/mes = $45/mes.
Custo Tesseract: infra de GPU = ~$100/mes.

Hibrido possivel: Tesseract primeiro, se confianca baixa manda pra Vision.

## Decisao

**Tesseract local como padrao**, Google Vision como fallback.

Fluxo:
1. OCR com Tesseract
2. Se confianca media < 70%, envia pra Google Vision
3. Resultado com maior confianca e usado

Documentos sensiveis: flag pra nunca mandar pra externo (usa so Tesseract).

## Riscos

- Qualidade insuficiente em casos edge - mitigacao: fallback pra Vision, review manual
- Custo de GPU - mitigacao: batch processing fora de horario de pico
