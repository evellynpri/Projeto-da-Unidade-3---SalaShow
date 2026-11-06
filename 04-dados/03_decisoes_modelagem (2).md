# Decisões de Modelagem: SalaShow (Reserva de Salas)

**Equipe:** Grupo 02: Evellyn, Anna Julia Soares, Selena, Jullia Santini, Eduardo Barcelos
**Versão:** v1 (06/11/2026)
**Base:** narrativa do cenário (edital, Seção 6), `01_inventario_dados.md` e `02_modelo_conceitual_v1` (DER em notação Chen)

Este documento registra **o que é entidade, atributo ou relacionamento no DER v1 e por quê**. Cada decisão tem justificativa e, quando houve, a alternativa descartada. A seção 7 lista os pontos em que o DER ainda se afasta do enunciado.

---

## 1. Visão geral do DER v1

| Item | Como aparece no DER | Mínimo do edital |
| :--- | :--- | :---: |
| Entidades (sem contar a associativa do N:N) | 5: Usuário, Administrador, Sala, Reserva, Cancelamento | 5 |
| Relacionamento N:N | Administrador ↔ Sala (`gerencia`, com `data_acao` e `tipo_acao`) | 1 |
| Entidade dependente (fraca) | Cancelamento (borda dupla; losango duplo `gera`) | 1 |
| Atributo multivalorado | `{recursos_disponiveis}` em Sala | 1 |
| Atributo derivado | `/duracao_minutos` em Reserva | 1 |
| Tabelas no modelo lógico | 8 (ver seção 6) | 6 |

**Relacionamentos do DER:**

| Relacionamento | Entidades | Cardinalidade | Participação |
| :--- | :--- | :---: | :--- |
| `solicita` | Usuário – Reserva | 1:N | Reserva obrigatória (linha contínua) |
| `recebe` | Sala – Reserva | 1:N | Reserva obrigatória (linha contínua) |
| `analisa` | Administrador – Reserva | 1:N | Opcional nos dois lados (tracejada) |
| `gerencia` | Administrador – Sala | N:N | Atributos próprios: `data_acao`, `tipo_acao` |
| `gera` (identificador) | Reserva – Cancelamento | 1:1 | Reserva opcional; Cancelamento obrigatório |

---

## 2. Decisões sobre entidades

### D-01. Reserva é uma entidade (e não só um relacionamento)
- **Decisão:** Reserva tem identificador próprio (`id_reserva`) e liga Usuário e Sala por dois relacionamentos 1:N (`solicita` e `recebe`).
- **Por quê:** o mesmo usuário pode reservar a mesma sala várias vezes, então o par (usuário, sala) não identifica a reserva. Ela também tem atributos e ciclo de vida próprios (data, horários, finalidade, status).
- **Participação obrigatória:** toda reserva tem exatamente um usuário e uma sala. No lógico, as FKs ficam NOT NULL.

### D-02. Cancelamento é entidade fraca de Reserva
- **Decisão:** Cancelamento não tem chave própria. Sua identificação vem de Reserva por meio do relacionamento identificador `gera` (losango duplo). A participação de Cancelamento é total e a de Reserva é opcional.
- **Por quê:** um cancelamento não existe sem a reserva que o originou, e cada reserva é cancelada no máximo uma vez. Manter data, motivo e responsável em outra entidade evita colunas quase sempre vazias em Reserva.
- **No lógico:** a PK de `cancelamento` é a própria `id_reserva`, que também é FK.
- **Descartado:** campos de cancelamento dentro de Reserva (muitos NULLs).

### D-03. Administrador é entidade separada de Usuário
- **Decisão:** Administrador é uma entidade própria, com `login` e `senha_hash`, e não um tipo de Usuário.
- **Por quê:** os atributos são diferentes (credenciais de acesso ao painel) e o papel é outro: o administrador analisa reservas e gerencia salas, enquanto o usuário solicita. Misturar os dois deixaria `login` e `senha_hash` vazios para quase todos os usuários.

### D-04. `analisa`: o administrador que analisa a reserva é opcional
- **Decisão:** Administrador 1:N Reserva com participação opcional nos dois lados (linha tracejada).
- **Por quê:** uma reserva recém-criada (status Pendente) ainda não foi analisada, e um administrador pode nunca ter analisado nenhuma. No lógico, `id_admin_aprovador` é uma FK **que aceita NULL** em `reserva`.

