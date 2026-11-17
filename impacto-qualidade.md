# Exercício 4 — Impacto da Abordagem Spec-First na Qualidade

## 1. Versão Spec-First (resumo revisado)

```python
def notificar_mudanca_status(ticket, status_anterior, status_novo):
    if status_anterior == status_novo:
        return

    destinatarios = []

    if ticket.cliente and ticket.cliente.email and ticket.cliente.pref_notificacao:
        destinatarios.append(ticket.cliente.email)

    if ticket.agente and ticket.agente.email:
        destinatarios.append(ticket.agente.email)

    destinatarios = list(dict.fromkeys(destinatarios))

    if not destinatarios:
        registrar_log("sem_destinatario", ticket.id)
        return

    assunto = f"Ticket {ticket.id}: {status_anterior} -> {status_novo}"
    corpo = montar_corpo(ticket, status_anterior, status_novo)

    try:
        enviar_email(destinatarios, assunto, corpo)
        registrar_log("enviado", ticket.id, status_anterior, status_novo)

    except Exception as erro:
        registrar_log("falha_envio", ticket.id, str(erro))
        # Não reverte o status; agenda reenvio.
```

## 2. Versão Informal (instrução curta)

**Prompt utilizado:**

> Faz um código que avisa o cliente por e-mail quando o ticket mudar de status.

```python
def avisar(ticket):
    enviar_email(
        ticket.cliente.email,
        "Status mudou",
        f"Ticket {ticket.id} agora está {ticket.status}"
    )
```

## 3. Comparação

- **Tratamento de erros:** A versão Spec-First trata ausência de destinatário, duplicidade e falha de envio sem reverter o status. A versão informal não possui `try`, não registra falhas e pode comprometer a operação caso ocorra um erro no envio.
- **Cobertura das regras:** A versão Spec-First contempla cliente e agente, preferência de notificação, status anterior e novo, deduplicação e registro em log. A versão informal utiliza apenas o e-mail do cliente e o status atual.
- **Manutenção:** A versão Spec-First separa serviço, registro em log e formatação, facilitando testes e reenvios. A versão informal concentra tudo em uma única função e é mais frágil.
- **Revisão humana:** Ao revisar o código da IA, corrigi o ponto em que ela bloqueava a mudança de status quando o envio do e-mail falhava, pois a regra estabelecia que a falha deveria ser registrada, e não impedir a mudança.

## 4. Conclusão

A abordagem Spec-First produziu código de maior qualidade porque a especificação definiu destinatários, exceções, falhas e deduplicação antes da geração.

A versão informal funciona no caminho feliz, mas não é confiável para produção, pois não considera situações excepcionais nem contempla todas as regras de negócio necessárias.
