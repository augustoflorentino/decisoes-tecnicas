# TRD-0001: Integracao direta vs iPaaS

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao de arquitetura inicial)

## Contexto

Precisamos integrar com 3 ERPs: SAP, TOTVS, Sankhya. Cada um tem API diferente (REST, SOAP, arquivo).

## Opcoes

### A: Integracao direta (codigo pra cada ERP)

- **Pros:** Controle total, sem custo de licenca, customizacao livre
- **Contras:** Manutencao de N conectores, expertise em cada ERP

### B: iPaaS (MuleSoft, Dell Boomi, etc)

- **Pros:** Conectores prontos, interface visual, menos codigo
- **Contras:** Custo alto ($$$/mes), vendor lock-in, limitacoes do low-code

### C: Plataforma open-source (Apache Camel, n8n)

- **Pros:** Sem custo de licenca, padronizado, comunidade
- **Contras:** Curva de aprendizado, menos conectores prontos que comercial

### D: Hibrido (core proprio + conectores padronizados)

- **Pros:** Controle do core, conectores seguem padrao, extensivel
- **Contras:** Esforco inicial de definir padrao

## Analise

iPaaS comercial:
- MuleSoft: ~$50k/ano minimo
- Boomi: ~$30k/ano
- Conectores SAP prontos, mas customizacao limitada

Custo de desenvolver internamente:
- ~3 meses pra base + primeiro conector
- ~1 mes por conector adicional

Em 2 anos, custo interno e menor que iPaaS. Alem disso:
- Logica de negocio especifica nao cabe em low-code
- Debugging de iPaaS e caixa preta
- Dependencia de vendor pra evolucoes

## Decisao

**Plataforma propria** com arquitetura de conectores padronizada.

Arquitetura:
```
Core Engine
  -> Connector Interface (padrao)
    -> SAP Connector
    -> TOTVS Connector
    -> Sankhya Connector
```

Cada conector implementa interface padrao. Core cuida de fila, retry, transformacao, log.

Novo ERP = novo conector seguindo interface.

## Riscos

- Reinventar roda - mitigacao: escopo focado no que iPaaS nao resolve bem
- Manter conectores atualizado quando ERP muda API - mitigacao: testes de contrato, alerta de breaking change