### D-05. `gerencia` é o N:N Administrador ↔ Sala, com atributos próprios
- **Decisão:** um administrador gerencia várias salas e uma sala é gerenciada por vários administradores. Cada ação registra `data_acao` e `tipo_acao` (por exemplo: cadastro, atualização, manutenção).
- **Por quê:** esses dois atributos não pertencem nem ao Administrador nem à Sala, só existem no encontro dos dois. É o caso de atributo do relacionamento.
- **No lógico:** vira a tabela associativa `gerencia`. Como o mesmo administrador pode agir várias vezes sobre a mesma sala, o par (admin, sala) **não pode ser PK**. Por isso a tabela usa `id_gerencia` como chave substituta, com FKs para as duas entidades.

### D-06. Identificadores
- **Decisão:** todas as entidades fortes usam **identificador substituto** (`id_usuario`, `id_admin`, `id_sala`, `id_reserva`). Os identificadores naturais ficam como **UNIQUE**: `cpf`, `email` e `login`.
- **Por quê:** é uma decisão de projeto, e não obrigação, como visto em aula. Chaves substitutas não mudam, e as naturais podem ser corrigidas sem quebrar as referências.

---

## 3. Decisões sobre atributos

### D-07. `{recursos_disponiveis}` é multivalorado
- **Decisão:** no DER, os recursos de uma sala aparecem como atributo **multivalorado** (chaves em `{ }`), pois uma sala tem vários recursos (projetor, ar-condicionado, quadro branco).
- **Por quê:** o enunciado diz que uma sala pode ter mais de um recurso. Marcar como multivalorado deixa claro que não é um valor único.
- **No lógico:** conforme visto em aula, atributo multivalorado vira **tabela própria**. Os recursos ficam num **cadastro** (`recurso`) ligado a Sala pela tabela associativa `sala_recurso` (seção 6). Assim a busca "sala com projetor" parte do cadastro de recursos e não de texto livre, como o enunciado exige. O mesmo recurso pode aparecer em várias salas.
- **Descartado:** guardar a lista em uma coluna TEXT/JSON. Isso fere a 1FN e impede busca confiável ("Projetor" ≠ "projetor" ≠ "datashow").

### D-08. `/duracao_minutos` é derivado
- **Decisão:** a duração da reserva é derivada de `horario_termino − horario_inicio`. Aparece em itálico no DER e **não vira coluna** no lógico.
- **Por quê:** é calculável a partir de dados já guardados. Armazená-lo criaria um segundo lugar para o mesmo dado, que poderia divergir.

### D-09. Atributos com domínio fechado (ENUM)
- **Decisão:** estes atributos só aceitam valores de uma lista fechada (restrição de domínio, equivalente a um CHECK):
  - `Usuário.tipo_usuario`: Aluno, Professor, Funcionário
  - `Sala.tipo_sala`: Reunião, Laboratório, Aula, etc.
  - `Sala.situacao`: Disponível, Ocupada, Em manutenção
  - `Reserva.status`: Pendente, Aprovada, Recusada, Cancelada
  - `Cancelamento.responsavel`: Usuário ou Admin
- **Por quê:** evita grafias diferentes e garante que regras de negócio possam testar valores exatos (por exemplo, "só reservas Aprovadas bloqueiam horário").

### D-10. Atributos obrigatórios e opcionais
- **Opcional:** apenas `Usuário.telefone` (identificado como "opc." no DER) e a FK `id_admin_aprovador` (D-04).
- **Obrigatórios:** todos os demais, inclusive `finalidade` e `qtd_pessoas` em Reserva, e `motivo` em Cancelamento.
- **Por quê:** telefone é contato complementar, enquanto os demais dados são necessários para o sistema funcionar.

### D-11. `data_solicitacao` separada de `data_reserva`
- **Decisão:** `data_solicitacao` é **quando** o pedido foi feito; `data_reserva` é **o dia** em que a sala será usada.
- **Por quê:** são fatos diferentes, e a primeira serve para auditoria e fila de análise.

---

## 4. Decisão sobre "choque de horários"

- **Regra:** duas reservas **Aprovadas** da mesma sala e da mesma data não podem se sobrepor. Há sobreposição quando
  `novo.inicio < existente.termino` **e** `novo.termino > existente.inicio`.
- **Decisão:** registrar como **regra de processo (verificação antes de aprovar a reserva)**, documentada em `06_regras_integridade.md`, e **não** como CHECK.
- **Por quê:** como visto na aula (limite do CHECK), um CHECK só enxerga a própria linha e não compara com outras reservas. Registrar explicitamente evita esconder a regra atrás de uma restrição que "finge resolver".
- **O que o esquema garante:** `horario_termino > horario_inicio` e `qtd_pessoas > 0` (CHECK de linha).

## 5. Decisão sobre "cancelar libera o horário"

