# Inventário de Dados — Sistema de Reserva de Salas

O presente documento consolida os dados essenciais para o funcionamento do sistema de reserva de salas, servindo como base técnica para a elaboração do Diagrama Entidade-Relacionamento (DER) e do Modelo Lógico Relacional.

---

## 1. Dicionário de Dados e Atributos

### 1.1 Usuário
Representa os solicitantes do espaço físico no sistema.

| Atributo | Tipo de Dado | Restrição / Chave | Descrição / Observação |
| :--- | :--- | :--- | :--- |
| `id_usuario` | Inteiro / UUID | Chave Primária (PK) | Identificador exclusivo do solicitante |
| `nome_completo` | VARCHAR(100) | Obrigatório (NOT NULL) | Nome completo do usuário |
| `cpf` | VARCHAR(11) | Obrigatório, Único (UNIQUE) | Cadastro de Pessoa Física (apenas dígitos) |
| `email` | VARCHAR(100) | Obrigatório, Único (UNIQUE) | E-mail para contato e notificações |
| `telefone` | VARCHAR(15) | Opcional (NULL) | Telefone/WhatsApp com DDD |
| `curso_depto` | VARCHAR(50) | Obrigatório (NOT NULL) | Curso acadêmico ou departamento de vínculo |
| `tipo_usuario` | ENUM | Obrigatório (NOT NULL) | Classificação: `Aluno`, `Professor`, `Funcionário` |

---

### 1.2 Sala
Representa o espaço físico a ser reservado e seus atributos de infraestrutura.

| Atributo | Tipo de Dado | Restrição / Chave | Descrição / Observação |
| :--- | :--- | :--- | :--- |
| `id_sala` | Inteiro / UUID | Chave Primária (PK) | Identificador exclusivo da sala |
| `nome_numero` | VARCHAR(50) | Obrigatório (NOT NULL) | Exemplo: "Laboratório 3", "Auditório A", "Sala 204" |
| `bloco_localizacao` | VARCHAR(50) | Obrigatório (NOT NULL) | Informação de bloco, prédio ou andar |
| `capacidade_maxima` | Inteiro | Obrigatório (CHECK > 0) | Capacidade total de assentos/lotação |
| `tipo_sala` | ENUM | Obrigatório (NOT NULL) | Categoria: `Reunião`, `Laboratório`, `Aula`, etc. |
| `recursos_disponiveis` | TEXT / JSON | Opcional (NULL) | Lista de itens (ex: projetor, ar-condicionado, lousa) |
| `situacao` | ENUM | Obrigatório (NOT NULL) | Estado atual: `Disponível`, `Ocupada`, `Em manutenção` |

---

### 1.3 Administrador
Representa os usuários com permissões de gestão do sistema.

| Atributo | Tipo de Dado | Restrição / Chave | Descrição / Observação |
| :--- | :--- | :--- | :--- |
| `id_admin` | Inteiro / UUID | Chave Primária (PK) | Identificador do administrador |
| `nome_completo` | VARCHAR(100) | Obrigatório (NOT NULL) | Nome do responsável pelo sistema |
| `email` | VARCHAR(100) | Obrigatório, Único (UNIQUE) | E-mail funcional |
| `login` | VARCHAR(50) | Obrigatório, Único (UNIQUE) | Identificador de acesso ao painel |
| `senha_hash` | VARCHAR(255) | Obrigatório (NOT NULL) | Hash criptográfico da credencial de acesso |

---

### 1.4 Reserva
Representa a solicitação de uso de uma sala em determinada data e horário.

