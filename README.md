# LabTrack

> Aprender arquitetura de software construindo um sistema real, uma decisão
> defendível por vez.

O **LabTrack** é um sistema de reservas e empréstimos de equipamentos de
laboratório. O domínio é propositalmente simples. O desafio está na
arquitetura: cada capítulo apresenta um problema concreto do produto, que
justifica uma decisão técnica. Essa decisão é implementada, testada, medida e
registrada.

Este repositório é o diário desse percurso. Ele segue o roteiro
[LabTrack - roteiro de arquitetura aplicada](labtrack-roteiro-de-arquitetura-aplicada.pdf),
que converte os tópicos do
[Software Architect Roadmap](https://roadmap.sh/software-architect) em
problemas do LabTrack.

## O problema

A **Orion Labs** opera três laboratórios compartilhados. Cerca de **350 itens**
(notebooks, câmeras, projetores, kits de robótica e ferramentas) circulam
entre pesquisadores e instrutores, com até **80 pedidos** em dias cheios.

Hoje tudo passa por planilhas e mensagens, e o resultado é conhecido:

- duas pessoas aparecem para retirar o mesmo equipamento;
- um item danificado continua aparecendo como disponível;
- ninguém sabe com certeza quem está com cada ativo.

O LabTrack resolve isso com uma jornada única:

```
consultar → solicitar → aprovar → retirar → devolver → manter
```

## Regras de negócio

O produto se apoia em cinco regras curtas. Toda a arquitetura existe para que
elas continuem verdadeiras, inclusive sob falha e concorrência.

| ID    | Regra                                                                  |
| ----- | ---------------------------------------------------------------------- |
| RB-01 | Um ativo não pode ter dois empréstimos ativos ao mesmo tempo.          |
| RB-02 | Ativo em manutenção não pode ser reservado nem emprestado.             |
| RB-03 | Somente o gestor aprova solicitações e confirma retirada e devolução.  |
| RB-04 | Retirada, devolução e manutenção geram histórico imutável.             |
| RB-05 | Empréstimo não devolvido até a data prevista fica atrasado.            |

## Objetivos do projeto

- **Construir critério, não decorar padrões**: cada tecnologia entra só quando
  resolve um problema observável do produto. Quando o tópico não é natural ao
  domínio (SOAP, Hadoop, microfrontends), ele vira um **laboratório isolado**,
  com dados simulados e fronteira clara.
- **Provar cada decisão com evidência**: toda decisão importante tem uma ADR,
  um diagrama, um teste, uma demo e uma métrica.
- **Terminar com um portfólio**: um produto demonstrável, 13 ADRs, diagramas
  C4, testes de regra, contrato, integração, E2E e carga, além de runbooks de
  operação.

### O ciclo de cada capítulo

1. Ler e tomar notas.
2. Desenhar (C4, UML ou fluxo).
3. Registrar uma ADR.
4. Implementar o menor corte vertical.
5. Testar e demonstrar.
6. Defender a decisão em cinco minutos.

## Arquitetura

O LabTrack começa como um **monolito modular**
([ADR-001](step01/adr/ADR-001-monolito-modular.md)): um backend, um banco e
módulos com fronteiras explícitas. Os capítulos seguintes testam os limites
dessa escolha com filas, cache, CQRS e serviços, mas só distribuem o sistema
quando existe um motivo medido para isso.

| Camada         | Responsabilidade                                   | Tecnologia               |
| -------------- | -------------------------------------------------- | ------------------------ |
| Experiência    | Portal do solicitante e painel do gestor           | Next.js + TypeScript     |
| Aplicação      | Casos de uso, autorização, contratos, orquestração | Node.js                  |
| Domínio        | Ativo, empréstimo, transições e políticas          | TypeScript + testes      |
| Infraestrutura | Banco, cache, fila, e-mail simulado, observabilidade | PostgreSQL, Redis, RabbitMQ, Docker |
| Plataforma     | Provisionamento, pipeline, deploy e recuperação    | Terraform + CI/CD        |

> Acoplamento baixo não significa criar muitos serviços. Significa que uma
> mudança em notificação, banco ou interface não altera a regra de que um item
> não pode estar emprestado duas vezes.

## Roteiro

São 35 semanas produtivas, com 4 a 5 horas por semana, a partir de 28/09/2026.

| Cap. | Tema                                                  | Prazo      | Status       |
| ---- | ----------------------------------------------------- | ---------- | ------------ |
| 01   | Descoberta, stakeholders e requisitos                 | 11/10/2026 | ✅ Concluído |
| 02   | Primeiro corte vertical: o produto existe             | 01/11/2026 | ⏳           |
| 03   | Domínio protegido por arquitetura em camadas          | 22/11/2026 | ⏳           |
| 04   | Corretude sob concorrência: o item não duplica        | 13/12/2026 | ⏳           |
| —    | Buffer                                                | 03/01/2027 |              |
| 05   | Segurança, privacidade e trilha de auditoria          | 17/01/2027 | ⏳           |
| 06   | Contratos e integrações (REST, GraphQL, gRPC, SOAP)   | 07/02/2027 | ⏳           |
| 07   | Mensageria e processamento assíncrono                 | 28/02/2027 | ⏳           |
| 08   | Tarefas agendadas, cache e serverless                 | 21/03/2027 | ⏳           |
| 09   | Dados, ETL e análise operacional                      | 11/04/2027 | ⏳           |
| 10   | Limites de distribuição: CQRS e consistência eventual | 02/05/2027 | ⏳           |
| 11   | Experiência web: SSR, SPA, SSG e microfrontends       | 16/05/2027 | ⏳           |
| 12   | Plataforma operável: cloud, IaC e recuperação         | 06/06/2027 | ⏳           |
| 13   | Visão corporativa, integração e defesa final          | 20/06/2027 | ⏳           |
| —    | Buffer final: hardening, portfólio e retrospectiva    | 04/07/2027 |              |

## Como navegar

Cada capítulo tem uma pasta `stepNN/` e uma branch de mesmo nome. A branch é
integrada à `main` com um merge sem fast-forward e **nunca é removida**, então
dá para ver cada capítulo isolado no histórico:

```
*   Merge branch 'step01'
|\
| * docs(step01): ADR-001 monolito modular
| * docs(step01): ...
|/
* chore: adiciona roteiro do LabTrack
```

### Capítulo 01: descoberta

- [Visão do produto e stakeholders](step01/visao.md)
- [Requisitos e regras de negócio](step01/requisitos.md)
- [Riscos](step01/riscos.md)
- [Backlog](step01/backlog.md)
- [Jornada do usuário](step01/jornada.excalidraw) (abra no
  [excalidraw.com](https://excalidraw.com))
- [ADR-001: monolito modular](step01/adr/ADR-001-monolito-modular.md)

---

> O objetivo não é decorar arquitetura. É construir critério para decidir o
> que entra, o que fica de fora, como provar que funciona e como sustentar a
> decisão quando o sistema cresce.
