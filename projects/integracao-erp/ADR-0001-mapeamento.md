# ADR-0001: Mapeamento de campos com transformacao bidirecional

- **Status:** Aceita
- **Tags:** integracao, mapeamento, transformacao

## Contexto

Nosso sistema e ERP usam estruturas diferentes pro mesmo conceito.

Exemplo - produto:
```
Nosso sistema:          ERP (SAP):
{                       {
  "sku": "ABC-123",       "MATNR": "000000000ABC123",
  "name": "Camiseta",     "MAKTX": "CAMISETA TAM M",
  "price": 99.90,         "NETPR": 9990,
  "active": true          "LVORM": ""
}                       }
```

Problemas:
- Nomes de campos diferentes
- Formatos diferentes (preco em centavos vs reais)
- Semantica diferente (active vs deletion flag invertido)

## Decisao

Mapeamento declarativo com transformadores bidirecionais.

```typescript
const productMapping = {
  fields: [
    {
      source: 'sku',
      target: 'MATNR',
      transform: {
        toErp: (v) => v.replace('-', '').padStart(18, '0'),
        fromErp: (v) => v.replace(/^0+/, '').replace(/(.{3})/, '$1-')
      }
    },
    {
      source: 'price',
      target: 'NETPR',
      transform: {
        toErp: (v) => Math.round(v * 100),
        fromErp: (v) => v / 100
      }
    },
    {
      source: 'active',
      target: 'LVORM',
      transform: {
        toErp: (v) => v ? '' : 'X',
        fromErp: (v) => v !== 'X'
      }
    }
  ]
}
```

Engine aplica mapeamento nas duas direcoes.

## Justificativa

**Codigo de transformacao manual por entidade**
- Flexivel
- Mas duplicacao, dificil manter
- Mudanca de mapeamento requer deploy

**Mapeamento em banco (configuracao)**
- Sem deploy pra mudancas
- Mas transformacoes complexas nao cabem em config
- Debugging dificil

**Mapeamento declarativo em codigo**
- Transformacoes tipadas, testáveis
- Mapeamento e documentacao
- Reutiliza transformadores comuns

## Consequencias

- Novo ERP: cria mapeamento, reutiliza engine
- Mudanca de formato: altera transformador, testa
- Debug: log mostra valor antes/depois da transformacao
- Validacao: schema do ERP valida output
