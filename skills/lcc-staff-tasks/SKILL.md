---
name: lcc-staff-tasks
description: Tarefas internas do time (staff) — criar tarefa para alguém, prazo, comentário, listar as minhas/abertas, editar, apagar ou comentar. Use quando o usuário pedir “cria uma task para fulano”, “tarefa até tal dia”, “o que está aberto”, ou acompanhar tarefa ligada a um deal. Só staff; SUPERVISOR não grava. Não use para deals/propostas do cliente.
---

# Staff Tasks (time LCC)

Tarefas internas. Sessão **superadmin** (skill `lcc-sessao`). `assignee` é e-mail de **StaffUser** ativo — não é borrower da company.

**SUPERVISOR:** só leitura. Create/update/delete devolve erro do middleware.

## Listar

`listar_tarefas_staff`:

- **Default:** abertas (`TODO` + `IN_PROCESS`). `DONE` não entra, salvo `status=todos` ou `status=DONE`
- `status`: `TODO` | `IN_PROCESS` | `DONE` | `todos`
- `priority`: `LOW` | `MEDIUM` | `HIGH`
- `assignee_email` **ou** `apenas_minhas` (não os dois)
- `deal_id`: ligadas àquele negócio
- `due_date_gte` / `due_date_lte` (`YYYY-MM-DD`)
- `search`: title ou description
- `page`, `page_size`

Soft-deleted não aparecem.

`obter_tarefa_staff(task_id)` — **sem** a thread. Comentários: `listar_comentarios_tarefa_staff`.

## Criar (“tarefa para fulano fazer X até tal dia”)

`criar_tarefa_staff`:

- Obrigatório: `title`, `assignee_email`
- Opcional: `description` (o “fazer isso, isso e isso”), `status` (default `TODO`), `priority` (default `MEDIUM`), `due_date`, `deal_id`

Não existe comentário inicial no create. Se o usuário pedir um comentário na hora:

1. `criar_tarefa_staff(...)`
2. `adicionar_comentario_tarefa_staff(task_id, comment)`

## Editar e apagar

- `atualizar_tarefa_staff` — parcial. `clear_due_date=True` / `clear_deal=True` limpam prazo ou deal.
- `excluir_tarefa_staff` — soft-delete, **sem restore** no MCP
- `adicionar_comentario_tarefa_staff` / `excluir_comentario_tarefa_staff` — comentário é hard-delete

## Ordem típica

1. `quem_sou_eu` — superadmin
2. Criar se pediram task nova; senão `listar_tarefas_staff`
3. Thread: `listar_comentarios_tarefa_staff`
4. Mutação só se o usuário pedir

## O que não fazer

- Não criar task para e-mail que não é staff.
- Não assumir que SUPERVISOR grava.
- Não tratar exclusão como arquivo recuperável.
- Não usar esta skill no lugar de deals/propostas do cliente.
