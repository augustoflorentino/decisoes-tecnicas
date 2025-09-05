# ADR-0001: Separacao de bancos por criticidade de dados

- **Status:** Aceita
- **Tags:** banco, infraestrutura, backup
- **Trade-off:** [TRD-0001 - Microservicos com banco compartilhado](TRD-0001-banco-compartilhado.md)

## Contexto

Sistema vai armazenar dois tipos de dados com caracteristicas bem diferentes:

1. **Dados core:** usuarios, configuracoes, campanhas, instancias WhatsApp - criticos, nao recuperaveis
2. **Dados de mensagens:** historico de conversas, logs - volume alto, mas recuperaveis via sync com Evolution API

Tratar os dois igual significa backup caro e desnecessario pra dados que podem ser resincronizados.

## Decisao

Dois bancos PostgreSQL separados fisicamente.

**Banco principal:**
- Usuarios, permissoes, configuracoes, campanhas, contatos
- Backup diario completo + WAL archiving
- RTO: 1 hora

**Banco de mensagens:**
- Historico de conversas, contexto de chatbot, logs
- Sem backup (dados recuperaveis via Evolution API)
- Se perder, resync automatico

## Justificativa

**Banco unico com backup seletivo**
- Postgres nao suporta backup parcial de forma simples
- Teria que fazer dump de tabelas especificas
- Complexidade de restore aumenta

**Banco unico sem backup de mensagens via exclusao**
- Risco de excluir tabela errada
- Schema fica confuso (algumas tabelas "especiais")
- Nao resolve o problema de performance

**Dois bancos separados**
- Politica de backup clara por instancia
- Queries de mensagens nao competem com core
- Restore do principal nao precisa trazer terabytes de mensagens

## Consequencias

- Custo de infra levemente maior (duas instancias)
- Prisma precisa de dois clients separados
- Joins entre bancos nao sao possiveis (design intencional)
- Connection string diferente por servico
