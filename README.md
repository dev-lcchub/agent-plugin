# Agent Plugin `lcc.recebify`

Pacote [Agent Plugins](https://agent-plugins.org/) para Codex e Cursor, com Skills de domínio e conexão ao MCP do LCC/Recebify em produção.

Este repositório só empacota descoberta no cliente (plugin + Skills). O servidor MCP roda em produção:

`https://api.sistema.lcchub.com.br/mcp`

## Conteúdo

| Caminho | Função |
| --- | --- |
| `plugin.json` | Identidade do plugin (`name`: `lcc.recebify`) |
| `mcp.json` | Conexão MCP direta `lcc-backend` → `https://api.sistema.lcchub.com.br/mcp` |
| `skills/lcc-sessao/` | Auth, níveis, centavos |
| `skills/lcc-consultas-publicas/` | `consultar_cnpj` (BrasilAPI, sem auth) |
| `skills/lcc-empresas/` | `listar_empresas`, `obter_empresa` |
| `skills/lcc-usuarios-empresa/` | Usuários, roles, flags OF/CERC |
| `skills/lcc-deals-propostas/` | Produtos, deals, propostas, comentários |
| `skills/lcc-recebiveis-antecipacao/` | CERC AP005 + `calcular_antecipacao` |
| `skills/lcc-open-finance/` | Resumos, lançamentos, atualização oficial e backfill histórico |
| `skills/lcc-dashboard/` | KPIs agregados e vendas por produto |
| `skills/lcc-staff-tasks/` | Tarefas internas (superadmin) |

## Uso no Codex

O Codex carrega o plugin pelo marketplace deste repositório. Para instalar em uma máquina nova:

```bash
codex plugin marketplace add https://github.com/dev-lcchub/agent-plugin.git --sparse .agents/plugins
codex plugin add lcc.recebify@lcc-recebify
```

Depois, abra uma nova tarefa e mencione `@lcc.recebify`. A cópia instalada fica no cache do Codex; editar um checkout local não atualiza automaticamente a instalação.

### Autenticação

No Codex, o plugin usa somente a conexão MCP direta `lcc-backend` de `mcp.json`. Autentique essa conexão com `codex mcp login lcc-backend`, abra uma nova tarefa e teste `quem_sou_eu`.

Nesse fluxo, o `redirect_uri` com `127.0.0.1` é apenas o retorno temporário do OAuth no computador; o MCP segue em `https://api.sistema.lcchub.com.br/mcp`. Se o servidor responder `Redirect URI ... does not match allowed patterns`, confira a configuração do cliente e o retorno enviado antes de decidir se alguma alteração no servidor é necessária.

O antigo conector registrado **MCP LCC.hub v0.3** não faz parte deste plugin. Atualizá-lo não atualiza a conexão `lcc-backend`.

## Instalação no Cursor (uma vez)

O Cursor carrega plugins locais de `~/.cursor/plugins/local` ([docs](https://cursor.com/docs/plugins)):

```bash
# Na raiz deste repositório:
mkdir -p ~/.cursor/plugins/local
ln -sfn "$(pwd)" ~/.cursor/plugins/local/lcc.recebify
```

Depois: **Developer: Reload Window**. Em **Customize**, devem aparecer o plugin `lcc.recebify`, o server `lcc-backend` e as nove Skills.

O symlink faz o Cursor ler **esta pasta do repo**. Não copie os arquivos — senão as atualizações do git não chegam.

## Como atualiza no Cursor

No fluxo local do Cursor, não há marketplace nem auto-refresh. O fluxo é:

1. Alguém altera este repo (skill nova, `mcp.json`, etc.) e isso entra no git.
2. Você dá `git pull` (ou já está na branch com as mudanças).
3. Como o Cursor aponta para a pasta via symlink, os arquivos novos já estão no disco.
4. Rode **Developer: Reload Window** para o Cursor reler `plugin.json`, `mcp.json` e as Skills.

Sem o reload, o Cursor pode continuar com a versão antiga em memória.

Auth continua no MCP (`auth_login` no modo legado, ou OAuth no cliente). Nada de credencial no `mcp.json`.

## Checklist

- [ ] Serviço MCP em `https://api.sistema.lcchub.com.br/mcp`
- [ ] No Codex, plugin instalado e `lcc-backend` autenticado
- [ ] Symlink `~/.cursor/plugins/local/lcc.recebify` → esta pasta
- [ ] Após reload: server `lcc-backend` conecta e as Skills aparecem
- [ ] `quem_sou_eu` funciona
- [ ] `solicitar_atualizacao_transacoes` e `solicitar_backfill_transacoes` aparecem no catálogo de tools do `lcc-backend`

## Referências

- [Agent Plugins](https://agent-plugins.org/)
- [Cursor Plugins](https://cursor.com/docs/plugins)
