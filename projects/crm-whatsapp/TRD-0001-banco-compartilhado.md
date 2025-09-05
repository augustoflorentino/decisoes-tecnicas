# TRD-0001: Microservicos com banco compartilhado

- **Status:** Concluida
- **ADR resultante:** [ADR-0001 - Separacao de bancos por criticidade](ADR-0001-separacao-bancos.md)

## Contexto

Arquitetura de microservicos, mas qual estrategia de banco?

Restricoes:
- 6 servicos (Core, Chatbot, Disparador, CRM, Webhook, Sender)
- Alguns servicos precisam de dados de outros (CRM precisa de contatos do Core)

## Opcoes

### A: Banco por servico (isolamento total)

```
Core     -> postgres_core
Chatbot  -> postgres_chatbot
CRM      -> postgres_crm
...
```

- **Pros:** Isolamento completo, escala independente, schema proprio
- **Contras:** Joins via API (lento), transacoes distribuidas, 6 instancias pra manter

### B: Banco unico compartilhado

```
Todos -> postgres_principal
```

- **Pros:** Joins nativos, transacao ACID, uma instancia
- **Contras:** Schema vira bagunca, servicos acoplados no banco, migracao coordenada

### C: Banco compartilhado por dominio

```
Core + CRM + Disparador -> postgres_principal (dados de negocio)
Chatbot + Webhook + Sender -> postgres_mensagens (dados de conversa)
```

- **Pros:** Agrupa por caracteristica (criticidade, volume), menos instancias
- **Contras:** Ainda tem acoplamento dentro do grupo, migracao coordenada por grupo

## Analise

| Criterio | A (por servico) | B (unico) | C (por dominio) |
|----------|-----------------|-----------|-----------------|
| Complexidade de infra | Alta | Baixa | Media |
| Transacoes | Sagas | ACID | ACID por grupo |
| Performance de join | Ruim | Otima | Boa |
| Isolamento de falha | Total | Nenhum | Parcial |
| Custo | Alto | Baixo | Medio |

Operar 6 bancos e overhead desnecessario. Banco unico acopla demais. Opcao C equilibra.

## Decisao

**Opcao C: Banco compartilhado por dominio**

Dois bancos:
1. Principal: dados criticos de negocio
2. Mensagens: alto volume, recuperavel

Servicos do mesmo grupo podem fazer join direto. Entre grupos, via API.

## Riscos

- Schema do grupo ficar bagunçado - mitigacao: prefixo de tabela por servico, review de migration
- Servico lento afetar outros do grupo - mitigacao: connection pool separado, timeout agressivo
