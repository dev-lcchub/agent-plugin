---
name: lcc-sessao
description: "Identidade e conta no LCC — quem está logado, nível de acesso, empresa/ecossistema ativo e troca de company. Use no começo da conversa, quando o usuário pedir “quem sou eu”, dados da conta, login/logout, ou “trocar para tal empresa/company/ecossistema”. Dinheiro nas tools vem em centavos: chamar em centavos e só formatar em reais na resposta."
---

# Sessão e identidade no MCP LCC

Use esta skill antes de qualquer fluxo de negócio no servidor MCP `lcc-backend`.

## Convenções obrigatórias

- **Valores em dinheiro** (`amount`, `available_value`, totais de Pix, etc.) são **inteiros em centavos**. Nas tools, passe centavos; na resposta ao usuário, formate em reais (R$ 1.234,56 ← `123456`).
- **Taxas e percentuais** (`tax_per_month`, `commission`, `rate` de proposta) são **fração** (`0.03` = 3%). Não misture com centavos.
- Nível da sessão (`client` | `owner` | `superadmin`) limita o queryset. Não invente dado fora do retorno.
- Tool abaixo do nível devolve erro acionável — leia a mensagem (autenticar, trocar empresa ou credencial com nível adequado).

## Níveis

| Nível | Quem | Escopo típico |
| --- | --- | --- |
| `client` | BORROWER | próprios dados; próprios deals |
| `client` | LENDER | próprios dados de usuário; **deals da empresa + filiais**; propostas em que é o lender |
| `owner` | OWNER / MANAGER | empresa + filiais (`company.descendants`) |
| `superadmin` | Staff | tudo (`SUPERVISOR` é só leitura) |

## Autenticação

### Modo legado (`MCP_AUTH_MODE=legacy`, default local)

1. `auth_login` com **um** par: M2M (`client_id` + `client_secret`) **ou** staff (`email` + `password`)
2. `quem_sou_eu` — nível, usuário, empresa
3. Várias companies no mesmo e-mail: `trocar_empresa(company_scope=...)`
4. `auth_logout` encerra a sessão MCP

### Modo OAuth (`MCP_AUTH_MODE=oauth`)

1. Auth na conexão MCP `lcc-backend` do cliente — não use `auth_login` (devolve aviso)
2. `quem_sou_eu` confere a identidade do token
3. `trocar_empresa` vale para o token atual
4. Sair: revogar OAuth no cliente; `auth_logout` não encerra sessão local

## Codex: MCP direto

O plugin expõe a conexão MCP direta `lcc-backend` de `mcp.json`. Use somente as tools dessa conexão. Tools rotuladas `MCP LCC.hub v0.3` pertencem ao conector antigo e não validam este plugin.

Se a conexão falhar com `Auth required`, seu login pode ser iniciado por `codex mcp login lcc-backend`. O `redirect_uri` com `127.0.0.1` é o retorno local do OAuth; o MCP permanece na URL remota. Se `/authorize` devolver `Redirect URI ... does not match allowed patterns`, explique que o login foi recusado pelo servidor de autorização. Não peça credenciais no chat nem sugira trocar a URL do MCP como solução.

## Ordem em conversa nova

1. Confirme que as tools vêm da conexão MCP `lcc-backend`.
2. `quem_sou_eu` — confirme nível, usuário e empresa; no modo legado, usar `auth_login` se necessário.
3. Só então tools de domínio.
4. Empresa errada: `trocar_empresa` e `quem_sou_eu` de novo.

## O que não fazer

- Não chamar tools de negócio sem sessão válida.
- Não assumir `owner`/`superadmin` sem `quem_sou_eu`.
- Não colocar secret em header do plugin nem repetir senha no chat depois do login.
- Não converter taxa/comissão como se fosse centavos.
