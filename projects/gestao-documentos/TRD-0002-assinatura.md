# TRD-0002: Assinatura digital ICP-Brasil vs assinatura simples

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada na feature de assinatura)

## Contexto

Sistema precisa permitir assinatura de documentos. Clientes variam:
- Empresa que precisa de validade juridica plena (ICP-Brasil)
- Startup que quer so rastreabilidade (quem assinou quando)

## Opcoes

### A: Apenas assinatura simples (click to sign)

- **Pros:** UX simples, sem certificado, gratuito
- **Contras:** Sem validade juridica garantida, facil de contestar

### B: Apenas ICP-Brasil

- **Pros:** Validade juridica plena, aceito em qualquer orgao
- **Contras:** Usuario precisa de certificado (e-CPF), UX complexa, custo

### C: Ambos (usuario escolhe)

- **Pros:** Atende todos os casos
- **Contras:** Complexidade de implementacao, dois fluxos

## Analise

Legislacao brasileira (MP 2.200-2):
- ICP-Brasil tem presuncao de validade (inversao de onus da prova)
- Assinatura simples e valida se partes concordarem
- Contratos ate certo valor nao exigem ICP

Casos de uso:
- Contrato de trabalho: ICP-Brasil preferivel
- Aceite de termos de uso: simples e suficiente
- Laudo medico: ICP-Brasil exigido por lei

## Decisao

**Ambos os tipos de assinatura**, selecionavel por tipo de documento.

Assinatura simples:
- Captura: IP, timestamp, hash do documento, aceite explicito
- Armazena evidencia em log imutavel
- Envelope de assinatura anexado ao PDF

Assinatura ICP-Brasil:
- Integracao com certificado A1 (arquivo) ou A3 (token/cartao)
- Assinatura no padrao PAdES (PDF)
- Validacao via cadeia ICP-Brasil

Configuracao por template de documento define qual tipo exigir.

## Riscos

- UX de ICP-Brasil afastar usuarios - mitigacao: guia passo a passo, suporte a A1 (mais simples)
- Certificado expirado no momento de assinar - mitigacao: validar antes de iniciar fluxo
