# Backlog inicial

Histórias pequenas derivadas dos [requisitos e regras de negócio](requisitos.md).
A prioridade segue a jornada principal e a [análise de riscos](riscos.md):
primeiro o que protege RB-01 e RB-04.

**Prioridade**: Alta = necessária para o primeiro corte vertical (capítulo 02);
Média = completa a jornada; Baixa = pode esperar.

| ID    | História                                                                                                                    | Prioridade | Referências          |
| ----- | --------------------------------------------------------------------------------------------------------------------------- | ---------- | -------------------- |
| HU-01 | Como **administrador**, quero cadastrar, editar e desativar ativos, para manter o catálogo atualizado.                       | Alta       | RF-04                |
| HU-02 | Como **usuário**, quero me autenticar, para acessar apenas as funções do meu papel.                                          | Alta       | RNF-01, RNF-02       |
| HU-03 | Como **solicitante**, quero consultar os ativos disponíveis, para saber o que posso reservar.                                | Alta       | RF-01, RB-02, RNF-06 |
| HU-04 | Como **solicitante**, quero solicitar a reserva de um ativo para um período, para garantir o equipamento na data necessária. | Alta       | RF-01                |
| HU-05 | Como **gestor**, quero aprovar ou rejeitar solicitações com justificativa, para controlar o uso dos ativos.                  | Alta       | RF-02, RB-03         |
| HU-06 | Como **gestor**, quero confirmar a retirada de um ativo, para iniciar o empréstimo sem conflitos.                            | Alta       | RF-02, RB-01, RB-04  |
| HU-07 | Como **gestor**, quero confirmar a devolução e decidir se o ativo volta a ficar disponível ou vai para manutenção.           | Alta       | RF-02, RB-04         |
| HU-08 | Como **solicitante**, quero ver minhas reservas e empréstimos com status e datas, para acompanhar meus pedidos.              | Média      | RF-01                |
| HU-09 | Como **solicitante**, quero cancelar uma reserva ainda não retirada, para liberar o ativo para outras pessoas.               | Média      | RF-01                |
| HU-10 | Como **solicitante**, quero informar que vou devolver um ativo, para o gestor se preparar para recebê-lo.                     | Média      | RF-01                |
| HU-11 | Como **gestor**, quero enviar um ativo para manutenção a qualquer momento, para tirá-lo de circulação.                       | Média      | RF-02, RB-02, RB-04  |
| HU-12 | Como **técnico**, quero consultar os ativos em manutenção e o motivo do bloqueio, para organizar os reparos.                 | Média      | RF-03                |
| HU-13 | Como **técnico**, quero registrar a conclusão da manutenção e liberar o ativo, para que ele volte a ser reservado.           | Média      | RF-03, RB-02, RB-04  |
| HU-14 | Como **gestor**, quero que empréstimos vencidos sejam marcados como atrasados automaticamente, para agir sobre eles.          | Média      | RB-05, RNF-04        |
| HU-15 | Como **gestor**, quero consultar o histórico de cada ativo, para saber quem esteve com ele e quando.                          | Média      | RF-02, RB-04         |
| HU-16 | Como **administrador**, quero gerenciar os perfis de acesso dos usuários, para que cada pessoa tenha o papel correto.         | Baixa      | RF-04, RNF-01        |

## Critérios de pronto de cada história

- Regra de negócio implementada no backend, não na interface.
- Teste automatizado cobrindo o caminho feliz e ao menos um caminho de falha.
- Critérios de aceitação das regras relacionadas atendidos.
