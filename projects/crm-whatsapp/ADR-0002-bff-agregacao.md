# ADR-0002: BFF para agregacao de servicos

- **Status:** Aceita
- **Tags:** arquitetura, frontend, api

## Contexto

Frontend Angular precisa de dados de varios servicos pra montar uma tela. Exemplo: dashboard mostra metricas de chatbot + campanhas + atendimentos.

Opcoes:
1. Frontend faz N requests paralelos
2. GraphQL federation
3. BFF (Backend for Frontend) que agrega

## Decisao

BFF dedicado em NestJS entre frontend e microservicos.

```
Angular -> Nginx -> BFF -> [Core, Chatbot, CRM, Disparador]
```

BFF e responsavel por:
- Agregar dados de multiplos servicos em uma resposta
- Transformar estrutura pro que o Angular espera
- Validar JWT e propagar contexto de usuario
- Cache de agregacoes frequentes

## Justificativa

**Frontend fazendo N requests**
- Latencia acumulada (waterfall se tiver dependencias)
- Logica de agregacao duplicada no frontend
- Mais dificil cachear

**GraphQL federation**
- Complexidade de setup e operacao
- Overhead pra volume atual

**BFF**
- Uma chamada do frontend, resposta pronta
- Logica de apresentacao fica no backend
- Cache trivial (Redis)
- Mesma stack do resto (NestJS)

Trade-off: mais um servico pra manter. Aceitavel porque simplifica muito o frontend e centraliza transformacoes.

## Consequencias

- Frontend faz menos requests (melhor UX em mobile)
- Mudancas de estrutura de resposta nao quebram frontend imediatamente
- BFF pode virar gargalo se mal dimensionado - monitorar latencia
- Servicos internos podem ter API mais "crua", BFF formata
