# TRD-0002: Monolito modular vs Microservices

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada na stack inicial)

## Contexto

Tres dominios principais: vendedor, cliente, admin. Cada um com regras diferentes.

Questao: separar em servicos desde o inicio ou comecar monolito?

Restricoes:
- Time de 3 devs
- Precisa entregar MVP em 3 meses
- Dominios ainda nao estao 100% definidos

## Opcoes

### A: Microservices desde o inicio
- **Pros:** Escala independente, deploys isolados, fronteiras claras
- **Contras:** Overhead de infra, complexidade de comunicacao, latencia entre servicos, debugging distribuido

### B: Monolito modular
- **Pros:** Deploy simples, debugging facil, refatorar barato, time pequeno consegue manter
- **Contras:** Escala junto (tudo ou nada), acoplamento pode crescer se nao cuidar

### C: Monolito agora, microservices depois
- **Pros:** Velocidade agora, aprende o dominio, extrai quando tiver certeza
- **Contras:** Migracao dolorosa se deixar acoplar demais

## Decisao

**Opcao B: Monolito modular** com fronteiras claras entre modulos.

Estrutura de pastas separa vendedor/cliente/admin. Comunicacao entre modulos via interfaces, nao imports diretos. Quando (se) precisar extrair, os cortes ja estao definidos.

## Riscos

- Desenvolvedores ignorarem as fronteiras - mitigacao: lint de arquitetura, PR review focado nisso
- Performance de um modulo afetar outros - mitigacao: monitoramento por modulo, circuit breaker interno se precisar
