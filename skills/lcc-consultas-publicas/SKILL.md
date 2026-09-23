---
name: lcc-consultas-publicas
description: Consulta dados públicos de um CNPJ que não estão na base do LCC (Receita/BrasilAPI) — razão social, situação cadastral, CNAE, sócios, endereço. Use quando o usuário perguntar cadastro da empresa, CNAE, quem são os sócios, endereço fiscal, se o CNPJ está ativo/baixado, ou qualquer ficha cadastral do CNPJ. Não use para taxas, limites ou dados internos da company.
---

# Consultas públicas

Playbook para tools **sem autenticação** no MCP `lcc-backend`.

## consultar_cnpj

- **Nível:** público — não exige `auth_login` nem sessão OAuth
- Fonte: [BrasilAPI](https://brasilapi.com.br/) (dados cadastrais públicos)
- Entrada: CNPJ com 14 dígitos (máscara aceita)

Retorno típico: razão social, situação cadastral, CNAE, sócios, endereço.

## Quando usar

- Enriquecer CNPJ visto em deal, empresa ou `user_company` **antes** de cruzar com tools autenticadas
- Validar formato/existência cadastral de um CNPJ desconhecido
- Complementar `obter_empresa` quando só se tem o CNPJ, não o `company_id`

## Quando **não** usar

- Dados internos LCC (taxas, limites, usuários) → `obter_empresa` / `listar_empresas` (owner+)
- Score Serasa ou recebíveis → domínios autenticados respectivos

## Ordem típica

1. `consultar_cnpj(cnpj)` — se necessário enriquecer
2. `auth_login` / OAuth + `quem_sou_eu` — para dados internos
3. Buscar empresa/deal no escopo com o contexto obtido

## O que não fazer

- Não tratar BrasilAPI como fonte de dados contratuais ou financeiros do LCC.
- Não repetir a consulta em loop para muitos CNPJs sem necessidade (rate limit externo).
- Não confundir situação cadastral pública com `status` interno da company (`PENDING`/`APPROVED`/`REJECTED`).
- Não invente sócio, CNAE ou endereço se a BrasilAPI não devolver.
