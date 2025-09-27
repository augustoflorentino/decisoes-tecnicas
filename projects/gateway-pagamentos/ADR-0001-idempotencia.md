# ADR-0001: Idempotencia com chave composta e janela temporal

- **Status:** Aceita
- **Tags:** pagamentos, idempotencia, consistencia

## Contexto

Cliente envia requisicao de pagamento, timeout na resposta, reenvia. Se nao tratado:
- Cobra duas vezes
- Cliente reclama
- Chargeback
- Prejuizo

Idempotencia e obrigatorio. Mas como implementar sem criar problemas novos?

## Decisao

Chave de idempotencia composta: `merchant_id + idempotency_key + operacao`

Armazenamento em Redis com TTL de 24 horas.

```
Chave: idempotency:{merchant_id}:{idempotency_key}:{operation}
Valor: {status, response, created_at}
TTL: 86400s (24h)
```

Fluxo:
1. Request chega com header `Idempotency-Key`
2. Monta chave composta
3. GET no Redis
4. Se existe e status=completed, retorna response salva
5. Se existe e status=processing, retorna 409 Conflict
6. Se nao existe, SET com status=processing, processa, atualiza pra completed

## Justificativa

**Idempotency-Key simples (so o header)**
- Cliente pode reusar key entre merchants diferentes
- Colisao de chaves
- Falha de seguranca (um merchant ve resposta de outro)

**Hash do payload como chave**
- Qualquer mudanca no payload gera chave nova
- Retry com payload identico funciona
- Mas cliente que muda um campo irrelevante (metadata) perde idempotencia

**Chave composta com escopo**
- Isolamento por merchant
- Cliente controla a chave
- Operacao na chave evita reusar key de authorize pra capture

Trade-off: Redis como dependencia critica. Se Redis cair, sistema recusa requests ate voltar (fail closed).

## Consequencias

- Zero cobrancas duplicadas desde implementacao
- Merchants podem implementar retry agressivo sem medo
- Monitoramento de hit rate de idempotencia (detecta clientes com problema)
- TTL de 24h cobre retry humano (usuario tenta de novo no dia seguinte)