| Atributo | Tipo de Dado | Restrição / Chave | Descrição / Observação |
| :--- | :--- | :--- | :--- |
| `id_reserva` | Inteiro / UUID | Chave Primária (PK) | Identificador exclusivo da reserva |
| `id_usuario` | Inteiro / UUID | Chave Estrangeira (FK) | Usuário que solicitou o agendamento |
| `id_sala` | Inteiro / UUID | Chave Estrangeira (FK) | Sala solicitada |
| `id_admin_aprovador` | Inteiro / UUID | Chave Estrangeira (FK, NULL) | Administrador que analisou o pedido |
| `data_solicitacao` | TIMESTAMP | Obrigatório (CURRENT_TIMESTAMP) | Data e hora em que a reserva foi solicitada |
| `data_reserva` | DATE | Obrigatório (NOT NULL) | Data pretendida para utilização |
| `horario_inicio` | TIME | Obrigatório (NOT NULL) | Início do período de ocupação |
| `horario_termino` | TIME | Obrigatório (CHECK > inicio) | Fim do período de ocupação |
| `finalidade` | VARCHAR(255) | Obrigatório (NOT NULL) | Justificativa ou descrição do evento |
| `qtd_pessoas` | Inteiro | Obrigatório (CHECK > 0) | Total previsto de participantes |
| `status` | ENUM | Obrigatório (NOT NULL) | Situação: `Pendente`, `Aprovada`, `Recusada`, `Cancelada` |

---

### 1.5 Cancelamento
Entidade de auditoria responsável por rastrear justificativas e responsáveis pelo cancelamento.

| Atributo | Tipo de Dado | Restrição / Chave | Descrição / Observação |
| :--- | :--- | :--- | :--- |
| `id_cancelamento` | Inteiro / UUID | Chave Primária (PK) | Identificador do registro de cancelamento |
| `id_reserva` | Inteiro / UUID | Chave Estrangeira (FK, UNIQUE) | Reserva associada ao cancelamento |
| `data_cancelamento` | TIMESTAMP | Obrigatório (CURRENT_TIMESTAMP) | Momento exato em que o cancelamento foi feito |
| `motivo` | VARCHAR(255) | Obrigatório (NOT NULL) | Motivo/justificativa informada |
| `responsavel` | VARCHAR(50) | Obrigatório (NOT NULL) | Agente que executou a ação (`Usuário` ou `Admin`) |

---

## 2. Relacionamentos e Cardinalidades

```
[ Usuário ] 1 ----------- 0..N [ Reserva ] 0..1 ----------- 1 [ Cancelamento ]
                                     |
                                   0..N
                                     |
                                     1
                                 [ Sala ]
                                     |
                                 (gerencia)
                                     |
                                   0..N
                               [ Administrador ]
```

| Entidade Origem | Entidade Destino | Cardinalidade | Descrição da Regra |
| :--- | :--- | :---: | :--- |
| **Usuário** | **Reserva** | `1 : N` | Um usuário pode realizar várias reservas; cada reserva pertence obrigatoriamente a um único usuário. |
| **Sala** | **Reserva** | `1 : N` | Uma sala pode receber diversas reservas (em horários não coincidentes); cada reserva é para uma sala específica. |
| **Reserva** | **Cancelamento** | `1 : 0..1` | Uma reserva cancelada possui exatamente um cancelamento correspondente; reservas normais não possuem registro nesta tabela. |
| **Administrador** | **Reserva** | `1 : N` | Um administrador pode aprovar ou recusar múltiplas solicitações de reserva. |
| **Administrador** | **Sala** | `N : N` | Administradores cadastram, atualizam e colocam salas em manutenção. |

---

## 3. Regras de Negócio e Validações

1. **Bloqueio de Choque de Horários:** Uma sala não pode ter mais de uma reserva com status `Aprovada` com sobreposição de horário na mesma data.
2. **Obrigatoriedade de Cadastro:** Apenas usuários ativos e devidamente cadastrados podem solicitar reservas.
3. **Limite de Lotação:** A quantidade de pessoas indicada na solicitação não pode exceder o valor de `capacidade_maxima` cadastrado na sala.
4. **Disponibilidade para Reserva:** Ambientes com `situacao = 'Em manutenção'` não podem receber novas solicitações de reserva.
5. **Ciclo de Vida e Auditoria:** Ao cancelar uma reserva aprovada ou pendente, seu status em `Reserva` migra para `Cancelada`, exigindo a criação do registro associado em `Cancelamento`.