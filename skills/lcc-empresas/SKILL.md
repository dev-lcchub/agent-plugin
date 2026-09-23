---
name: lcc-empresas
description: Dados da company/empresa que já estão no LCC — nome, CNPJ, status, filiais, taxas e limites cadastrados. Use quando o usuário pedir informações da empresa dele, de outra company do escopo (staff), listar empresas ou “qual o status da company”. Só o que existir na base; não invente. Não use para ficha da Receita (CNAE, sócios) — isso é consulta de CNPJ público.
---

# Empresas

Dados da **company no LCC**, não a ficha da Receita. Sessão válida (skill `lcc-sessao`).

Não invente CNPJ, status, taxa ou filial. Se o campo não veio, diga que não está na base.

## Nível

- `listar_empresas` / `obter_empresa`: **owner** ou **superadmin**
- `client` não lista empresas — `quem_sou_eu` já traz a company da sessão

## O que é dinheiro e o que não é

- Taxas (`commission`, `contract_interest_rate`, `recebify_fee`, `card_interest_rate`): **fração** (ex.: `0.05` = 5%). Não são centavos.
- `max_users` / `extra_seats`: inteiros (limites de assento), não dinheiro.

## Listar

`listar_empresas`:

- `search`: nome, razão social ou CNPJ (contém)
- `status`: `PENDING` | `APPROVED` | `REJECTED`
- `page`, `page_size`

Owner: própria empresa + filiais. Superadmin: todas.

## Detalhar

`obter_empresa`:

- `company_id` ou `company_scope` (slug)
- Sem os dois: empresa da sessão (só `CompanyUser`)
- Staff **precisa** de `company_id` ou `company_scope`

No detalhe: taxas (fração), limites de usuários, `branch_ids`, domínio, motivo de rejeição se houver.

## Ordem típica

1. `quem_sou_eu`
2. `listar_empresas` ou `obter_empresa`
3. CNAE, sócios, endereço fiscal → skill `lcc-consultas-publicas` (`consultar_cnpj`)

## O que não fazer

- Não completar ficha cadastral com chute — só o payload.
- Não converter taxa para centavos.
- Não usar esta skill no lugar de `consultar_cnpj`.
