---
name: lcc-usuarios-empresa
description: Pessoas da empresa — listar ou detalhar usuário por nome, e-mail, papel (borrower/lender/manager/owner) ou opt-in CERC ativo. Use quando o usuário pedir “pesquisa fulano”, “quem tem opt-in”, “lista da equipe”, “me traz o usuário do e-mail X” ou dados de uma pessoa da company. Busca da lista é nome/e-mail; CNPJ cadastral público é outra skill. Não invente dados que a base não devolver.
---

# Usuários de empresa

Pessoas da company no MCP `lcc-backend`. Sessão válida (skill `lcc-sessao`). Não invente nome, opt-in, score ou CNPJ.

## Escopo

| Nível | Visibilidade |
| --- | --- |
| `client` | só si (`obter_usuario_empresa` sem `user_id`) |
| `owner` | usuários da empresa + filiais |
| `superadmin` | todos |

## Listar (owner+)

`listar_usuarios_empresa` — filtros que **existem**:

- `search`: **nome ou e-mail** (contém). Não busca CNPJ nem telefone.
- `role`: `BORROWER` | `LENDER` | `MANAGER` | `OWNER`
- `company_scope`: slug (útil para staff)
- `has_kyc`: consentimento Open Finance autorizado (não é KYC cadastral)
- `has_cerc_opt_in`: opt-in CERC ativo e ainda válido
- `page`, `page_size`

Não existe filtro de data de cadastro. Cada item traz `created` — se o usuário pedir “quem entrou há dois meses”, pagine e filtre pelo `created` da resposta; não invente parâmetro.

`has_kyc` e `has_cerc_opt_in` são flags **distintas**.

## Detalhar

`obter_usuario_empresa(user_id?)`:

- Client omite `user_id`
- Owner passa `user_id` no escopo
- Staff **precisa** de `user_id`

No detalhe: telefone, CPF, PJ (`user_company` com CNPJ se existir), renda (`monthly_income` em **centavos** se vier). Score Serasa só se já persistido e o viewer for staff.

## Se pedirem CNPJ da pessoa

A lista **não** filtra por CNPJ. Caminhos:

- Ficha pública do CNPJ → `consultar_cnpj` (skill `lcc-consultas-publicas`)
- Company do LCC com aquele CNPJ → `listar_empresas(search=...)` e depois usuários com `company_scope`
- CNPJ do PJ do usuário só aparece em `obter_usuario_empresa` (campo `user_company`)

## Cruzamento

- Recebíveis: skill `lcc-recebiveis-antecipacao` (`user_id`)
- Extrato/Pix, atualizar conta ou buscar lançamentos que faltam: skill `lcc-open-finance` (`user_id`)
- Negócios da pessoa: `listar_deals(borrower_id=...)`

## Ordem típica

1. `quem_sou_eu`
2. `listar_usuarios_empresa` (nome/e-mail/opt-in/role) ou `obter_usuario_empresa`
3. Domínio relacionado com o `user_id` obtido

## O que não fazer

- Não tratar `has_kyc` como “fez KYC”.
- Não afirmar score Serasa consultado agora.
- Não listar usuários como `client` (`listar_usuarios_empresa` exige owner+).
- Não inventar filtro de CNPJ ou de data na listagem.
