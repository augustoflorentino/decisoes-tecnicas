# ADR-0002: Template engine com fallback por canal

- **Status:** Aceita
- **Tags:** templates, notificacoes, conteudo

## Contexto

Mesma notificacao vai por multiplos canais. Conteudo precisa adaptar:
- Email: HTML rico, imagens, botoes
- SMS: 160 caracteres, texto puro
- Push: titulo curto + corpo de 2 linhas
- WhatsApp: markdown simples, sem HTML

Manter N versoes de cada notificacao e trabalhoso.

## Decisao

Template base unico com transformadores por canal.

```
Template base (Handlebars):
"Ola {{nome}}, seu pedido #{{pedido_id}} foi {{status}}. {{#if link}}Acompanhe: {{link}}{{/if}}"

Transformadores:
- email: renderiza HTML, adiciona header/footer, imagens
- sms: trunca em 160 chars, remove links longos
- push: extrai titulo (primeira frase), corpo (resto)
- whatsapp: converte pra markdown, adiciona emojis config
```

Fallback: se template especifico do canal existe, usa. Senao, transforma o base.

## Justificativa

**Template por canal (N versoes)**
- Controle total
- Mas manutencao multiplicada por 4
- Facil dessincronizar conteudo

**Template unico sem transformacao**
- Simples
- Mas SMS recebe HTML, push fica truncado errado

**Template base + transformadores**
- Um lugar pra manter conteudo
- Cada canal recebe formato apropriado
- Override quando necessario (campanha especifica)

## Consequencias

- Novo tipo de notificacao = 1 template, funciona em 4 canais
- Transformadores sao genericos, reutilizaveis
- Preview por canal no admin antes de enviar
- QA testa uma vez, confia nos transformadores
