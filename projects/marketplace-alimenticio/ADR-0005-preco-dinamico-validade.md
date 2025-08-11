# ADR-0005: Calculo de preco dinamico baseado em validade

- **Status:** Aceita
- **Tags:** negocio, pricing, algoritmo

## Contexto

Produto perto de vencer precisa de desconto agressivo pra sair. Vendedor nao quer ficar ajustando preco manualmente todo dia.

Abordagem simples (tipo "30% off se falta < 7 dias") trata produtos com 7 dias e com 1 dia igual. Nao faz sentido.

## Decisao

Curva de desconto exponencial configuravel por categoria.

```
desconto = base + (max - base) * (1 - dias_restantes/dias_limite)^fator

Exemplo (categoria lacteos):
- dias_limite: 14
- base: 10%
- max: 70%
- fator: 2 (curva mais agressiva no final)

Resultado:
- 14 dias: 10%
- 7 dias: 25%
- 3 dias: 48%
- 1 dia: 65%
- 0 dias (vence hoje): 70%
```

Vendedor pode sobrescrever com preco fixo se quiser. Preco dinamico e sugestao, nao obrigatorio.

## Justificativa

**Desconto linear**
- Facil de entender
- Mas incentivo pra comprar cedo e fraco
- Produto encalha ate o final

**Faixas fixas (< 7 dias = 30%, < 3 dias = 50%)**
- Saltos bruscos de preco
- Usuario espera cair pra proxima faixa
- Gaming do sistema

**Curva exponencial**
- Desconto cresce gradualmente
- Urgencia aumenta conforme aproxima
- Sem saltos, sem gaming
- Configuravel por categoria (pereciveis vs nao-pereciveis)

Trade-off: mais dificil de explicar pro vendedor. Dashboard mostra grafico da curva pra ajudar.

## Consequencias

- Vendedor configura uma vez e esquece
- Menos produtos vencem sem vender
- Suporte recebe menos ticket de "meu preco ta errado"
- Job noturno recalcula precos (~3min pra 15k produtos)
