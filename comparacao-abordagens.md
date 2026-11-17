# Exercício 2 — Comparação: Vibe Coding x Spec-First

## Comparação de abordagens no PATCH /tickets/{id}/resolver

### 1. Prompt Vibe Coding

**Prompt utilizado:**

> Cria um endpoint para resolver um ticket.

### 2. Código típico gerado (Vibe Coding)

```python
@app.patch("/tickets/<int:id>/resolver")
def resolver(id):
    ticket = Ticket.query.get(id)
    ticket.status = "resolvido"
    db.session.commit()
    return {"id": ticket.id, "status": ticket.status}
```

### 3. Especificação estruturada utilizada no Spec-First

- **Entrada:** PATCH /tickets/{id}/resolver, cliente autenticado.
- **Regra:** Só permitir a resolução de tickets pertencentes ao próprio cliente. Tickets inexistentes ou pertencentes a outro cliente devem retornar HTTP 404.
- **Regra:** Só permitir a resolução quando o status atual for "aberto" ou "pendente". Caso já esteja "resolvido", retornar HTTP 409.
- **Saída:** JSON do ticket atualizado, contendo `id`, `status` e `resolvido_em`.
- **Erros:** HTTP 401 para ausência de autenticação, HTTP 404 para ticket não encontrado e HTTP 409 para conflito de status.
- **Segurança:** Não expor dados de outros clientes.

### 4. Código Spec-First (resumo revisado)

```python
@app.patch("/tickets/<int:id>/resolver")
@auth_required
def resolver_ticket(id):
    ticket = Ticket.query.filter_by(
        id=id,
        cliente_id=g.cliente_id
    ).first()

    if not ticket:
        return jsonify({"erro": "ticket não encontrado"}), 404

    if ticket.status == "resolvido":
        return jsonify({"erro": "ticket já resolvido"}), 409

    if ticket.status not in ("aberto", "pendente"):
        return jsonify({"erro": "status não permite resolução"}), 409

    ticket.status = "resolvido"
    ticket.resolvido_em = datetime.utcnow()

    db.session.commit()

    return jsonify(ticket.to_dict()), 200
```

### 5. Diferenças concretas

- **Casos de erro:** O Vibe Coding não trata os erros 404, 401 e 409, nem verifica se o ticket pertence a outro cliente. O Spec-First cobre esses cenários.
- **Aderência às regras:** O Vibe Coding permite resolver qualquer ticket, inclusive de outro cliente ou que já esteja resolvido. O Spec-First valida a propriedade e o status antes de executar a operação.
- **Organização e manutenção:** O Vibe Coding concentra toda a lógica no endpoint e utiliza uma consulta sem escopo. O Spec-First utiliza uma consulta filtrada, validações explícitas e serialização da resposta.
- **Segurança:** O Vibe Coding permite uma vulnerabilidade do tipo IDOR (Insecure Direct Object Reference). O Spec-First restringe o acesso por meio do `cliente_id`.
- **Auditoria:** O Spec-First registra a data e hora da resolução em `resolvido_em`, enquanto o Vibe Coding não realiza esse registro.

### 6. Conclusão

O Vibe Coding é aceitável para situações como protótipos descartáveis, demonstrações internas, testes de conceito ou desenvolvimento de funcionalidades em que o custo de um possível erro é baixo.

Já o Spec-First é necessário quando o sistema envolve autenticação, dados de clientes, regras de negócio, auditoria, integrações ou funcionalidades destinadas à produção.

No **Nimbus Desk**, o endpoint PATCH altera o status dos tickets e envolve o isolamento de dados entre clientes. Portanto, exige uma especificação estruturada antes da implementação, garantindo maior segurança, confiabilidade e aderência às regras de negócio.
