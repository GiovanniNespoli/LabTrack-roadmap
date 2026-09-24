# ADR-001 - Monolito modular como ponto de partida

- **Status**: Aceita
- **Data**: 2026-09-24

## Contexto

O LabTrack é um produto interno da Orion Labs para reservas e empréstimos de
cerca de 350 ativos, com picos de 80 pedidos por dia. O domínio é pequeno, a
equipe é pequena e o objetivo da primeira versão é entregar a jornada
consultar → solicitar → aprovar → retirar → devolver → manter com
disponibilidade confiável.

As regras mais críticas ([análise de riscos](../riscos.md)) são:

- **RB-01**: um ativo não pode ter dois empréstimos ativos ao mesmo tempo.
- **RB-04**: retirada, devolução e manutenção geram histórico imutável.

Ambas exigem que a mudança de status do ativo, do empréstimo e o registro de
auditoria aconteçam juntos, de forma atômica.

## Opções consideradas

1. **Monolito tradicional**: uma aplicação sem fronteiras internas definidas.
   Simples no início, mas as regras tendem a se espalhar e a se misturar com a
   interface e a persistência.
2. **Microserviços**: serviços separados para catálogo, empréstimos, manutenção
   e auditoria. Isolamento forte, mas exige transações distribuídas para
   RB-01/RB-04, mais infraestrutura e mais operação do que a equipe e o volume
   justificam.
3. **Monolito modular**: uma aplicação e um banco, divididos em módulos com
   fronteiras explícitas e comunicação apenas por APIs públicas internas.

## Decisão

Adotar um **monolito modular no backend** (Node.js + TypeScript + PostgreSQL),
com um **frontend Next.js como container separado**, ambos em um monorepo.

- Módulos iniciais: `catalog`, `loans`, `maintenance`, `audit` e `identity`.
- Cada módulo tem as camadas `domain`, `application`, `infra` e `http`.
- Um módulo só acessa outro pelo seu `index.ts` (API pública); é proibido
  importar `domain` ou `infra` de outro módulo ou acessar suas tabelas.
- Regras e transições de estado ficam no domínio do backend, nunca no frontend.

## Consequências

**Positivas**

- RB-01 e RB-04 são garantidas por uma única transação no mesmo banco.
- Um único deploy do backend: menos infraestrutura e operação.
- Regras de negócio testáveis sem depender da interface (RNF-09).
- Fronteiras prontas para extrair um módulo como serviço se houver necessidade
  concreta (capítulos 07 e 10).

**Negativas**

- Todos os módulos escalam juntos.
- Uma falha grave no processo afeta todo o backend.
- As fronteiras dependem de disciplina: sem verificação automática, os módulos
  podem voltar a se acoplar.

**Mitigações**

- Regras de lint de dependência (`dependency-cruiser` ou
  `eslint-plugin-boundaries`) no pipeline para bloquear importações entre
  camadas internas de módulos diferentes.
- Revisão desta ADR ao fim dos capítulos 07 e 10.

## Como reverter

Se um módulo precisar escalar ou ser implantado de forma independente:

1. Manter a interface pública do módulo e trocar sua implementação por um
   cliente HTTP ou por mensagens em fila.
2. Mover as tabelas do módulo para um banco próprio.
3. Substituir as garantias transacionais que cruzam módulos por eventos e
   consistência eventual, registrando a decisão em uma nova ADR.
