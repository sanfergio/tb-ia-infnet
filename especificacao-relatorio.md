# Exercício 5 — Especificação Estruturada: Relatório Semanal de Tickets

## 1. Objetivo

Enviar automaticamente, toda segunda-feira às 08:00, um relatório semanal de tickets por cliente para os gestores do Nimbus Desk.

## 2. Entradas

- **Período:** Semana anterior, de segunda-feira às 00:00 até domingo às 23:59.
- **Clientes:** Clientes ativos que possuem tickets no período.
- **Tickets:** Dados contendo status, cliente, agente responsável e datas de criação e resolução.

## 3. Saídas

- Relatório em formato HTML para envio por e-mail.
- Arquivo CSV anexado ao e-mail.
- Indicadores apresentados por cliente:
  - Total de tickets.
  - Quantidade de tickets abertos.
  - Quantidade de tickets pendentes.
  - Quantidade de tickets resolvidos.
  - Taxa de resolução.
  - Tempo médio de resolução.

## 4. Regras de negócio

- Agrupar os resultados por cliente, sem misturar dados entre clientes.
- Considerar apenas tickets criados ou atualizados no período.
- Clientes sem tickets no período não recebem relatório, mas devem aparecer no log com o registro "sem movimento".
- Falhas no envio devem ser registradas e submetidas a novas tentativas.
- O relatório não deve expor dados sensíveis além dos campos definidos.
- O acesso ao disparo manual deve ser restrito a administradores e gestores.

## 5. Critérios de aceite

- O job deve ser executado semanalmente no horário definido.
- O relatório deve conter todos os clientes com movimentação no período, e os totais devem corresponder aos dados do banco.
- Cada cliente deve visualizar apenas seus próprios dados.
- Falhas no envio devem gerar registros em log e novas tentativas.
- Devem existir testes para validar período, agregação, formatação e envio.

## 6. Decomposição com Prompting Estruturado

**Etapa 1 — Consulta de dados**

Prompt:

> A partir da especificação acima, liste apenas a consulta SQL/ORM para obter tickets do período por cliente. Não gere endpoint.

Resultado esperado: consulta com filtros de período e cliente.

**Etapa 2 — Agregação dos dados**

Prompt:

> Com base na consulta anterior, defina uma função de agregação por cliente com contagens e taxa de resolução. Entrada: lista de tickets; saída: dicionário.

Resultado esperado: função `agregar_por_cliente`.

**Etapa 3 — Formatação dos relatórios**

Prompt:

> Gere a formatação do relatório em HTML simples e CSV a partir da agregação. Não envie e-mail.

Resultado esperado: template HTML e gerador de CSV.

**Etapa 4 — Automação e envio**

Prompt:

> Gere o job semanal e o endpoint de disparo manual que orquestram consulta, agregação, formatação e envio, com log e tratamento de erro.

Resultado esperado: job com agendamento e endpoint com acesso restrito.

**Etapa 5 — Testes**

Prompt:

> Liste testes de aceite cobrindo as regras e os critérios de aceite.

Resultado esperado: casos de teste envolvendo período, agregação, isolamento entre clientes, falha de envio e formato dos relatórios.
