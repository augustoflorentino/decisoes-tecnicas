# TRD-0002: Campanhas em engine de regras vs hardcoded

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada no modulo de campanhas)

## Contexto

Marketing quer criar campanhas promocionais:
- "Pontos em dobro no fim de semana"
- "Triple points em compras acima de R$ 200"
- "Bonus de 500 pontos na primeira compra do mes"

Quem cria essas regras? Como sao processadas?

## Opcoes

### A: Hardcoded (dev implementa cada campanha)

- **Pros:** Flexibilidade total, qualquer logica possivel
- **Contras:** Deploy pra cada campanha, dev vira gargalo

### B: Engine de regras generica (tipo Drools)

- **Pros:** Marketing cria regras sem dev, muito flexivel
- **Contras:** Curva de aprendizado alta, debug dificil, overkill

### C: Templates de campanha configuraveis

```json
{
  "type": "multiplier",
  "multiplier": 2,
  "conditions": {
    "day_of_week": ["saturday", "sunday"],
    "min_purchase": 0
  },
  "valid_from": "2025-01-01",
  "valid_until": "2025-01-31"
}
```

- **Pros:** UI simples, tipos comuns cobertos, sem dev pra maioria
- **Contras:** Campanhas muito custom precisam de novo tipo

## Analise

80% das campanhas sao variacoes de:
- Multiplicador (2x, 3x pontos)
- Bonus fixo (ganhe X pontos)
- Condicao de valor minimo
- Condicao de data/horario
- Condicao de categoria de produto

Templates cobrem esses casos. Os 20% restantes (campanha maluca do CEO) podem ser hardcoded.

Engine generica e overhead desnecessario pra volume de campanhas (~10/mes).

## Decisao

**Templates de campanha** com tipos pre-definidos.

Tipos iniciais:
- `multiplier`: multiplica pontos por fator
- `bonus`: adiciona pontos fixos
- `first_purchase`: bonus na primeira compra
- `category_boost`: multiplicador por categoria

Condicoes combinaveis:
- Periodo (data inicio/fim)
- Dia da semana
- Horario
- Valor minimo
- Categoria de produto
- Cliente novo vs recorrente

UI de criacao de campanha com dropdowns e campos. Deploy zero pra casos comuns.

## Riscos

- Marketing pede campanha que nao cabe nos templates - mitigacao: priorizar implementacao de novo tipo, hardcode como excecao
- Regras conflitantes - mitigacao: preview de resultado antes de ativar, limite de campanhas simultaneas
