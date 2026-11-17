# Exercício 6 — Análise da Decomposição

## 1. Tarefas com Correspondência Direta na Especificação

- **Consulta ao banco:** Decorre das seções "Entradas" e "Regras de negócio".
- **Agregação por cliente:** Decorre das seções "Saídas" e "Regras de negócio".
- **Formatação HTML/CSV:** Decorre da seção "Saídas".
- **Job semanal:** Decorre das seções "Objetivo" e "Regras de negócio".
- **Testes:** Decorrem dos "Critérios de aceite".
- **Log e reenvio:** Decorrem da regra que determina que falhas de envio devem ser registradas e submetidas a novas tentativas.

## 2. Tarefa Inserida sem Correspondência Clara

**Autenticação e autorização do endpoint de disparo manual:**

O assistente incluiu a restrição de que apenas administradores poderiam realizar o disparo manual, mas a especificação original não apresentava uma seção específica sobre controle de acesso.

Essa funcionalidade é necessária para a segurança do sistema, mas inicialmente foi uma suposição do modelo, e não um requisito explicitamente definido.

## 3. Seção Reescrita para Cobrir a Lacuna

### Acesso e Segurança

- O disparo manual do relatório deve ser restrito a administradores ou gestores autenticados.
- O job automático deve utilizar uma conta de serviço.
- Nenhum cliente pode consultar relatórios de outros clientes.
- Toda tentativa de acesso negado deve ser registrada.

## 4. Conclusão

Com a inclusão da seção de Acesso e Segurança, a tarefa de autenticação e autorização deixa de ser uma suposição da IA e passa a decorrer diretamente da especificação.

Isso demonstra a importância de revisar a decomposição gerada pelo modelo e identificar requisitos implícitos ou ausentes antes de iniciar a implementação.
