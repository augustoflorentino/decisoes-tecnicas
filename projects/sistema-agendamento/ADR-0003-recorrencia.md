# ADR-0003: Recorrencia como template com instancias materializadas

- **Status:** Aceita
- **Tags:** recorrencia, agendamento, modelo

## Contexto

Paciente faz fisioterapia toda terca 14h por 3 meses. Como modelar?

Opcoes:
- Criar 12 agendamentos individuais
- Criar regra de recorrencia e expandir em runtime

Problemas:
- Paciente quer cancelar so uma sessao
- Profissional quer mudar horario a partir de certa data
- Paciente quer ver historico de sessoes passadas

## Decisao

Template de recorrencia + instancias materializadas.

```sql
recurrence_templates (
  id,
  professional_id,
  patient_id,
  rrule TEXT,  -- 'FREQ=WEEKLY;BYDAY=TU;COUNT=12'
  start_time TIME,
  duration INTERVAL,
  created_at
)

appointments (
  id,
  recurrence_template_id,  -- nullable, link pro template
  start_time TIMESTAMPTZ,
  status,
  ...
)
```

Fluxo:
1. Cria template de recorrencia
2. Job expande proximas N semanas em appointments reais
3. Cada appointment pode ser editado/cancelado individualmente
4. Edicao no template afeta apenas instancias futuras nao-modificadas

## Justificativa

**Apenas regra (expand em runtime)**
- Storage minimo
- Mas query de disponibilidade precisa expandir toda vez
- Modificacao individual impossivel
- Performance ruim

**Apenas instancias (sem template)**
- Simples
- Mas "mudar todas as futuras" requer update em N registros
- Sem visao de "qual a regra original"

**Template + instancias materializadas**
- Query de disponibilidade e simples (appointments reais)
- Modificacao individual funciona
- Template preserva intencao original
- "Alterar futuras" e bulk update nos nao-modificados

## Consequencias

- Cancelar uma sessao: update no appointment especifico
- Mudar horario de todas: update no template + regenerar futuras
- Historico preservado (appointments passados nao mudam)
- Job diario materializa proximas 8 semanas
