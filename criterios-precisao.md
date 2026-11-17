# Exercício 9 — Critérios de Precisão de Especificação para Geração de Código Confiável

## 1. Comparação: Spec de Análise (Exercício 3) x Spec Estruturada (Exercício 5)

A **spec de análise** do Exercício 3 descreve problema, atores e regras de negócio — é um modelo conceitual, não uma especificação de implementação. Ela responde "o que o sistema deve fazer e por quê", mas deixa em aberto formato de mensagem, gatilhos técnicos, contrato de dados e critérios de aceite.

Por isso, sozinha, **não é suficiente** para gerar código confiável sem intervenção manual significativa: o assistente preenche as lacunas com suposições (destinatários, formato, tratamento de falha), e o revisor precisa decidir tudo isso depois.

A **spec estruturada** do Exercício 5 (objetivo, entradas, saídas, regras e critérios de aceite) é mais precisa: define contrato de dados, período, formato de saída, política de falha e critérios testáveis.

Ela é **suficientemente precisa para gerar código próximo do aceitável**, mas ainda exige revisão humana em pontos que a spec não fixa — fuso horário, biblioteca de envio, autenticação do disparo manual e formato exato do CSV.

Ou seja: a spec estruturada reduz drasticamente a intervenção manual, mas não elimina a revisão; a spec de análise, sem complementação, exige intervenção pesada.

**Conclusão prática:** a spec de análise é o **modelo de análise** que valida o entendimento com o time; a spec estruturada é o **contrato de geração**. Para produção, é a segunda que deve ser usada como entrada do prompt, com a primeira anexada como contexto de negócio.

## 2. Critérios objetivos de precisão

### 1. Casos de erro explícitos com código e mensagem

Sem isso, o modelo só cobre o caminho feliz.

Exemplo: no Exercício 8, a spec combinada dizia "recusar transição inválida com 400 e mensagem explicativa", mas o código gerado não tinha nada disso — a lacuna ficou visível justamente porque a spec fixava o contrato de erro.

Na spec de análise do Exercício 3, "falha no envio não deve desfazer a mudança de status" já apontava a necessidade de tratamento, mas não definia código nem mensagem; isso exigiu complementação manual no Exercício 4.

### 2. Definição de tipos, formatos e contratos de dados

A spec precisa dizer o que entra e o que sai, com tipo e formato.

Exemplo: no Exercício 5, "saída em HTML e CSV anexo" e "campos id, status, atualizado_em" evitaram que o assistente inventasse payload.

Na spec de análise do Exercício 3, não havia contrato de dados — o assistente poderia gerar e-mail com campos diferentes dos que o time esperava. Sem esse critério, o código compila, mas a integração quebra.

### 3. Regras de negócio explícitas, testáveis e sem ambiguidade

Regra vaga gera interpretação livre.

Exemplo: no Exercício 5, "cliente sem tickets no período não recebe relatório, mas aparece no log como 'sem movimento'" é testável.

Já "notificar as pessoas interessadas" (Exercício 3) é ambíguo: quem são os interessados? O assistente decidiu notificar só o cliente na versão informal do Exercício 4 e errou.

A spec precisa nomear atores e condições.

### 4. Critérios de aceite verificáveis

A spec deve ter itens que possam virar teste automatizado.

Exemplo: no Exercício 5, "totais conferem com o banco", "cliente não vê dados de outro cliente" e "falha de envio gera log e nova tentativa" viraram casos de teste.

A spec de análise do Exercício 3 não tinha critérios de aceite, então não havia como medir se o código gerado estava correto — só se "parecia funcionar".

O ciclo iterativo do Exercício 7 mostrou que, sem critério, a primeira versão parecia OK e violava três regras.

### 5. Fronteiras de escopo (o que está fora)

Tão importante quanto dizer o que entra é dizer o que não entra.

Exemplo: no Exercício 5, "acesso ao disparo manual restrito a administrador/gestor" foi o ponto que o assistente inseriu por conta própria no Exercício 6 — sinal de que a spec não tinha fronteira de acesso explícita.

Definir escopo evita que o modelo invente autenticação, permissão ou integração que o time não pediu.

### 6. Não-funcionais críticos quando afetam o contrato

Segurança, isolamento entre clientes e idempotência precisam estar na spec quando mudam o código.

Exemplo: no Exercício 8, a ausência de "isolar por cliente" na spec (ou a não leitura dela pelo assistente) produziu um endpoint com IDOR.

No Exercício 5, "não misturar dados entre clientes" foi explicitado e virou teste.

Sem esse critério, o código gerado pode ser funcionalmente correto e inseguro ao mesmo tempo.

### 7. Rastreabilidade spec → tarefa → teste

Cada tarefa de implementação e cada teste deve poder ser ligado a uma seção da spec.

Exemplo: no Exercício 6, a auditoria da decomposição mostrou que "autenticação do endpoint de disparo" não tinha seção correspondente — sinal de que a spec estava incompleta ou de que o modelo inventou.

Esse critério é o que permite revisar a decomposição antes de distribuí-la ao time, como o líder pediu.

## 3. Síntese

Uma especificação é "precisa o suficiente" para gerar código confiável quando:

- Cobre erros com contrato.
- Define tipos e formatos.
- Tem regras testáveis.
- Possui critérios de aceite.
- Delimita o escopo.
- Inclui requisitos não funcionais críticos.
- Permite rastrear cada tarefa a uma seção da especificação.

A spec de análise do Exercício 3 atende parcialmente (regras de negócio) e falha nos demais critérios; a spec estruturada do Exercício 5 atende quase todos e por isso gerou código próximo do aceitável.

Na prática da Nimbus, o padrão mínimo para gerar com IA sem revisão pesada é a spec estruturada — e a revisão humana continua obrigatória nos pontos que a especificação não fixa.
