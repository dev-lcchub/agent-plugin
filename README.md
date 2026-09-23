# Agent Plugin `lcc.recebify`

Pacote [Agent Plugins](https://agent-plugins.org/) para Codex e Cursor, com Skills de domínio e conexão ao MCP do LCC/Recebify em produção.

Este repositório só empacota descoberta no cliente (plugin + Skills). O servidor MCP roda em produção:

`https://api.sistema.lcchub.com.br/mcp`

## Conteúdo

| Caminho | Função |
| --- | --- |
| `plugin.json` | Identidade do plugin (`name`: `lcc.recebify`) |
| `mcp.json` | Conexão MCP direta `lcc-backend` → `https://api.sistema.lcchub.com.br/mcp` |
| `.app.json` | Conector LCC.hub registrado no Codex |
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

No Codex, o plugin aponta em `.app.json` para o conector **MCP LCC.hub v0.3** já registrado. Instale e conecte esse conector à sua conta; depois abra uma nova tarefa e teste `quem_sou_eu`. A conexão direta de `mcp.json` continua disponível para clientes que carregam MCP pelo pacote.

O comando `codex mcp login lcc-backend` autentica a **conexão direta**, separada do conector registrado. Nesse fluxo, o `redirect_uri` com `127.0.0.1` é apenas o retorno temporário do OAuth no computador; o MCP segue em `https://api.sistema.lcchub.com.br/mcp`. Se o servidor responder `Redirect URI ... does not match allowed patterns`, o login direto está bloqueado, embora o conector registrado possa funcionar normalmente.

O ID em `.app.json` identifica o conector de desenvolvimento usado neste teste. Antes de distribuir o plugin a outras contas, confirme que elas têm acesso ao conector ou substitua o ID pelo conector publicado para esse público.

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

Auth continua no MCP (`auth_login` no modo legado, ou OAuth no conector). Nada de credencial no `mcp.json`.

## Checklist

- [ ] Serviço MCP em `https://api.sistema.lcchub.com.br/mcp`
- [ ] No Codex, plugin e conector LCC.hub instalados, com a conta conectada
- [ ] Symlink `~/.cursor/plugins/local/lcc.recebify` → esta pasta
- [ ] Após reload: server `lcc-backend` conecta e as Skills aparecem
- [ ] `quem_sou_eu` funciona

## Referências

- [Agent Plugins](https://agent-plugins.org/)
- [Cursor Plugins](https://cursor.com/docs/plugins)