- Ao cancelar, `status` passa para **Cancelada** e nasce o registro em Cancelamento (entidade fraca). Como o bloqueio de horário considera **só reservas Aprovadas**, o horário fica livre **sem apagar nada**, e o histórico se mantém.
- **Redundância controlada:** `status = 'Cancelada'` e a existência de um Cancelamento expressam o mesmo fato. A regra de integridade é: *status Cancelada ⇔ existe Cancelamento para aquela reserva*, e será listada em `06_regras_integridade.md`.

---

## 6. Modelo lógico esperado (`04_modelo_logico_v1`)

| Tabela | PK | FKs | Origem no DER |
| :--- | :--- | :--- | :--- |
| `usuario` | `id_usuario` | | Entidade Usuário |
| `administrador` | `id_admin` | | Entidade Administrador |
| `sala` | `id_sala` | | Entidade Sala (sem a coluna `recursos_disponiveis`) |
| `reserva` | `id_reserva` | `id_usuario` (NOT NULL), `id_sala` (NOT NULL), `id_admin_aprovador` (NULL) | Entidade Reserva + `solicita`, `recebe`, `analisa` |
| `cancelamento` | `id_reserva` | `id_reserva` | Entidade fraca + `gera` |
| `gerencia` | `id_gerencia` | `id_admin`, `id_sala` | N:N Administrador ↔ Sala |
| `recurso` | `id_recurso` | | Cadastro dos recursos (nome UNIQUE) |
| `sala_recurso` | (`id_sala`, `id_recurso`) | `id_sala`, `id_recurso` | Multivalorado `{recursos_disponiveis}` |

Total: **8 tabelas** (o edital exige pelo menos 6).

---

## 7. Pontos de atenção (o DER v1 versus o enunciado)

O DER foi construído a partir do inventário, e o inventário era genérico. Vale a equipe revisar estes pontos em 10/11, porque o professor pode perguntar sobre eles na defesa:

1. **Recursos ainda não aparecem como entidade no DER.** O enunciado diz que "um mesmo recurso está disponível em várias salas", o que é um N:N entre Sala e Recurso. Com `{recursos_disponiveis}` como atributo, o DER mostra só metade da ideia. A solução lógica (seção 6) resolve, mas o ideal na v2 é incluir a entidade **Recurso** e o N:N **Sala ↔ Recurso** no próprio DER. Isso também deixaria a busca por recurso explícita no conceitual.
2. **Identificação do solicitante.** O enunciado fala em **matrícula, nome, setor e telefone**. O DER usa `cpf`, `email`, `curso_depto` e `tipo_usuario`, e não tem matrícula. Sugestão: incluir `matricula` (UNIQUE) em Usuário e confirmar se `curso_depto` cobre o "setor".
3. **Nomes de campos de Sala.** O enunciado diz **código, nome, localização e capacidade**. O DER usa `nome_numero`, `bloco_localizacao` e `capacidade_maxima`. Os campos equivalem, mas convém garantir que exista um "código" da sala (pode ser o `nome_numero` ou um campo novo).
4. **Administrador e aprovação não estão no enunciado.** O enunciado fala só em reservar e cancelar. O fluxo Pendente/Aprovada/Recusada e a entidade Administrador são uma **ampliação de domínio** da equipe, permitida pelo edital (Seção 4), e ajudam a atingir o mínimo de entidades e o N:N `gerencia`. Se a regra de choque de horários considerar só reservas Aprovadas, vale explicar na defesa por que o pedido Pendente não bloqueia a sala.
5. **`situacao = 'Ocupada'` em Sala.** Esse estado é derivável das reservas aprovadas e pode virar redundância. Sugestão: restringir `situacao` a Disponível e Em manutenção.
6. **`responsavel` em Cancelamento** é texto (Usuário/Admin). Se a equipe quiser rastrear **qual** administrador cancelou, seria necessária uma FK opcional para Administrador.

---

## 8. Rastreabilidade com as regras de negócio

> Numeração provisória; ajustar conforme `00_catalogo_rn.md`.

| RN | Regra (origem) | Onde aparece no modelo |
| :--- | :--- | :--- |
| RN-01 | A mesma sala não pode ter reservas com horários sobrepostos (cenário) | Regra de processo sobre Reserva (seção 4) + CHECK `termino > inicio` |
| RN-02 | Cancelar uma reserva libera o horário (cenário) | `status` + entidade fraca Cancelamento (seção 5) |
| RN-03 | A busca de sala por recurso parte do cadastro de recursos (cenário) | `{recursos_disponiveis}` → tabelas `recurso` e `sala_recurso` (D-07) |
| RN-04 | A quantidade de pessoas não pode exceder a capacidade da sala (inventário) | Regra de processo: compara `reserva.qtd_pessoas` com `sala.capacidade_maxima` |
| RN-05 | Sala em manutenção não recebe novas reservas (inventário) | Regra de processo sobre `sala.situacao` |
