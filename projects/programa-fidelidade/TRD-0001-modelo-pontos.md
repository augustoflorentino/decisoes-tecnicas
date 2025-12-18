# TRD-0001: Pontos como moeda vs pontos como score

- **Status:** Concluida
- **ADR resultante:** [ADR-0001 - Ledger de pontos com double-entry](ADR-0001-ledger.md)

## Contexto

O que sao "pontos" no sistema?

Duas visoes:
1. Moeda virtual (acumula, gasta, tem saldo)
2. Score/XP (so sobe, determina nivel)

## Opcoes

### A: Pontos como score (so acumula)

```
Cliente ganha 100 pontos
Saldo: 100
Cliente "resgata" premio de 50 pontos
Saldo: 100 (nao muda, so desbloqueia premio)
```

- **Pros:** Simples, gamificacao pura, cliente nunca "perde"
- **Contras:** Nao funciona como moeda, resgate ilimitado uma vez destravado

### B: Pontos como moeda (acumula e gasta)

```
Cliente ganha 100 pontos
Saldo: 100
Cliente resgata premio de 50 pontos
Saldo: 50
```

- **Pros:** Modelo economico real, incentiva acumulo, resgate tem custo
- **Contras:** Cliente "perde" pontos ao resgatar, precisa de contabilidade

### C: Dois tipos (XP + coins)

```
Compra gera:
- 100 XP (determina nivel, nao gasta)
- 100 coins (resgatavel)
```

- **Pros:** Nivel permanente, moeda pra resgates
- **Contras:** Complexidade, usuario confuso

## Analise

Objetivo do programa:
- Incentivar compras recorrentes
- Premiar fidelidade
- Custo controlado de premios

Pontos como score:
- Usuario resgata tudo de uma vez, some
- Sem incentivo pra continuar depois de atingir nivel

Pontos como moeda:
- Precisa continuar comprando pra manter saldo
- Cada resgate incentiva novo acumulo
- Custo de premios e previsivel (X pontos = Y reais)

## Decisao

**Pontos como moeda** com conversao definida.

- 1 ponto = R$ 0,01 (valor contabil)
- Compra de R$ 100 = 100 pontos
- Resgate de premio de R$ 50 = 5000 pontos
- Nivel baseado em pontos acumulados (nao saldo atual)

Nivel considera total historico, saldo considera disponivel. Cliente pode subir de nivel mas ter saldo zero (resgatou tudo).

## Riscos

- Cliente frustrado ao "perder" pontos - mitigacao: comunicacao clara, valor tangivel dos premios
- Contabilidade complexa - mitigacao: ledger double-entry, auditoria automatica
