# ADR-0001: Lock otimista para conflitos de horario

- **Status:** Aceita
- **Tags:** agendamento, concorrencia, conflito
- **Trade-off:** [TRD-0001 - Slots fixos vs horarios flexiveis](TRD-0001-slots.md)

## Contexto

Dois clientes tentam agendar mesmo horario com mesmo profissional. Race condition classica.

Cenario:
- Cliente A ve horario 14h disponivel
- Cliente B ve horario 14h disponivel
- Cliente A confirma 14h
- Cliente B confirma 14h
- Conflito: dois agendamentos no mesmo horario

## Decisao

Lock otimista com verificacao de sobreposicao no momento do INSERT.

```sql
INSERT INTO appointments (professional_id, start_time, end_time, ...)
SELECT $1, $2, $3, ...
WHERE NOT EXISTS (
  SELECT 1 FROM appointments
  WHERE professional_id = $1
  AND status != 'cancelled'
  AND (start_time, end_time) OVERLAPS ($2, $3)
)
RETURNING id
```

Se retornar 0 linhas, conflito detectado. Retorna erro pro cliente com horarios alternativos.

Indice parcial pra performance:
```sql
CREATE INDEX idx_appointments_overlap
ON appointments (professional_id, start_time, end_time)
WHERE status != 'cancelled'
```

## Justificativa

**Lock pessimista (SELECT FOR UPDATE)**
- Funciona, mas serializa todas as reservas do profissional
- Latencia alta em horario de pico

**Lock no Redis antes de tentar**
- Rapido, mas adiciona ponto de falha
- Inconsistencia se Redis e Postgres divergirem

**Lock otimista com constraint**
- Banco garante consistencia
- Nao serializa requests
- Retry e barato (cliente escolhe outro horario)

## Consequencias

- Zero agendamentos duplicados
- UX: cliente recebe erro claro com alternativas
- Query de disponibilidade exclui horarios com conflito potencial
- Cancelamento libera slot imediatamente
