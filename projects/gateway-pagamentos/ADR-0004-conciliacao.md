# ADR-0004: Conciliacao assincrona com tolerancia a divergencia

- **Status:** Aceita
- **Tags:** pagamentos, conciliacao, financeiro

## Contexto

Adquirentes enviam arquivo de liquidacao diario (quais transacoes foram pagas, taxas cobradas). Precisamos bater com nossos registros.

Problemas reais:
- Arquivo chega com atraso (as vezes 3 dias)
- Formato diferente por adquirente
- Valores com centavos de diferenca (arredondamento)
- Transacoes no arquivo que nao temos (processadas por outro canal)
- Transacoes nossas que nao estao no arquivo (ainda)

## Decisao

Conciliacao assincrona com janela de tolerancia e estados intermediarios.

Estados de conciliacao:
```
PENDING   -> aguardando arquivo do adquirente
MATCHED   -> bateu valor e status
DIVERGENT -> diferenca dentro da tolerancia, aceito
DISPUTED  -> diferenca fora da tolerancia, requer analise
MANUAL    -> resolvido manualmente
```

Regras de tolerancia:
- Diferenca de ate R$ 0,05: aceita automaticamente (DIVERGENT)
- Diferenca de ate 0,1%: aceita automaticamente
- Acima: marca DISPUTED pra analise

Janela de espera: 5 dias uteis. Se transacao nao aparecer no arquivo apos 5 dias, alerta.

## Justificativa

**Conciliacao sincrona (bate na hora)**
- Arquivo chega em horario variavel
- Processamento bloqueia outras operacoes
- Timeout se arquivo grande

**Match exato ou falha**
- Qualquer centavo de diferenca vira disputa
- Volume de disputas inviabiliza analise manual
- Maioria das diferencas e arredondamento

**Tolerancia + janela temporal**
- Aceita diferencas irrelevantes automaticamente
- Tempo pra adquirente enviar arquivo
- Disputas reais sao poucas e recebem atencao

Trade-off: aceitar divergencia pequena sem investigar. Risco de perder dinheiro em escala. Monitoramento de divergencias agregadas mitiga.

## Consequencias

- 95% das transacoes conciliam automaticamente
- Analistas focam em disputas reais (< 1%)
- Dashboard de divergencias por adquirente (identifica padrao)
- Relatorio de "dinheiro perdido em arredondamento" pra visibilidade
- SLA de 5 dias pra identificar transacao perdida
