# TRD-0003: Estrategia de fallback entre adquirentes

- **Status:** Concluida
- **ADR resultante:** [ADR-0002 - Roteamento dinamico de adquirente](ADR-0002-roteamento-adquirente.md)

## Contexto

Adquirente principal do merchant esta fora. O que fazer?

Opcoes:
1. Falhar a transacao
2. Tentar outro adquirente automaticamente
3. Perguntar pro merchant em tempo real

Cada uma tem implicacoes de UX, custo e risco.

## Opcoes

### A: Fail fast (retorna erro)

- **Pros:** Simples, merchant decide o que fazer, sem risco de taxa diferente
- **Contras:** Transacao perdida, cliente vai embora, merchant perde venda

### B: Fallback automatico silencioso

- **Pros:** Transacao passa, cliente feliz, merchant nao perde venda
- **Contras:** Taxa pode ser maior no adquirente secundario, merchant surpreso na fatura

### C: Fallback com notificacao

- **Pros:** Transacao passa, merchant sabe que foi fallback, pode contestar se quiser
- **Contras:** Contestacao pos-fato e inutil (transacao ja foi)

### D: Fallback com limite de diferenca

- **Pros:** So faz fallback se taxa do secundario estiver dentro de margem aceitavel
- **Contras:** Precisa saber taxa em tempo real (nem sempre disponivel)

## Analise

Dados de 3 meses:
- 2.3% das transacoes falharam no adquirente primario
- Dessas, 89% passariam em outro adquirente
- Diferenca media de taxa: 0.15%
- Ticket medio: R$ 180

Conta:
- Transacao perdida = R$ 180 de GMV perdido
- Fallback com taxa maior = R$ 0,27 a mais de custo (0.15% de R$ 180)

Merchant prefere pagar R$ 0,27 a perder R$ 180.

## Decisao

**Opcao C: Fallback automatico com notificacao**

- Fallback habilitado por padrao
- Merchant pode desabilitar se quiser
- Webhook de `transaction.fallback` notifica quando acontece
- Relatorio mensal mostra quantas transacoes foram salvas por fallback

Limite opcional: merchant pode configurar "so fallback se taxa < X%".

## Riscos

- Merchant reclama de taxa maior - mitigacao: opt-out disponivel, transparencia total
- Adquirente secundario tambem fora - mitigacao: permite ate 2 tentativas de fallback
- Fallback pra adquirente com taxa muito maior - mitigacao: configuracao de limite por merchant
