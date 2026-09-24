# Riscos

Este documento classifica os riscos iniciais do LabTrack. Um risco é um problema
que ainda não aconteceu, mas pode acontecer. Cada risco recebe uma nota de
probabilidade e de impacto; o cruzamento das duas define o nível e, com ele, a
prioridade de proteção técnica.

## Critérios de classificação

- **Probabilidade**: chance de o problema ocorrer no uso normal do sistema.
  - Baixa: improvável, depende de falha rara ou ação incomum.
  - Média: pode ocorrer algumas vezes ao longo do uso.
  - Alta: tende a ocorrer com frequência, principalmente em dias cheios
    (cerca de 80 pedidos).
- **Impacto**: tamanho do prejuízo caso o problema ocorra.
  - Baixo: incômodo pontual, resolvido sem perda relevante.
  - Médio: atrasa a operação do laboratório ou exige correção manual.
  - Alto: perda de ativo, conflito entre usuários, exposição de dados ou perda de
    confiança no sistema.

## Níveis

| Probabilidade \ Impacto | Baixo | Médio | Alto        |
| ----------------------- | ----- | ----- | ----------- |
| **Alta**                | Médio | Alto  | **Crítico** |
| **Média**               | Baixo | Médio | Alto        |
| **Baixa**               | Baixo | Baixo | Médio       |

- **Crítico**: exige proteção técnica no backend e no banco, com teste
  automatizado.
- **Alto**: exige mitigação implementada e teste da regra.
- **Médio**: exige mitigação planejada e monitoramento.
- **Baixo**: basta documentar e acompanhar.

## Riscos identificados

### R-01 - Conflito de reserva

- **Descrição**: duas solicitações para o mesmo ativo são aprovadas ou retiradas
  ao mesmo tempo, e duas pessoas aparecem para o mesmo equipamento.
- **Onde ocorre no fluxo**: aprovar solicitação e confirmar retirada.
- **Probabilidade**: Alta, pelo volume de pedidos e pelo acesso simultâneo de
  gestores.
- **Impacto**: Alto, pois gera conflito entre usuários e quebra a confiança na
  disponibilidade mostrada.
- **Nível**: **Crítico**.
- **Mitigação**: validação de disponibilidade e alteração de status em transação
  atômica no backend, com restrição no banco que impeça dois empréstimos ativos
  para o mesmo ativo; teste de concorrência.
- **Relacionados**: RB-01, RNF-03.

### R-02 - Ativo perdido

- **Descrição**: um ativo sai do laboratório e ninguém sabe com certeza quem
  está com ele ou quando deveria voltar.
- **Onde ocorre no fluxo**: entre a confirmação da retirada e a confirmação da
  devolução.
- **Probabilidade**: Média.
- **Impacto**: Alto, pois envolve prejuízo financeiro e interrupção do uso do
  equipamento.
- **Nível**: **Alto**.
- **Mitigação**: registro imutável de retirada, devolução e manutenção com
  ativo, usuário, data/hora e ação; marcação automática de empréstimos
  atrasados por job agendado; consulta do histórico pelo gestor.
- **Relacionados**: RB-04, RB-05, RNF-04, RNF-05.

### R-03 - Ativo danificado disponível para reserva

- **Descrição**: um item com defeito continua aparecendo como disponível e é
  reservado ou emprestado novamente.
- **Onde ocorre no fluxo**: confirmação da devolução e consulta de ativos
  disponíveis.
- **Probabilidade**: Média.
- **Impacto**: Médio, pois o solicitante recebe um equipamento inutilizável e o
  uso planejado é atrasado.
- **Nível**: **Médio**.
- **Mitigação**: decisão obrigatória na devolução entre disponível e
  manutenção; bloqueio de reserva e empréstimo para ativos em manutenção;
  recusa de transições de estado inválidas.
- **Relacionados**: RB-02, RNF-04.

### R-04 - Indisponibilidade do sistema

- **Descrição**: o sistema fica fora do ar ou perde dados, e o laboratório volta
  a depender de planilhas e mensagens.
- **Onde ocorre no fluxo**: todo o fluxo.
- **Probabilidade**: Baixa.
- **Impacto**: Médio, pois a operação continua de forma manual, mas com risco de
  inconsistência ao voltar.
- **Nível**: **Baixo**.
- **Mitigação**: rotina documentada de backup e restauração; meta de desempenho
  para a consulta de ativos; monitoramento básico.
- **Relacionados**: RNF-06, RNF-07.

### R-05 - Exposição de dados pessoais e acesso indevido

- **Descrição**: senhas, tokens ou dados pessoais vazam, ou um usuário executa
  operações que não pertencem ao seu papel (por exemplo, aprovar a própria
  solicitação).
- **Onde ocorre no fluxo**: login, ações do gestor e registros de log.
- **Probabilidade**: Baixa.
- **Impacto**: Alto, pois envolve dado pessoal e permite burlar as regras de
  aprovação.
- **Nível**: **Médio**.
- **Mitigação**: senhas com hash seguro; expiração de sessão; autorização por
  papel validada no backend; logs sem senhas, tokens ou dados pessoais
  desnecessários.
- **Relacionados**: RB-03, RNF-01, RNF-02.

## Resumo

| ID   | Risco                                    | Probabilidade | Impacto | Nível       |
| ---- | ---------------------------------------- | ------------- | ------- | ----------- |
| R-01 | Conflito de reserva                      | Alta          | Alto    | **Crítico** |
| R-02 | Ativo perdido                            | Média         | Alto    | Alto        |
| R-03 | Ativo danificado disponível para reserva | Média         | Médio   | Médio       |
| R-05 | Exposição de dados e acesso indevido     | Baixa         | Alto    | Médio       |
| R-04 | Indisponibilidade do sistema             | Baixa         | Médio   | Baixo       |

R-01 e R-02 estão no topo da matriz. Por isso, RB-01 (um empréstimo ativo por
ativo) e RB-04 (histórico imutável) são as regras que merecem maior proteção
técnica: restrição no banco, transação atômica e trilha de auditoria que não
pode ser alterada pela interface.
