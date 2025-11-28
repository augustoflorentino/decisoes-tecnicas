# TRD-0002: Sync com calendario externo vs unidirecional

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada na integracao)

## Contexto

Profissionais querem ver agendamentos no Google Calendar pessoal. Alguns tambem querem bloquear horarios no nosso sistema quando tem compromisso externo.

Opcoes:
1. Export unidirecional (nosso sistema -> Google)
2. Sync bidirecional (ambos os lados)

## Opcoes

### A: Sem integracao

- **Pros:** Simples, sem dependencia externa
- **Contras:** Profissional precisa checar dois lugares, risco de conflito

### B: Export unidirecional (push pro Google)

- **Pros:** Visualizacao unificada, implementacao simples
- **Contras:** Compromisso pessoal nao bloqueia no sistema, conflito possivel

### C: Import unidirecional (pull do Google)

- **Pros:** Eventos externos bloqueiam disponibilidade
- **Contras:** Nossos agendamentos nao aparecem no Google

### D: Sync bidirecional

- **Pros:** Visao completa em ambos, disponibilidade sempre atualizada
- **Contras:** Conflitos de edicao, loop de sync, complexidade alta

## Analise

Sync bidirecional e notoriamente dificil:
- Evento editado dos dois lados ao mesmo tempo
- Qual e a fonte de verdade?
- Latencia de sync causa inconsistencia temporaria
- Google Calendar nao foi feito pra ser banco de dados

Alternativa pragmatica:
- Export pro Google (nossos agendamentos aparecem la)
- Import como "bloqueio" (eventos externos viram indisponibilidade, nao agendamento)

Nao e sync verdadeiro, mas resolve o problema real.

## Decisao

**Export + Import assimetrico**

Export (nosso -> Google):
- Agendamento criado/atualizado/cancelado -> evento no Google
- Profissional ve tudo em um lugar

Import (Google -> nosso):
- Eventos do Google marcam horario como indisponivel
- Nao cria agendamento, apenas bloqueia slot
- Sync a cada 15 minutos ou via webhook

Fonte de verdade: nosso sistema. Google e visualizacao + bloqueio.

## Riscos

- Evento externo nao sincronizado a tempo - mitigacao: profissional pode bloquear manualmente
- Muitos eventos externos - mitigacao: importa apenas calendario selecionado
