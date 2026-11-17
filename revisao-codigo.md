# Exercício 8 — Revisão de Código: PATCH /tickets/{id}/status

## 1. Código recebido

```python
@app.route("/tickets/<int:ticket_id>/status", methods=["PATCH"])
def update_status(ticket_id):
    novo_status = request.json.get("status")
    ticket = Ticket.query.get(ticket_id)
    ticket.status = novo_status
    db.session.commit()
    return jsonify(ticket.to_dict())
```

## 2. Lacunas em relação à especificação combinada

A especificação prevê apenas as transições `aberto → em_andamento` e `em_andamento → resolvido`, recusando qualquer outra com **HTTP 400 e mensagem explicativa**.

O código recebido apresenta as seguintes lacunas:

1. **Não valida a transição:** aceita qualquer valor, inclusive `resolvido → aberto`, `aberto → resolvido` e status inexistentes.
2. **Não retorna 400:** qualquer transição inválida vira atualização silenciosa com HTTP 200.
3. **Não retorna mensagem explicativa:** o esperado é um JSON com o motivo da recusa.
4. **Não trata ticket inexistente:** `Ticket.query.get(ticket_id)` retorna `None` e o código quebra com `AttributeError` (HTTP 500), quando deveria retornar 404.
5. **Não valida `request.json`:** se o corpo não vier ou não tiver `status`, `novo_status` fica `None` e o ticket é salvo com status nulo.
6. **Não valida o domínio de status:** aceita qualquer string, mesmo fora de `{aberto, em_andamento, resolvido}`.
7. **Não verifica autenticação nem dono do ticket:** qualquer cliente pode mudar o status de um ticket alheio (IDOR).
8. **Não trata erro de commit:** falha de banco sobe como HTTP 500 sem rollback explícito.
9. **Não registra auditoria:** não grava quem mudou, quando e de qual status para qual.
10. **Retorna o ticket inteiro:** a especificação não define o payload de retorno; assumir `to_dict()` completo pode expor campos indevidos.

## 3. Código corrigido

```python
from datetime import datetime
from flask import request, jsonify, g
from sqlalchemy.exc import SQLAlchemyError

TRANSICOES_PERMITIDAS = {
    "aberto": "em_andamento",
    "em_andamento": "resolvido",
}

STATUS_VALIDOS = {"aberto", "em_andamento", "resolvido"}


@app.patch("/tickets/<int:ticket_id>/status")
@auth_required
def update_status(ticket_id):
    # 5. valida corpo da requisição
    payload = request.get_json(silent=True) or {}
    novo_status = payload.get("status")

    if not novo_status or not isinstance(novo_status, str):
        return jsonify({
            "erro": "campo 'status' é obrigatório e deve ser string"
        }), 400

    # 6. valida domínio de status
    if novo_status not in STATUS_VALIDOS:
        return jsonify({"erro": f"status inválido: {novo_status}"}), 400

    # 7 + 4. busca com escopo de cliente
    # inexistente ou de outro cliente -> 404
    ticket = Ticket.query.filter_by(
        id=ticket_id,
        cliente_id=g.cliente_id
    ).first()

    if ticket is None:
        return jsonify({"erro": "ticket não encontrado"}), 404

    status_atual = ticket.status

    # 1 + 2 + 3. valida transição permitida e responde 400 explicativo
    if TRANSICOES_PERMITIDAS.get(status_atual) != novo_status:
        return jsonify({
            "erro": "transição de status não permitida",
            "de": status_atual,
            "para": novo_status,
            "transicoes_permitidas": {
                "aberto": "em_andamento",
                "em_andamento": "resolvido",
            },
        }), 400

    # atualização + auditoria
    try:
        ticket.status = novo_status
        ticket.atualizado_em = datetime.utcnow()
        ticket.status_alterado_por = g.cliente_id

        db.session.commit()

    except SQLAlchemyError:
        db.session.rollback()
        return jsonify({"erro": "falha ao atualizar status"}), 500

    # 10. retorno enxuto, conforme escopo do endpoint
    return jsonify({
        "id": ticket.id,
        "status": ticket.status,
        "atualizado_em": ticket.atualizado_em.isoformat(),
    }), 200
```

## 4. Verificação: lacuna → correção

- **Transição inválida:** `TRANSICOES_PERMITIDAS` + retorno HTTP 400.
- **Status fora do domínio:** `STATUS_VALIDOS` + HTTP 400.
- **Corpo ausente ou inválido:** validação de `payload` + HTTP 400.
- **Ticket inexistente:** `filter_by(...).first()` + HTTP 404.
- **IDOR:** consulta com `cliente_id=g.cliente_id`.
- **Falha de banco:** `try/except` + `rollback`.
- **Auditoria:** `atualizado_em` e `status_alterado_por`.
- **Payload exposto:** retorno enxuto com `id`, `status` e `atualizado_em`.

## 5. Observação de revisão

O código original é o exemplo clássico do PDF: o assistente cobre o caminho feliz e deixa as regras críticas de fora.

A revisão humana aqui não é opcional: o endpoint mexe no ciclo de vida de tickets e no isolamento entre clientes.

Sem essas correções, qualquer cliente poderia reabrir um ticket alheio com um PATCH.
