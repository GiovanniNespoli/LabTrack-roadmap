# Visão do produto e stakeholders - LabTrack

## Problema

A Orion Labs opera três laboratórios compartilhados, com cerca de 350 ativos
(notebooks, câmeras, projetores, kits de robótica e ferramentas) circulando
entre pesquisadores e instrutores. Em dias cheios chegam a 80 pedidos. Hoje,
planilhas e mensagens resolvem o básico, mas geram conflitos: duas pessoas
aparecem para o mesmo equipamento, itens danificados continuam como
disponíveis e ninguém sabe com certeza quem está com cada ativo.

## Proposta

O LabTrack é um produto interno para reservas e empréstimos de ativos de
laboratório. Ele mostra a disponibilidade de forma confiável e conduz cada
ativo pela jornada **consultar → solicitar → aprovar → retirar → devolver →
manter** ([jornada.excalidraw](jornada.excalidraw)), registrando cada etapa.

## Stakeholders

Stakeholders são todas as pessoas ou áreas afetadas, direta ou indiretamente,
pelo LabTrack.

| Stakeholder                      | Papel no sistema      | Objetivo                                                                             | Poder de decisão                                                   | Dor atual                                                                                        |
| -------------------------------- | --------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| Diretora de operações            | Patrocinadora         | Ter disponibilidade confiável e rastreabilidade dos ativos, reduzindo perdas.        | **Alto**: aprova o produto, define escopo e prioridades.           | Não tem visão confiável de onde estão os ativos nem de quantos estão atrasados ou em manutenção. |
| Gestores do laboratório          | Gestor                | Aprovar pedidos e controlar retiradas e devoluções sem conflitos.                    | **Médio**: valida o fluxo operacional e as regras do dia a dia.    | Conciliam planilhas e mensagens manualmente; recebem pedidos duplicados para o mesmo item.       |
| Pesquisadores e instrutores      | Solicitante           | Saber o que está disponível e garantir o equipamento na data necessária.             | **Baixo**: influenciam pela adesão e pelo feedback de usabilidade. | Chegam para retirar e o equipamento já está com outra pessoa ou está danificado.                 |
| Técnicos de manutenção           | Técnico de manutenção | Saber quais itens precisam de reparo e por quê, e liberá-los quando estiverem aptos. | **Baixo**: definem o critério técnico de liberação.                | Itens com defeito não são sinalizados e voltam a circular sem inspeção.                          |
| Administrador (TI da Orion Labs) | Administrador         | Manter catálogo, perfis de acesso e parâmetros operacionais com segurança.           | **Médio**: define restrições de segurança, hospedagem e acesso.    | Catálogo desatualizado e acessos concedidos de forma informal.                                   |

**Indiretos**: financeiro e compras não usam o sistema na primeira versão, mas
se beneficiam do histórico para identificar perdas e itens com manutenção
recorrente.

## Dores e objetivos

| Dor                                             | Objetivo do LabTrack                                             | Como medir                                                    | Regra |
| ----------------------------------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------- | ----- |
| Duas pessoas aparecem para o mesmo equipamento  | Garantir no máximo um empréstimo ativo por ativo                 | Zero ativos com dois empréstimos ativos ao mesmo tempo.       | RB-01 |
| Item danificado continua como disponível        | Bloquear reserva e empréstimo de itens em manutenção             | Nenhum ativo em manutenção reservado ou emprestado.           | RB-02 |
| Qualquer pessoa altera a planilha               | Restringir aprovação, retirada e devolução ao gestor             | Nenhuma aprovação feita por quem não é gestor.                | RB-03 |
| Ninguém sabe com certeza quem está com o item   | Registrar histórico imutável de retirada, devolução e manutenção | 100% das retiradas e devoluções com registro de auditoria.    | RB-04 |
| Atrasos só são percebidos quando alguém reclama | Marcar automaticamente empréstimos atrasados                     | Empréstimos vencidos marcados como atrasados automaticamente. | RB-05 |

Além disso, a lista de ativos disponíveis deve responder em até 2 segundos
(RNF-06).

## Escopo da primeira versão

**Dentro do escopo**

- Catálogo de ativos e perfis de acesso.
- Solicitação, aprovação, rejeição e cancelamento de reservas.
- Confirmação de retirada e devolução.
- Envio para manutenção e liberação do ativo.
- Marcação de empréstimos atrasados.
- Histórico de auditoria por ativo.

**Fora do escopo**

- Compras, faturamento e contratos.
- Integração com sistemas externos da empresa.
- Aplicativo móvel nativo (as telas devem ser responsivas).

## Restrições e premissas

- Equipe pequena e domínio simples: a arquitetura começa como monolito modular
  ([ADR-001](adr/ADR-001-monolito-modular.md)).
- Stack-base: TypeScript, Node.js, Next.js e PostgreSQL.
- Regras de negócio e transições de estado ficam no backend, nunca na interface.
- RB-01 e RB-04 recebem a maior proteção técnica, conforme a
  [análise de riscos](riscos.md).

## Documentos da etapa

- [Requisitos e regras de negócio](requisitos.md)
- [Riscos](riscos.md)
- [Backlog](backlog.md)
- [Jornada do usuário](jornada.excalidraw)
- [ADR-001 - Monolito modular](adr/ADR-001-monolito-modular.md)
