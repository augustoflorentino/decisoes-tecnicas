# TRD-0001: Framework proprio vs admin generico

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao de criar o framework)

## Contexto

Precisamos de backoffice pra 4 produtos. Opcoes:
1. Cada produto faz o seu
2. Usar solucao pronta (AdminJS, Retool, Forest Admin)
3. Construir framework interno

## Opcoes

### A: Backoffice por produto

- **Pros:** Autonomia total, sem dependencia
- **Contras:** Retrabalho, inconsistencia, manutencao multiplicada

### B: AdminJS (open source)

- **Pros:** Pronto, customizavel, comunidade
- **Contras:** Adapter limitado pro Prisma, visual nao alinhado, customizacao profunda e dificil

### C: Retool (SaaS)

- **Pros:** Muito rapido pra prototipar, drag-and-drop
- **Contras:** Custo por usuario alto ($$$), lock-in, dados sensiveis em terceiro

### D: Forest Admin (SaaS)

- **Pros:** Conecta direto no banco, UI pronta
- **Contras:** Custo alto, nuvem deles, customizacao limitada

### E: Framework proprio

- **Pros:** Controle total, alinhado com stack, sem custo por usuario
- **Contras:** Tempo de desenvolvimento, manutencao interna

## Analise

Custo de Retool: $50/usuario/mes. Com 20 usuarios = $12k/ano.
Tempo pra construir framework basico: ~3 meses.
Custo interno: menor que 1 ano de Retool.

Alem disso:
- Dados sensiveis (PII, pagamentos) ficam internos
- Customizacao profunda quando necessario
- Integracao nativa com auth existente

## Decisao

**Framework proprio** com CRUD dinamico.

Escopo inicial:
- CRUD via metadata
- Permissoes granulares
- Audit log
- Componentes reutilizaveis (tabela, form, filtros)

Produtos migram gradualmente. Casos muito custom continuam manuais.

## Riscos

- Virar projeto eterno - mitigacao: escopo fechado, "bom o suficiente" > perfeito
- Ninguem querer usar - mitigacao: dogfooding em produto interno primeiro
