---
name: lcc-deals-propostas
description: Listar ou detalhar negócios (deals) e propostas comerciais — um caso específico ou vários com filtro (status, produto, período, pessoa, empresa). Use quando o usuário pedir um deal, uma proposta, histórico do negócio, comentar no deal, catálogo de produtos, “meus negócios”, negócios do mês ou propostas pré-aprovadas/aceitas. Não use para funil agregado (dashboard) nem para tarefas internas do staff. Valores em centavos.
---

# Deals e propostas comerciais

Playbook para negócios (`deals`) e propostas no MCP `lcc-backend`. Sessão válida primeiro (skill `lcc-sessao`).

`amount` do deal e da proposta está em **centavos**. `rate` da proposta é **fração** (ex.: `0.03` = 3%), não centavos.

Só use o que as tools devolverem. Não invente status, valor, score Serasa ou proposta.

## Escopo

| Quem | Deals | Propostas |
| --- | --- | --- |
| Borrower (`client`) | só os próprios | só as dos próprios deals |
| Lender (`client`) | empresa + filiais (igual owner) | só as em que ele é o `lender` |
| Owner / Manager | empresa + filiais | deals da empresa + filiais |
| Superadmin | todos | todas |

Confirme com `quem_sou_eu` antes de filtrar.

## Listar deals

`listar_deals` — todos os filtros são opcionais:

- `status_category`: `NOT_STARTED` | `ACTIVE` | `DONE` | `CLOSED`
- `product_id`, `company_scope`, `borrower_id`
- `search`: nome ou telefone do **borrower** (não busca CNPJ)
- `start_date` / `end_date`: data de **criação** do deal (`YYYY-MM-DD`)
- `page`, `page_size`

Resposta: `total_amount` (soma do filtro, centavos) + `items`. Score Serasa só se já estiver persistido e o viewer for staff — a tool **não** consulta Serasa na hora.

## Detalhe e histórico

- `obter_deal(deal_id)` — status, respostas do formulário, responsável, `proposals_count`.
- `obter_historico_deal(deal_id)` — timeline (status, comentários, updates). Comentários `is_public=False` só para staff.

## Comentário

`adicionar_comentario_deal(deal_id, comment, is_public=True)`.

- Default público.
- Só **staff** pode `is_public=False`.

## Propostas

`listar_propostas_negocio`:

- `status`: `PRE_APPROVED` | `APPROVED` | `ACCEPTED` | `REJECTED` | `EXPIRED`
- `deal_id`: propostas daquele negócio
- `page`, `page_size`

`obter_proposta_negocio(proposal_id)` — valor (centavos), taxa (fração), prazo, garantia, banco, lender.

Para “propostas daquele deal”: `listar_propostas_negocio(deal_id=...)`.

## Produtos

`listar_produtos` / `obter_produto(product_id)` — use antes de filtrar deals por produto. Borrower só vê produtos ofertados à empresa.

Funil/volume agregado: skill `lcc-dashboard`. Tarefa interna do time: skill `lcc-staff-tasks`.

## Ordem típica

1. `quem_sou_eu`
2. `listar_produtos` se a pergunta for por produto
3. `listar_deals` (com os filtros que o usuário pediu) → `deal_id`
4. `obter_deal` e/ou `obter_historico_deal` para um caso
5. `listar_propostas_negocio` / `obter_proposta_negocio` se a pergunta for a proposta
6. Comentário só se o usuário pedir para registrar

## O que não fazer

- Não inventar status, scores, valores ou propostas fora do retorno.
- Não tratar lender como “só os próprios deals” — lender vê os deals da empresa.
- Não expor comentário interno a quem não é staff.
- Não usar dashboard no lugar de `obter_deal` para um negócio específico.
