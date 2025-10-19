# ADR-0004: Retencao em camadas com downsampling

- **Status:** Aceita
- **Tags:** retencao, storage, custo

## Contexto

Dados de analytics crescem indefinidamente. Storage custa dinheiro. Mas cliente quer ver historico de anos atras.

Dilema: guardar tudo pra sempre (caro) ou deletar apos X dias (perde historico).

## Decisao

Retencao em camadas com downsampling progressivo.

```
Camada 1: Raw events
- Retencao: 90 dias
- Granularidade: evento individual
- Storage: ~80% do total

Camada 2: Agregado por hora
- Retencao: 1 ano
- Granularidade: metricas por hora
- Storage: ~15% do total

Camada 3: Agregado por dia
- Retencao: indefinido
- Granularidade: metricas por dia
- Storage: ~5% do total
```

Jobs noturnos:
1. Agregar eventos > 90 dias em hourly, deletar raw
2. Agregar hourly > 1 ano em daily, deletar hourly

## Justificativa

**Retencao unica (ex: 1 ano de tudo)**
- Simples
- Mas ou guarda demais (caro) ou de menos (perde historico)
- Nao reflete valor dos dados (recente vale mais)

**Compressao agressiva de dados antigos**
- Reduz storage
- Mas nao reduz volume de linhas
- Queries antigas continuam lentas

**Downsampling em camadas**
- Storage otimizado por valor dos dados
- Historico longo disponivel (agregado)
- Queries sobre dados antigos sao rapidas (menos linhas)
- Cliente pode pagar por retencao maior de raw se quiser

Trade-off: perde granularidade de evento apos 90 dias. Aceitavel porque 99% das queries olham ultimos 30 dias.

## Consequencias

- Storage cresce linear com novos clientes, nao com tempo
- Dashboard de 2 anos atras carrega rapido
- Custo previsivel por cliente
- Tier de pricing baseado em retencao de raw
- Jobs de downsampling precisam de monitoramento
