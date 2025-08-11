# TRD-0004: Estrategia de concorrencia em estoque

- **Status:** Concluida
- **ADR resultante:** [ADR-0001 - Consistencia de estoque em alta concorrencia](ADR-0001-consistencia-estoque.md)

## Contexto

Flash sale: 50 unidades de produto, 200 usuarios tentando comprar ao mesmo tempo. Race condition classica.

Cenario real que vai acontecer:
- Usuario A le estoque = 1
- Usuario B le estoque = 1
- Usuario A decrementa pra 0, salva
- Usuario B decrementa pra 0, salva (sobrescreveu)
- Vendemos 2, tinhamos 1. Alguem vai ficar sem produto.

Restricoes:
- Postgres como banco principal (nao tem DynamoDB ou similar)
- Precisa funcionar em burst de requests
- Experiencia do usuario nao pode degradar muito

## Opcoes

### A: Lock pessimista (SELECT FOR UPDATE)

```sql
BEGIN;
SELECT * FROM products WHERE id = ? FOR UPDATE;
-- processa
UPDATE products SET stock = stock - 1;
COMMIT;
```

- **Pros:** Garantia absoluta, simples de entender, funciona em qualquer cenario
- **Contras:** Serializa requests pro mesmo produto, throughput despenca, deadlock possivel se logica complexa

### B: Lock otimista com versao

```sql
UPDATE products
SET stock = stock - 1, version = version + 1
WHERE id = ? AND version = ? AND stock > 0;
-- checa rows affected
```

- **Pros:** Sem lock no banco, retry e barato, escala melhor
- **Contras:** Retry storm em alta concorrencia, logica de retry na aplicacao, pode frustrar usuario se retry falhar

### C: Fila de reservas (Redis + processamento sequencial)

- **Pros:** Ordem garantida, sem race condition, pode confirmar reserva async
- **Contras:** Latencia adicional, complexidade de infra, usuario espera confirmacao

### D: CQRS com evento de reserva

- **Pros:** Escalabilidade extrema, audit trail nativo, desacopla leitura de escrita
- **Contras:** Consistencia eventual, complexidade absurda pra time pequeno, debugging dificil

## Analise

Testamos A e B em carga simulada (100 requests simultaneos pro mesmo produto):

**Lock pessimista:**
- p50: 45ms
- p99: 890ms
- Sem erro, mas lento demais no pico

**Lock otimista:**
- p50: 12ms
- p99: 180ms (com retry)
- ~15% precisou de 1 retry, ~2% precisou de 2

Opcao C adiciona ~50ms de latencia base. Pra flash sale, usuario quer resposta instantanea.

Opcao D e overengineering. Time de 3 pessoas nao vai manter event sourcing.

## Decisao

**Opcao B: Lock otimista com versao**, com ajustes:

1. Retry com backoff exponencial (max 3 tentativas)
2. Se todas falharem, retorna "produto esgotado" (melhor que vender o que nao tem)
3. Monitoramento de taxa de retry pra detectar problema

Trade-off aceito: em concorrencia extrema, alguns usuarios vao ver "esgotado" mesmo tendo estoque. Preferivel a oversell.

## Riscos identificados

- Retry storm degradar banco - mitigacao: circuit breaker, rate limit por usuario
- Usuario frustrado com "esgotado" falso - mitigacao: mensagem clara, botao de "avisar quando voltar"
