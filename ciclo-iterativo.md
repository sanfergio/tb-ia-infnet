# Exercício 7 — Ciclo Iterativo do Relatório Semanal

## 1. Iteração 1 — Planejar, Gerar e Revisar

- **Planejamento:** Pedi ao assistente: "Gere o relatório semanal completo", utilizando a especificação como referência.
- **Geração:** O código consultou todos os tickets, agrupou os resultados por cliente e enviou e-mails para todos os clientes ativos.
- **Revisão:** Encontrei três problemas:
  - Não filtrava corretamente os tickets da semana anterior.
  - Enviava relatórios para clientes sem movimentação no período.
  - Não registrava falhas de envio nem tentava realizar o reenvio.
- **Correção:** Ajustei a especificação para explicitar o período, os clientes com movimentação e a política de falhas. Em seguida, refiz o prompt, dividindo a implementação em etapas.

## 2. Iteração 2 — Planejar, Gerar e Revisar

- **Planejamento:** Utilizei prompts separados para consulta, agregação, formatação e execução do job.
- **Geração:** O código passou a filtrar corretamente o período, excluir clientes sem tickets do envio, registrar falhas e agendar novas tentativas.
- **Revisão:** Validei os totais utilizando dados de exemplo e confirmei o isolamento das informações entre clientes.
- **Mudança principal:** A segunda rodada foi necessária porque a primeira implementação resolvia apenas o caminho feliz, mas não contemplava todas as regras de negócio e situações excepcionais.

## 3. Diferença entre o Ciclo Iterativo e Aceitar o Primeiro Código

Aceitar o primeiro código sem revisão seria considerar um protótipo que atende parcialmente aos requisitos como se fosse uma solução pronta para produção.

O ciclo iterativo transforma a saída da IA em código avaliado por meio da comparação com a especificação, identificação de lacunas, correção dos prompts e ajustes nos requisitos.

A revisão não é uma etapa opcional, mas sim o controle humano que impede que suposições da IA se tornem regras definitivas dentro do sistema.
