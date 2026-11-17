# Especificação do endpoint GET /tickets

## Especificação (5–8 linhas)
1. O endpoint GET /tickets exige cliente autenticado.
2. Retorna os tickets pertencentes ao cliente autenticado; por padrão, somente status “aberto”.
3. Aceita query param opcional `status`; se informado, valida contra “aberto”, “pendente” e “resolvido” e filtra por ele.
4. A resposta é uma lista JSON com id, titulo, status e criado_em.
5. A ordenação padrão é criado_em decrescente.
6. Sem autenticação, retornar 401; status inválido, retornar 400; sem tickets, retornar lista vazia.
7. Não expor tickets de outros clientes.

## Prompt usado
“Crie um endpoint Python com Flask chamado GET /tickets para o Nimbus Desk. Requer cliente autenticado; retorna tickets do cliente autenticado, por padrão status aberto; aceita filtro opcional por status, validando valores; resposta JSON com id, titulo, status e criado_em; ordenar por criado_em decrescente; erros 401 e 400.”

## Código gerado (resumo revisado)
```python
from flask import Blueprint, request, jsonify, g
from datetime import datetime

tickets_bp = Blueprint("tickets", __name__)
STATUS_VALIDOS = {"aberto", "pendente", "resolvido"}

@tickets_bp.get("/tickets")
@auth_required  # complementação manual: decorator real do projeto
def listar_tickets():
    status = request.args.get("status")
    if status is not None and status not in STATUS_VALIDOS:
        return jsonify({"erro": "status inválido"}), 400

    status_filtro = status or "aberto"
    tickets = (
        Ticket.query
        .filter_by(cliente_id=g.cliente_id, status=status_filtro)
        .order_by(Ticket.criado_em.desc())
        .all()
    )
    return jsonify([
        {
            "id": t.id,
            "titulo": t.titulo,
            "status": t.status,
            "criado_em": t.criado_em.isoformat()
        } for t in tickets
    ]), 200
