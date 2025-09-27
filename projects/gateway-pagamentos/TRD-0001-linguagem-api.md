# TRD-0001: Go vs Node para API de pagamentos

- **Status:** Concluida
- **ADR resultante:** Nenhum (decisao incorporada na stack inicial)

## Contexto

API de pagamentos precisa de:
- Latencia baixa (cada ms conta na experiencia de checkout)
- Alta concorrencia (picos de Black Friday)
- Confiabilidade extrema (dinheiro envolvido)
- Integracao com multiplos adquirentes (HTTP clients)

## Opcoes

### A: Node.js (NestJS)

- **Pros:** Ecossistema rico, async nativo, mesmo stack do backoffice, contratacao facil
- **Contras:** Single-threaded (CPU-bound trava), GC pode causar picos de latencia, tipagem runtime

### B: Go

- **Pros:** Compilado, goroutines pra concorrencia, latencia previsivel, binario estatico
- **Contras:** Ecossistema menor, menos bibliotecas prontas, contratacao mais dificil

### C: Java (Spring)

- **Pros:** Ecossistema enterprise, bibliotecas de pagamento maduras, JVM otimizada
- **Contras:** Memoria alta, cold start lento, verboso

### D: Rust

- **Pros:** Performance maxima, memory safety, zero-cost abstractions
- **Contras:** Curva de aprendizado ingreme, tempo de desenvolvimento maior

## Analise

Benchmark com carga simulada (10k req/s):

| Metrica | Node | Go | Java | Rust |
|---------|------|-----|------|------|
| p50 | 12ms | 3ms | 8ms | 2ms |
| p99 | 85ms | 15ms | 45ms | 12ms |
| Memoria | 512MB | 64MB | 1.2GB | 32MB |
| Cold start | 2s | 50ms | 8s | 30ms |

Node tem picos de p99 por causa do GC. Inaceitavel pra pagamentos.

Rust tem melhor performance mas tempo de desenvolvimento estimado 2x maior.

Go equilibra performance e produtividade.

## Decisao

**Go para API de pagamentos**, NestJS para backoffice.

Backoffice e interno, latencia menos critica, pode usar mesma stack de outros projetos.

API de pagamentos e critica, justifica linguagem otimizada.

## Riscos

- Duas stacks pra manter - mitigacao: backoffice e CRUD simples, Go so na API critica
- Pool de devs Go menor - mitigacao: API bem definida, pouca mudanca apos estabilizar
