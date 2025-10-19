# TRD-0001: Kafka vs ingestao direta no ClickHouse

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada na infra inicial)

## Contexto

API de ingestao recebe eventos dos clientes. Precisa persistir no ClickHouse.

Volume: 100M eventos/dia = ~1.2k eventos/segundo media, picos de 10k/s.

Opcoes:
1. INSERT direto no ClickHouse
2. Buffer em Kafka, consumer faz batch insert

## Opcoes

### A: Ingestao direta

```
SDK -> API -> ClickHouse (INSERT)
```

- **Pros:** Simples, menos infra, latencia menor ate persistir
- **Contras:** ClickHouse nao gosta de muitos INSERTs pequenos, picos podem derrubar

### B: Buffer com Kafka

```
SDK -> API -> Kafka -> Consumer -> ClickHouse (batch INSERT)
```

- **Pros:** Absorve picos, batch insert e eficiente, replay se consumer falhar
- **Contras:** Mais infra, latencia ate aparecer no dashboard, complexidade

### C: Buffer em memoria na API

```
SDK -> API (buffer) -> ClickHouse (batch INSERT periodico)
```

- **Pros:** Simples, batch insert eficiente
- **Contras:** Perde dados se API crashar, nao escala horizontal bem

## Analise

ClickHouse performa melhor com INSERTs grandes e infrequentes:
- 1000 INSERTs de 1 linha: lento, cria muitas parts
- 1 INSERT de 1000 linhas: rapido, uma part

INSERT direto sob pico de 10k/s = 10k INSERTs/s = ClickHouse sofre.

Kafka permite:
- API retorna 200 rapido (evento no Kafka)
- Consumer acumula por 1 segundo, faz INSERT de 10k linhas
- Se consumer cair, eventos nao perdidos

Buffer em memoria e fragil. Perder eventos de analytics e inaceitavel.

## Decisao

**Kafka como buffer** entre API e ClickHouse.

Configuracao:
- Topic particionado por tenant_id (ordenacao por cliente)
- Consumer faz batch de 5000 eventos ou 1 segundo (o que vier primeiro)
- Retention de 24h no Kafka (replay de problemas recentes)

Latencia aceita: evento demora ate 2 segundos pra aparecer no dashboard. Ok pra analytics.

## Riscos

- Kafka como ponto de falha - mitigacao: cluster com replicacao, monitoramento
- Lag do consumer crescer - mitigacao: alerta, autoescala de consumers
- Ordem de eventos por usuario - mitigacao: particionamento por tenant garante ordem por cliente
