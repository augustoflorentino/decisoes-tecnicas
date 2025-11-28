# ADR-0002: Timezone por estabelecimento com normalizacao UTC

- **Status:** Aceita
- **Tags:** timezone, data, internacionalizacao

## Contexto

Sistema atende estabelecimentos em diferentes fusos:
- Clinica em Sao Paulo (UTC-3)
- Clinica em Manaus (UTC-4)
- Cliente viajando pode agendar de outro fuso

Problemas comuns:
- Agendamento as 14h de SP mostrado como 13h pra cliente em Manaus
- Horario de verao muda offset
- Recorrencia semanal cruza mudanca de horario de verao

## Decisao

Armazenamento em UTC. Timezone do estabelecimento salvo separado. Conversao na exibicao.

```sql
appointments (
  start_time TIMESTAMPTZ,  -- sempre UTC
  end_time TIMESTAMPTZ,
  ...
)

establishments (
  timezone TEXT  -- 'America/Sao_Paulo'
)
```

Regras:
- API recebe horario local + timezone (ou assume do estabelecimento)
- Armazena convertido pra UTC
- Retorna UTC + timezone do estabelecimento
- Frontend converte pra exibicao

Recorrencia usa timezone do estabelecimento pra "14h toda terca" significar 14h local, independente de horario de verao.

## Justificativa

**Armazenar em timezone local**
- Intuitivo
- Mas comparacao entre fusos e impossivel
- Horario de verao quebra queries

**Armazenar offset junto (14h, -03:00)**
- Melhor que local puro
- Mas offset muda com horario de verao
- Historico fica confuso

**UTC + timezone nomeado**
- Comparacao trivial (tudo em UTC)
- Biblioteca de timezone resolve horario de verao
- "America/Sao_Paulo" sabe quando tem horario de verao

## Consequencias

- Queries de disponibilidade funcionam independente do fuso do cliente
- Relatorios agregam corretamente entre estabelecimentos
- Mudanca de horario de verao tratada automaticamente
- Frontend precisa de lib de timezone (date-fns-tz)
