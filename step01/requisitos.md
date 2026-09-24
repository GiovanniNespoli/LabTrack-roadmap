# Requisitos e regras de negócio

Este documento reúne os requisitos iniciais do LabTrack. Requisitos funcionais
(RF) descrevem o que o sistema deve permitir; requisitos não funcionais (RNF)
definem as qualidades e restrições que o sistema deve respeitar; regras de
negócio (RB) são as políticas do domínio, cada uma com critérios de aceitação
no formato **Dado / Quando / Então** que servem de base para os testes.

## Requisitos funcionais

### RF-01 - Solicitante (pesquisador ou instrutor)

- Consultar os ativos disponíveis para reserva.
- Solicitar a reserva de um ativo para um período definido.
- Consultar apenas as próprias reservas e empréstimos, com seus respectivos
  status e datas previstas de devolução.
- Cancelar uma reserva enquanto ela ainda não tiver sido retirada.
- Informar a intenção de devolver um ativo.

### RF-02 - Gestor do laboratório

- Consultar todos os ativos, reservas e empréstimos, incluindo seus status.
- Aprovar ou rejeitar solicitações de reserva, informando uma justificativa em
  caso de rejeição.
- Confirmar a retirada de um ativo pelo solicitante, iniciando o empréstimo.
- Confirmar a devolução de um ativo e definir se ele volta a ficar disponível ou
  se deve seguir para manutenção.
- Enviar um ativo para manutenção quando identificar defeito ou necessidade de
  inspeção.
- Consultar o histórico de retirada, devolução e manutenção de cada ativo.

### RF-03 - Técnico de manutenção

- Consultar os ativos em manutenção e o motivo do bloqueio.
- Registrar a conclusão da manutenção.
- Liberar um ativo para disponibilidade após a manutenção, quando ele estiver
  apto para novo uso.

### RF-04 - Administrador

- Cadastrar, editar, desativar e consultar ativos do catálogo.
- Gerenciar perfis de acesso dos usuários.
- Consultar os parâmetros operacionais do sistema.

## Requisitos não funcionais

### RNF-01 - Autenticação e autorização

- O sistema deve exigir autenticação para acessar funcionalidades internas.
- Cada usuário deve acessar somente as operações permitidas pelo seu papel:
  solicitante, gestor, técnico de manutenção ou administrador.
- Um solicitante não pode aprovar a própria solicitação, confirmar retirada ou
  confirmar devolução.

### RNF-02 - Proteção de credenciais e dados pessoais

- Senhas devem ser armazenadas somente com hash seguro; nunca em texto puro.
- Sessões de acesso devem expirar após um período definido de inatividade ou de
  duração máxima.
- Logs não devem registrar senhas, tokens de acesso ou dados pessoais além do
  necessário para diagnóstico.

### RNF-03 - Consistência e concorrência

- Um ativo não pode possuir dois empréstimos ativos ao mesmo tempo, mesmo que
  duas solicitações sejam processadas simultaneamente.
- A validação de disponibilidade e a alteração de status devem ocorrer no
  backend de forma atômica.

### RNF-04 - Integridade dos estados

- Um ativo em manutenção não pode ser reservado nem emprestado.
- O sistema deve recusar transições de estado inválidas para ativos, reservas e
  empréstimos.
- Um empréstimo cuja data prevista de devolução tenha passado deve ser marcado
  como atrasado.

### RNF-05 - Auditoria

- Retiradas, devoluções, entradas em manutenção e liberações devem registrar
  ativo, usuário responsável, data/hora e ação executada.
- Os registros de auditoria não podem ser alterados pela interface do sistema.

### RNF-06 - Desempenho

- A consulta da lista de ativos disponíveis deve responder em até 2 segundos em
  condições normais de uso.

### RNF-07 - Recuperação de dados

- O sistema deve possuir uma rotina documentada de backup e restauração dos
  dados de ativos, reservas e empréstimos.

### RNF-08 - Usabilidade e acessibilidade

- As telas principais devem funcionar em celular e desktop.
- Campos devem possuir rótulos claros, e ações inválidas devem apresentar
  mensagens compreensíveis ao usuário.

