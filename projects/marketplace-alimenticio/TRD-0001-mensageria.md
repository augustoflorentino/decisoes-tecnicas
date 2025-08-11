# TRD-0001: Escolha de sistema de mensageria

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada na stack inicial)

## Contexto

Precisa de fila pra operacoes async: notificacoes, processamento de imagem, emails. Volume estimado: ~10k jobs/dia no inicio, podendo chegar a 100k.

Restricoes:
- Time pequeno, pouca experiencia com infra complexa
- Budget limitado pra servicos gerenciados
- Redis ja vai existir pro cache

## Opcoes

### A: Bull + Redis
- **Pros:** Ja temos Redis, zero custo adicional, API simples, dashboard incluso
- **Contras:** Redis nao foi feito pra ser message broker, limites de escala

### B: RabbitMQ
- **Pros:** Feito pra isso, robusto, dead letter queues nativas, routing flexivel
- **Contras:** Mais uma peca de infra, curva de aprendizado, custo de operacao

### C: AWS SQS
- **Pros:** Gerenciado, escala infinita, integracao com Lambda
- **Contras:** Vendor lock-in, custo por request escala mal, latencia maior

## Decisao

**Opcao A: Bull + Redis**

100k jobs/dia ainda ta dentro do que Redis aguenta. Complexidade baixa, time consegue operar. Se precisar migrar pra RabbitMQ no futuro, a abstracao do Bull ajuda.

## Riscos

- Redis como ponto unico de falha (cache + fila) - mitigacao: monitoramento agressivo, replica em standby
- Se escalar alem do previsto, vai precisar migrar - mitigacao: manter interface abstrata
