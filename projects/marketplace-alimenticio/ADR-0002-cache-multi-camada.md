# ADR-0002: Cache multi-camada com invalidacao seletiva

- **Status:** Aceita
- **Tags:** cache, performance, redis

## Contexto

Catalogo projetado pra ~15k produtos ativos. Cada produto tem: info basica, preco dinamico (muda conforme proximidade do vencimento), estoque, avaliacoes.

Desafios previstos:
- Home com 50+ produtos: muitas queries por request
- Preco muda frequentemente, cache ingenuo vai mostrar preco errado
- Picos de acesso em horarios especificos

## Decisao

Cache em 3 camadas com TTLs e invalidacao diferentes:

```
L1: In-memory (Node) - 30s - dados quentes
L2: Redis - 5min - catalogo geral
L3: CDN - 1h - assets estaticos
```

Invalidacao:
- Preco/estoque: pub/sub Redis → invalida L1 e L2 imediatamente
- Info basica: TTL natural, nao invalida
- Imagens: CDN com cache-busting via hash no filename

## Justificativa

**Cache unico no Redis**
- Simples, mas todo request bate no Redis
- Latencia de rede (~2ms) se soma
- Em pico, Redis pode virar gargalo

**Cache so em memoria**
- Rapido, mas inconsistente entre instancias
- Invalidacao impossivel sem coordenacao

**Multi-camada**
- L1 absorve requests repetidos na mesma instancia
- L2 serve quando L1 expira
- Invalidacao seletiva: so invalida o que realmente muda

Trade-off: complexidade de operacao. Precisa entender onde o dado ta cached pra debugar. Documentacao do fluxo e obrigatoria.

## Consequencias

- Banco protegido de picos de leitura
- Latencia baixa pra dados quentes
- Time precisa entender o modelo mental pra debugar problemas de "dado desatualizado"
- Monitoramento de invalidacao como metrica critica
