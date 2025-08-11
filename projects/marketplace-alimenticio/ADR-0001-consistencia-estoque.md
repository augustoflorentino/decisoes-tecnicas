# ADR-0001: Consistencia de estoque em alta concorrencia

- **Status:** Aceita
- **Tags:** estoque, concorrencia, redis
- **Trade-off:** [TRD-0004 - Estrategia de concorrencia em estoque](TRD-0004-estrategia-concorrencia-estoque.md)

## Contexto

Marketplace com produtos de estoque baixo (1-5 unidades por item tipicamente). Esperamos cenarios onde multiplos usuarios tentam comprar o mesmo item simultaneamente.

Risco principal: overselling. Usuario ve "1 disponivel", clica comprar, e no momento do checkout ja vendeu pra outro.

Cenario projetado: promocoes relampago com ~50 unidades e centenas de requests simultaneos.

## Decisao

Lock otimista com versioning no banco + reserva temporaria com TTL no Redis.

Fluxo:
1. Usuario adiciona ao carrinho → reserva no Redis (TTL 10min)
2. Checkout → lock otimista no Postgres com version field
3. Conflito → retry automatico ate 3x, depois falha graceful

## Justificativa

Alternativas consideradas:

**Lock pessimista (SELECT FOR UPDATE)**
- Funciona, mas serializa todas as escritas
- Latencia sobe em pico
- Descartado

**Fila unica por produto**
- Garante ordem, mas complexidade alta
- Precisa de infra adicional (particionamento por produto_id)
- Overkill pro volume inicial

**Lock otimista + reserva**
- Redis aguenta pico de leitura
- Postgres so recebe writes que provavelmente vao passar
- Retry transparente pro usuario

Trade-off aceito: usuario pode perder o item se demorar mais de 10min no checkout. Ok pro contexto - produto ta quase vencendo, urgencia faz sentido.

## Consequencias

- Overselling prevenido desde o inicio
- Checkout com latencia levemente maior (~40ms a mais)
- Redis precisa de monitoramento de memoria (reservas acumulam)
- Logica de retry no codigo - complexidade adicional no service
