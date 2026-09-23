---
name: lcc-recebiveis-antecipacao
description: Recebíveis de cartão (CERC) e simulação de antecipação — valor livre, unidades, bloqueados, resumo da empresa e quanto daria antecipar com uma taxa. Use quando o usuário falar antecipação, recebíveis, valor disponível, agenda, AP005 ou “simula antecipar”. Não use para Pix/extrato bancário nem para volume de deals do dashboard. Valores em centavos; taxa da simulação é fração (0.03 = 3% a.m.).
---

# Recebíveis e antecipação

Playbook para tools de recebíveis CERC no MCP `lcc-backend`. Exige sessão válida (veja skill `lcc-sessao`). Valores em **centavos**.

## Pré-requisitos

1. `quem_sou_eu` — confirme nível e empresa.
2. `calcular_antecipacao` e resumos agregados de empresa exigem **owner** ou **superadmin**. Cliente (`client`) lista/resume os próprios recebíveis, mas não simula antecipação.

## Consultar recebíveis de um usuário

### Resumo rápido

- `obter_resumo_recebiveis` — valor livre, projeção, evolução.
- `obter_historico_recebiveis_livres` — série mensal de valor livre.

`client` omite `user_id` (consulta a si). `owner` passa `user_id` de borrower no escopo. `superadmin` passa qualquer `user_id`.

### Lista detalhada (AP005)

Use `listar_recebiveis`:

- Default: último `snapshot_date` + modo `livres` (`free_amount > 0`, constituídos).
- `modo`: `livres` | `bloqueados` | `todos`.
- Filtros úteis: `start_date`/`end_date` (liquidation), `somente_precontratados`, `constitution_type` (1=CONSTITUTED, 2=TO_CONSTITUTE).
- `incluir_pagamentos=true` só quando precisar dos payments; default é mais leve.
- Resposta traz `aggregate` do filtro inteiro + página em `items`. IDs em `items[].id` alimentam a simulação.

Só leitura ORM — **não** dispara sync CERC.

## Visão por empresa (owner+)

- `obter_resumo_recebiveis_empresa` — agregado borrowers da empresa (+ filiais).
- `listar_resumos_recebiveis` — um resumo por borrower, ordenado por disponível; `search` por nome.

## Simular antecipação (owner+)

`calcular_antecipacao` **não persiste** nada; espelha `calculate-antecipation` da API.

1. Liste/filtre com `listar_recebiveis` (modo `livres` costuma ser o ponto de partida).
2. Colete os IDs AP005 desejados.
3. Chame:

```text
calcular_antecipacao(
  receivables_ids=[...],
  tax_per_month=0.03   # 3% a.m.; ajuste conforme o caso
)
```

4. Explique o resultado em BRL (converter centavos só na apresentação).

Se a tool disser que IDs estão fora do escopo, volte à listagem no nível/empresa corretos (`trocar_empresa` se preciso).

## Ordem típica

1. `quem_sou_eu`
2. Resumo (`obter_resumo_recebiveis` ou `obter_resumo_recebiveis_empresa`)
3. `listar_recebiveis` para obter IDs
4. `calcular_antecipacao` (se owner+)

## O que não fazer

- Não inventar IDs de recebíveis.
- Não chamar `calcular_antecipacao` como `client`.
- Não tratar `tax_per_month` como percentual inteiro (use fração: `0.03`, não `3`) nem como centavos.
- Não afirmar que a simulação criou operação/contrato — é cálculo apenas.
- Não misturar com Pix/extrato (`lcc-open-finance`) nem com volume de deals do dashboard.
