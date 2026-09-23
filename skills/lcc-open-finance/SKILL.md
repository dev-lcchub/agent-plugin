---
name: lcc-open-finance
description: Movimentação bancária da conta — Pix, TED, créditos, débitos, extrato, resumo, atualizar dados e buscar lançamentos que faltam. Use quando o usuário perguntar quanto fez de Pix, quanto pagou ou recebeu, gastos do mês, extrato, lançamentos, ou pedir para atualizar/sincronizar a conta, puxar um período passado ou completar dias que não aparecem. A pessoa em geral não fala “Open Finance”. Não use para recebíveis CERC/antecipação nem para volume de deals. Valores em centavos.
---

# Movimentação da conta (Open Finance)

Pix, TED, créditos e débitos. A pessoa em geral **não** fala “Open Finance”. Sessão válida (skill `lcc-sessao`). Valores em **centavos**.

Não invente lançamento. Se a conta não tiver Open Finance vinculado, as tools de leitura devolvem lista vazia + mensagem — não é erro de auth.

## Qual tool usar

| O usuário pede | Tool | O que acontece |
| --- | --- | --- |
| Extrato/resumo **já no banco** | `listar_lancamentos_conta` / `obter_resumos_transacoes` | Só leitura |
| “Atualiza a conta”, sync, dados frescos, primeira sincronização | `solicitar_atualizacao_transacoes` | Sync incremental + recálculo de resumo; move `last_synced_at` |
| “Faltam Pix do dia X”, lacuna, rebuscar um período passado | `solicitar_backfill_transacoes` | Insere só o intervalo; **não** move cursor nem resumo |

Não use backfill como primeira sync. Não use refresh para “puxar só o dia 12”.

## Pré-requisito

Confirme contas/flags com `obter_usuario_empresa` (`has_kyc` = consentimento OF autorizado — não é KYC cadastral).

## Escopo de `user_id`

Leitura (`listar` / `obter_resumos`): `client` omite; `owner` passa o borrower; `superadmin` **precisa** de `user_id`.

Escrita (refresh / backfill): **só `CompanyUser`**. Staff não enfileira — a tool devolve erro. Se a sessão for staff, explique isso; não tente de novo com outro `user_id`.

| Nível | Refresh | Backfill |
| --- | --- | --- |
| `client` | omitir `user_id` (atualiza a si) | omitir `user_id` |
| `owner` | omitir = **todos** os membros ativos; ou um `user_id` | **precisa** do `user_id` do borrower (não faz batch) |
| `superadmin` | não enfileira | não enfileira |

## Resumo por conta

`obter_resumos_transacoes(user_id?)` — summary mais recente de cada conta (fluxo de caixa, tickets, comparativo com o mês anterior). Use para panorama. Depois de um **backfill**, este resumo **não muda** — use a lista de lançamentos.

## Lançamentos detalhados

`listar_lancamentos_conta` — “quanto fiz de Pix no mês”, extrato, despesas:

- Sem datas: **últimos 30 dias**. Passe `start_date`/`end_date` (`YYYY-MM-DD`) para o período pedido.
- `account_id`: uma conta do alvo
- `direction`: `CREDITO` | `DEBITO`
- `classification`: valor exato (ex. `PIX`, `TED`)
- `search`: parcial no `name`
- `ordering`: `occurred_at` | `amount` | `created` (prefixo `-` para desc)
- `aggregate` / `by_account` cobrem o **filtro inteiro**, não só a página

## Atualização oficial (sync + resumo)

`solicitar_atualizacao_transacoes(user_id?)`:

- Equivale ao POST `/summaries/`. Sem parâmetros de data.
- Resposta assíncrona (`status: processing`). Aguarde `retry_after` segundos (cooldown ~60s) e chame `obter_resumos_transacoes`.
- Se já houver refresh em andamento, **não** re-enfileira — só devolve o `retry_after`.
- Cooldown de backfill **não** bloqueia refresh.

## Backfill histórico

`solicitar_backfill_transacoes(from_date, to_date, user_id?, account_id?)`:

- Intervalo explícito (`YYYY-MM-DD`). `to_date` não pode ser futuro. `from_date` no máximo **365 dias** atrás. `from_date` ≤ `to_date`.
- Insere só lançamentos novos. **Não** altera `last_synced_at` nem `TransactionSummary`.
- A conta precisa ter sido sincronizada pelo menos uma vez. Sem isso: peça refresh primeiro.
- `account_id` restringe a uma conta do alvo.
- Depois de `retry_after`, consulte `listar_lancamentos_conta` no **mesmo intervalo**.
- Cooldown de refresh **não** bloqueia backfill.

## Ordem típica

1. `quem_sou_eu`
2. `obter_usuario_empresa` se precisar confirmar contas/flags
3. Dados frescos: `solicitar_atualizacao_transacoes` → esperar → `obter_resumos_transacoes`
4. Dia/período que falta: `solicitar_backfill_transacoes` → esperar → `listar_lancamentos_conta`
5. Panorama: `obter_resumos_transacoes`. Detalhe: `listar_lancamentos_conta`

## O que não fazer

- Não confundir com recebíveis CERC (skill `lcc-recebiveis-antecipacao`).
- Não assumir período customizado sem passar datas em `listar_lancamentos_conta` — o default é 30 dias.
- Não interpretar `aggregate` como só a página atual.
- Não usar backfill como primeira sync.
- Não esperar mudança em `obter_resumos_transacoes` após backfill.
- Não enfileirar refresh/backfill como staff.
- Não omitir `user_id` no backfill quando a sessão for owner.
