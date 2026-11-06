# Cronograma do Projeto SalaShow – Reserva de Salas (Grupo 02)

**Disciplina:** Requisitos, Modelagem e Análise de Dados | 2026/2 – Prof. Edson Vaz Lopes
**Integrantes:** Evellyn Priester, Anna Julia Soares, Selena Lory, Jullia Santini, Eduardo Barcelos

> As datas de 13/11, 20/11, 27/11 e 02/12 são do edital. O conteúdo de cada etapa (entidades, RN, tabelas) é uma **proposta** para o SalaShow: o grupo decide e justifica as escolhas.

## Cronograma

| Período | Etapa | O que o grupo faz no SalaShow | Entregável | Responsável |
|---|---|---|---|---|
| 06/11 (sex) | Kickoff | Criar o repositório GitHub com `README.md` descrevendo o cenário SalaShow (instituição, salas, recursos, solicitantes, reservas), a equipe e a navegação. Criar a pasta `04-dados/`. Primeiro commit. | Repositório + README | Evellyn Priester |
| 06/11 (sex) | Cronograma da equipe | Montar o cronograma do SalaShow com as etapas, datas do edital e responsáveis. | `CRONOGRAMA_EQUIPE.md` | Jullia Santini e Selena Lory |
| 06/11 (sex) | Inventário de dados | Listar os dados do cenário: **Sala** (código, nome, localização, capacidade), **Solicitante** (matrícula, nome, setor, telefone), **Recurso** (projetor, ar-condicionado, quadro branco), **Reserva** (data, horário de início e término). | `01_inventario_dados.md` | Eduardo Barcelos |
| 06/11 (sex) | Diagrama conceitual (DER) | Desenhar o DER com no mínimo 5 entidades, por exemplo Sala, Recurso, Solicitante, Reserva e Setor. **N:N Sala–Recurso** (uma sala tem vários recursos; um recurso está em várias salas). Sala 1:N Reserva e Solicitante 1:N Reserva. Definir: **atributo multivalorado** (ex.: telefone do solicitante), **atributo derivado** (ex.: duração da reserva = término − início) e **entidade fraca** (ex.: telefone, dependente de Solicitante). | `02_modelo_conceitual_v1` (PDF + editável) | Anna Julia Soares |
| 06/11 (sex) | Decisões de modelagem | Justificar por que Recurso é entidade e não campo de texto (RN-03), por que Reserva é entidade, e as escolhas de multivalorado, derivado e entidade fraca. | `03_decisoes_modelagem.md` | Evellyn Priester |
| 06/11 a 08/11 | Regras de negócio | Formalizar as RN da narrativa: **RN-01** a mesma sala não pode ter duas reservas com horários sobrepostos; **RN-02** cancelar uma reserva libera o horário; **RN-03** a busca de sala por recurso parte do cadastro de recursos (não de texto livre). Para cada RN, indicar se vira atributo, restrição de integridade ou cardinalidade. Sugestões extras: horário de término maior que o de início; reserva exige sala e solicitante cadastrados. | `00_catalogo_rn.md` | Ainda vai ser feito |
| 09/11 a 10/11 | Revisão do conceitual | Revisar o DER e as decisões de modelagem com o catálogo de RN pronto. Ajustar cardinalidades e justificativas. | `02_modelo_conceitual_v1` e `03_decisoes_modelagem.md` atualizados | Ainda vai ser feito  |
| 11/11 a 12/11 | Modelo lógico inicial | Converter para tabelas. Proposta com 7 tabelas: SALA, RECURSO, **SALA_RECURSO** (associativa do N:N), SOLICITANTE, SETOR, TELEFONE_SOLICITANTE e RESERVA. Definir PKs e FKs. | `04_modelo_logico_v1` (PDF + editável) | Ainda vai ser feito  |
| 11/11 a 12/11 | Ensaio da defesa | Montar os slides e ensaiar com cronômetro (máx. 15 min). Sugestão de divisão das falas entre os 5 integrantes: (1) cenário e RN; (2) entidades e relacionamentos; (3) N:N Sala–Recurso e tabela associativa; (4) multivalorado, derivado e entidade fraca; (5) modelo lógico e integridade. | Slides prontos | Ainda vai ser feito |
| **13/11 (sex), 8h** | **Defesa em grupo (30%)** | Apresentar o SalaShow em 15 min, com todos falando. Levar os slides em notebook ou pendrive. Assistir às outras 6 equipes. | Apresentação | Todos |
| 14/11 a 17/11 | Dicionário e integridade | Detalhar cada tabela (campo, tipo, PK/FK, obrigatoriedade). Documentar as regras de integridade: sem sobreposição de horários na mesma sala, término > início, FKs obrigatórias, cancelamento de reserva liberando o horário. Ajustar o modelo com o feedback da defesa. | `05_dicionario_dados_v1.md` e `06_regras_integridade.md` | Ainda vai ser feito |
| 18/11 a 19/11 | Revisão do repositório | Conferir os arquivos, o README e o histórico de commits (distribuídos desde 30/10). Cada um estuda as decisões do SalaShow. | Repositório pronto | Todos |
| **20/11 (sex, feriado)** | **Entrega do repositório (10%)** | Enviar o link do GitHub pelo Teams. O ideal é entregar até 19/11. | Link no Teams | Ainda vai ser feito  |
| 21/11 (sáb) | Auditoria do professor | Sem tarefa para o grupo. | – | – |
| 22/11 a 26/11 | Estudo para a defesa individual | Cada integrante deve saber justificar qualquer decisão do SalaShow (por que Recurso é entidade, como funciona a tabela associativa, como evitar reservas sobrepostas). | – | Todos |
| **27/11 (sex)** | **Defesa individual (30%)** | Prova escrita, com 2 perguntas nominais sobre o projeto. | – | Cada um |
| 28/11 a 01/12 | Relatório final | Seções: introdução; o projeto SalaShow (síntese do conceitual e do lógico); análise individual das outras 6 equipes (TaskFlow, MesaFácil, CineClub, EvenTicket, VetCare, OficinaPro), com síntese comparando pelo menos 3 (por exemplo, SalaShow × EvenTicket, que têm regra de capacidade/horário parecida); retrospectiva do semestre; considerações finais. Apêndice de autoria e referências. | Rascunho completo |Todos|
| **02/12 (qua)** | **Entrega do relatório (20%)** | PDF único pelo Teams: `Projeto_U3_SALASHOW_v1.0.pdf`. | PDF no Teams | Todos |
| 04/12 (sex) | Prova substitutiva | Independente do projeto. | – | – |
