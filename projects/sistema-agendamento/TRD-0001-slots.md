# TRD-0001: Slots fixos vs horarios flexiveis

- **Status:** Concluida
- **ADR resultante:** [ADR-0001 - Lock otimista para conflitos](ADR-0001-conflito-horario.md)

## Contexto

Como representar disponibilidade de horarios?

Exemplo: profissional atende das 8h as 18h. Consulta dura 30min.

## Opcoes

### A: Slots fixos (grade de horarios)

```
08:00 - disponivel
08:30 - disponivel
09:00 - ocupado
09:30 - ocupado
...
```

- **Pros:** UI simples (seleciona slot), conflito impossivel por design
- **Contras:** Inflexivel (e se consulta dura 45min?), desperdicio de slots

### B: Horarios flexiveis (inicio + duracao)

```
Agendamento: inicio=09:15, duracao=45min
Disponibilidade: intervalo livre entre agendamentos
```

- **Pros:** Flexivel, otimiza ocupacao, duracao variavel
- **Contras:** Calculo de disponibilidade mais complexo, fragmentacao

### C: Hibrido (slots com merge)

Slots de 15min como unidade minima. Agendamento ocupa N slots consecutivos.

- **Pros:** Flexibilidade com estrutura
- **Contras:** Ainda tem limitacao de granularidade

## Analise

Caso real: clinica de estetica
- Procedimento A: 30 minutos
- Procedimento B: 1 hora
- Procedimento C: 2 horas

Slots fixos de 30min:
- Procedimento C ocupa 4 slots
- Se alguem agendou 08:30-09:00, proximo slot de 2h so as 09:00
- Fragmentacao: 08:00-08:30 fica vazio

Horarios flexiveis:
- Procedimento de 2h pode comecar 08:00
- Proximo pode comecar 10:00
- Sem fragmentacao forcada

## Decisao

**Horarios flexiveis** com granularidade minima de 15 minutos.

- Agendamento tem start_time e end_time
- Disponibilidade calculada como gaps entre agendamentos
- UI mostra horarios possiveis dado duracao do servico

Query de disponibilidade:
```sql
-- acha gaps entre agendamentos onde cabe duracao solicitada
```

## Riscos

- Query de disponibilidade mais lenta - mitigacao: indice otimizado, cache de slots disponiveis
- UI mais complexa - mitigacao: API retorna horarios sugeridos, nao gaps raw
