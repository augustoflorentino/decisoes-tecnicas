# Sistema Agendamento

Plataforma de agendamento para clinicas e prestadores de servico. Reservas, conflitos, recorrencia, lembretes.

## Stack

- NestJS
- PostgreSQL
- Redis (locks)
- Bull (lembretes)
- Google Calendar API (sync)

## ADRs

- [ADR-0001 - Lock otimista para conflitos de horario](ADR-0001-conflito-horario.md)
- [ADR-0002 - Timezone por estabelecimento com normalizacao UTC](ADR-0002-timezone.md)
- [ADR-0003 - Recorrencia como template com instancias materializadas](ADR-0003-recorrencia.md)

## Trade-offs

- [TRD-0001 - Slots fixos vs horarios flexiveis](TRD-0001-slots.md) → ADR-0001
- [TRD-0002 - Sync com calendario externo vs unidirecional](TRD-0002-calendar-sync.md)
