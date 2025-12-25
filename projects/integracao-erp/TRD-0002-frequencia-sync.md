# TRD-0002: Sync real-time vs batch periodico

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada na configuracao de sync)

## Contexto

Com que frequencia sincronizar dados entre sistemas?

Entidades:
- Produtos: mudam pouco (~10 alteracoes/dia)
- Estoque: muda muito (a cada venda)
- Pedidos: criados constantemente
- Financeiro: precisa bater no fechamento

## Opcoes

### A: Real-time (webhook/evento)

- **Pros:** Dados sempre atualizados, latencia minima
- **Contras:** Carga constante, ERP pode nao suportar webhook, complexidade

### B: Batch periodico (a cada X minutos)

- **Pros:** Previsivel, menos carga, funciona com qualquer ERP
- **Contras:** Dados defasados entre syncs, pode perder alteracao se falhar

### C: Hibrido (real-time pra critico, batch pra resto)

- **Pros:** Otimiza por caso de uso
- **Contras:** Dois mecanismos pra manter

### D: Change Data Capture (CDC)

- **Pros:** Captura tudo, sem polling, baixa latencia
- **Contras:** Precisa de acesso ao banco do ERP, invasivo, nem todo ERP permite

## Analise

Requisitos reais:
- Estoque: desatualizacao de 5min causa overselling
- Produto: desatualizacao de 1 hora e aceitavel
- Pedido: deve ir pro ERP em < 1 minuto
- Financeiro: batch diario e suficiente (fechamento)

ERPs:
- SAP: suporta webhook (IDocs)
- TOTVS: API REST com polling
- Sankhya: arquivo batch

Nao da pra ter estrategia unica.

## Decisao

**Configuracao por entidade e por ERP**

```
{
  "sap": {
    "products": {"mode": "batch", "interval": "1h"},
    "inventory": {"mode": "realtime", "webhook": true},
    "orders": {"mode": "realtime", "webhook": true},
    "financial": {"mode": "batch", "interval": "1d"}
  },
  "totvs": {
    "products": {"mode": "batch", "interval": "1h"},
    "inventory": {"mode": "batch", "interval": "5m"},
    "orders": {"mode": "batch", "interval": "1m"},
    "financial": {"mode": "batch", "interval": "1d"}
  }
}
```

Real-time quando ERP suporta e entidade e critica. Batch pro resto.

## Riscos

- Configuracao errada causa dessincronizacao - mitigacao: defaults sensiveis, validacao de config
- Polling frequente sobrecarrega ERP - mitigacao: respeitar rate limits, delta sync