### RNF-09 - Manutenibilidade

- As regras de negócio devem permanecer separadas da interface e da camada de
  persistência, permitindo testes sem depender do frontend.

## Regras de negócio

### Estados

| Entidade   | Estados                                                                                  |
| ---------- | ---------------------------------------------------------------------------------------- |
| Ativo      | AVAILABLE, RESERVED, LOANED, IN_MAINTENANCE, RETIRED                                     |
| Empréstimo | REQUESTED, APPROVED, READY_FOR_PICKUP, CHECKED_OUT, RETURNED, OVERDUE, REJECTED, CANCELLED |

### RB-01 - Um ativo não pode ter dois empréstimos ativos ao mesmo tempo

**Validação**: teste de concorrência e restrição no banco.

- **CA-01.1**: Dado um ativo com empréstimo CHECKED_OUT, quando o gestor tenta
  confirmar outra retirada desse ativo, então a operação é recusada com a
  mensagem "Ativo já emprestado".
- **CA-01.2**: Dado um ativo AVAILABLE e duas confirmações de retirada enviadas
  ao mesmo tempo, quando ambas são processadas, então apenas uma é concluída e a
  outra é recusada.
- **CA-01.3**: Dado um ativo com empréstimo OVERDUE, quando o gestor tenta
  confirmar uma nova retirada, então a operação é recusada, pois o empréstimo
  atrasado continua ativo.

### RB-02 - Ativo em manutenção não pode ser reservado nem emprestado

**Validação**: teste de transição de estado.

- **CA-02.1**: Dado um ativo IN_MAINTENANCE, quando o solicitante consulta os
  ativos disponíveis, então esse ativo não aparece na lista.
- **CA-02.2**: Dado um ativo IN_MAINTENANCE, quando alguém tenta solicitar sua
  reserva, então a operação é recusada com a mensagem "Ativo em manutenção".
- **CA-02.3**: Dado um ativo IN_MAINTENANCE, quando o técnico registra a
  conclusão e o libera, então o ativo passa para AVAILABLE e volta a aparecer
  na lista.

### RB-03 - Somente o gestor aprova solicitações e confirma retirada e devolução

**Validação**: teste de autorização por papel.

- **CA-03.1**: Dado um usuário com papel solicitante, quando ele tenta aprovar
  uma solicitação (inclusive a própria), então a operação é recusada com erro
  de permissão.
- **CA-03.2**: Dado um usuário com papel solicitante ou técnico, quando ele
  tenta confirmar retirada ou devolução, então a operação é recusada com erro
  de permissão.
- **CA-03.3**: Dado um usuário com papel gestor, quando ele aprova uma
  solicitação REQUESTED, então ela passa para APPROVED.

### RB-04 - Retirada, devolução e manutenção geram histórico imutável

**Validação**: teste de auditoria e consulta de trilha.

- **CA-04.1**: Dado um empréstimo APPROVED, quando o gestor confirma a
  retirada, então é gravado um registro com ativo, usuário responsável,
  data/hora e ação "retirada".
- **CA-04.2**: Dado um ativo com histórico, quando o gestor consulta a trilha
  do ativo, então vê retiradas, devoluções, entradas em manutenção e
  liberações em ordem cronológica.
- **CA-04.3**: Dado um registro de auditoria existente, quando qualquer usuário
  tenta alterá-lo ou excluí-lo pela interface ou pela API, então a operação não
  está disponível ou é recusada.

### RB-05 - Empréstimo não devolvido até a data prevista fica atrasado

**Validação**: job agendado e teste de tempo.

- **CA-05.1**: Dado um empréstimo CHECKED_OUT com data prevista de devolução
  ontem, quando o job agendado é executado, então o empréstimo passa para
  OVERDUE.
- **CA-05.2**: Dado um empréstimo CHECKED_OUT com data prevista de devolução
  amanhã, quando o job é executado, então o status não muda.
- **CA-05.3**: Dado um empréstimo OVERDUE, quando o gestor confirma a
  devolução, então o empréstimo passa para RETURNED.
