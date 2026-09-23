---
name: lcc-dashboard
description: Visão geral do negócio (dashboard) — volume no período, quantos negócios (deals) no total/mês/ativos, PJ vs PF, comissão, conversão, ticket médio, funil por categoria e quanto cada produto vendeu (fechados). Use quando o usuário pedir panorama, como está o mês, funil, comissão, mix de produtos, quantos clientes ou “como está o dashboard”. Não use para um negócio específico nem para Pix/extrato ou antecipação CERC. Valores em centavos.
---

# Dashboard e analytics

Indicadores **de deals**, não de unidades CERC nem de Pix. Sessão **owner+** (skill `lcc-sessao`).

`amounts` e `commission` estão em **centavos**. Não invente KPI que não veio no payload.

## Quando usar dashboard vs deals

| Pergunta | Tool |
| --- | --- |
| “Como está o funil / volume do mês?” | `obter_visao_dashboard` |
| “Qual produto vende mais?” | `obter_vendas_produtos` |
| “Status do deal X?” | `obter_deal` (skill `lcc-deals-propostas`) |

Dashboard consolida; não substitui detalhe de um deal específico.

## Visão geral

`obter_visao_dashboard`:

- `amounts.total_amount` / `month_amount`: soma de `deal.amount` (exclui categoria `DONE`)
- `deals`: total, mês, ativos, PJ (`business_credit`) vs PF, `by_category`
- `commission`: só deals `CLOSED` (total e mês)
- `conversion_rate`, `client_count` (borrowers)
- Filtros: `start_date` / `end_date` (criação do deal, `YYYY-MM-DD`), `company_scope` (staff)

`month_*` é o **mês calendário atual**, depois do filtro de datas — não um “mês” genérico do intervalo.

Owner: empresa + filiais. Superadmin: tudo, ou uma company via `company_scope`.

## Vendas por produto

`obter_vendas_produtos(company_scope?)`:

- Participação relativa de cada produto nos deals **CLOSED**
- `relative_percentage` sobre total de fechados no escopo

## Ordem típica

1. `quem_sou_eu` — confirme owner+
2. `obter_visao_dashboard` para KPIs gerais
3. `obter_vendas_produtos` se a pergunta for mix/composição por produto
4. `listar_deals` / `obter_deal` para investigar casos individuais

## O que não fazer

- Não usar dashboard como `client` (nível insuficiente).
- Não inventar KPIs fora do payload retornado.
- Não misturar com antecipação CERC (`lcc-recebiveis-antecipacao`) nem Pix/extrato (`lcc-open-finance`).
- Não tratar `total_amount` como valor livre de recebíveis.
