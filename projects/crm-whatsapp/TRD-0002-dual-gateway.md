# TRD-0002: Dual gateway externo e interno

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada na infra inicial)

## Contexto

Microservicos precisam de:
1. Ponto de entrada pro mundo externo (frontend, webhooks)
2. Comunicacao interna entre servicos

Uma solucao ou duas?

## Opcoes

### A: Gateway unico pra tudo (Kong/Nginx)

```
Internet -> Kong -> [BFF, Webhook, Core, Chatbot, ...]
Servicos internos -> Kong -> [outros servicos]
```

- **Pros:** Uma ferramenta, configuracao centralizada
- **Contras:** Kong/Enterprise e caro, Nginx nao tem service discovery nativo

### B: Mesh completo (Istio/Linkerd)

```
Tudo via sidecar proxy
```

- **Pros:** Observabilidade total, mTLS automatico, traffic management
- **Contras:** Complexidade absurda, overhead de recursos, overkill pro volume

### C: Nginx externo + Traefik interno

```
Internet -> Nginx -> [BFF, Webhook]
Servicos internos -> Traefik -> [outros servicos]
```

- **Pros:** Cada um otimizado pra seu caso, Traefik tem service discovery via labels
- **Contras:** Duas ferramentas, duas configs

## Analise

**Nginx** e imbativel pra:
- SSL termination
- Rate limiting de borda
- Configuracao estatica (endpoints conhecidos)
- Performance com recursos minimos

**Traefik** e melhor pra:
- Service discovery dinamico (containers sobem e descem)
- Health check automatico
- Dashboard de debug
- Configuracao via labels no docker-compose

Combinar os dois usa o melhor de cada.

## Decisao

**Opcao C: Nginx externo + Traefik interno**

```
Internet -> Nginx (SSL, rate limit) -> BFF/Webhook
BFF -> Traefik -> [Core, Chatbot, CRM, ...]
Servicos -> Traefik -> [outros servicos]
```

Traefik descobre servicos automaticamente. Escalar um servico nao precisa mudar config.

## Riscos

- Duas ferramentas pra aprender - mitigacao: Nginx config e simples, Traefik e declarativo
- Traefik pode ser ponto unico de falha interno - mitigacao: health check, replica em standby
