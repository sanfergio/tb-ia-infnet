# Exercício 3 — Filosofia: Especificar Antes de Codificar

## 1. Filosofia: especificar antes de codificar

Especificar antes de codificar significa transformar a intenção em um modelo de análise verificável antes de pedir código à IA.

A especificação não é documentação decorativa nem detalhe técnico: ela descreve problema, atores, entradas, saídas, regras e critérios de aceite.

Isso funciona como modelo do sistema porque obriga o time a decidir ambiguidades e validar entendimento compartilhado; o prompt passa a ter matéria-prima estável.

No PDF da disciplina, a especificação é a matéria-prima do prompt e o protótipo é descartável; sem esse modelo, a IA preenche lacunas por conta própria.

Validar antes reduz retrabalho e evita que erros de requisito virem código.

## 2. Especificação de análise: notificar por e-mail quando o status de um ticket muda

**Problema:** Quando o status de um ticket muda, as pessoas interessadas precisam ser avisadas por e-mail para não descobrir a mudança só ao abrir o sistema.

**Atores:**
- Cliente solicitante.
- Agente responsável.
- Sistema de tickets.
- Serviço de e-mail.
- Administrador.

**Regras:**
- Notificar apenas em mudança efetiva de status.
- Não notificar se o status for repetido.
- Os destinatários devem ser o cliente e o agente responsável, se houver.
- A mensagem deve identificar o ticket, o status anterior e o novo.
- Não enviar dados sensíveis além do necessário.
- A falha no envio não deve desfazer a mudança de status, mas deve ser registrada para reenvio.
- Respeitar a preferência de notificação do cliente.
- Evitar duplicidade por mudança de status.

## 3. Por que validar antes do prompt?

Porque destinatários, gatilhos, exceções e falhas são decisões de negócio.

Sem validação, a IA pode notificar todos, duplicar e-mails, bloquear a mudança de status por erro de envio ou ignorar preferências.

Validar com o time antes do prompt evita retrabalho e alinha o critério de aceite.
