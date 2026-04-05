# Claude-Mem no Cursor IDE

Integração do claude-mem com o Cursor, compartilhando a mesma memória já usada pelo Claude Code e Gemini CLI (`~/.claude-mem`).

## Status

- **Claude Code**: suporte completo nativo
- **Gemini CLI**: suporte completo via `/install-extension`
- **Cursor**: suporte em desenvolvimento — instalação via repositório clonado (sem `npx claude-mem install --ide cursor` ainda)

## Pré-requisitos

- claude-mem já instalado no Claude Code (worker rodando na porta `37777`)
- Bun instalado (`~/.bun/bin/bun`)
- Cursor IDE

## Instalação

O NPX installer não suporta Cursor ainda (`npx claude-mem install --ide cursor` retorna "Support for Cursor coming soon"). O caminho correto é via repositório clonado:

```bash
# 1. Clonar o repositório
git clone --depth 1 https://github.com/thedotmack/claude-mem.git /tmp/claude-mem-full

# 2. Instalar os hooks globalmente (para todos os projetos)
cd /tmp/claude-mem-full
~/.bun/bin/bun plugin/scripts/worker-service.cjs cursor install user

# 3. Limpar clone temporário (opcional)
rm -rf /tmp/claude-mem-full
```

Isso instala `~/.cursor/hooks.json` apontando para o worker-service já instalado pelo Claude Code plugin em:
`~/.claude/plugins/marketplaces/thedotmack/plugin/scripts/worker-service.cjs`

**A memória é automaticamente compartilhada** — os hooks se conectam ao worker existente na porta `37777`, que usa o mesmo `~/.claude-mem/claude-mem.db`.

## O que foi instalado

`~/.cursor/hooks.json`:

```json
{
  "version": 1,
  "hooks": {
    "beforeSubmitPrompt": [
      { "command": "\"~/.bun/bin/bun\" \"...worker-service.cjs\" hook cursor session-init" },
      { "command": "\"~/.bun/bin/bun\" \"...worker-service.cjs\" hook cursor context" }
    ],
    "afterMCPExecution": [
      { "command": "\"~/.bun/bin/bun\" \"...worker-service.cjs\" hook cursor observation" }
    ],
    "afterShellExecution": [
      { "command": "\"~/.bun/bin/bun\" \"...worker-service.cjs\" hook cursor observation" }
    ],
    "afterFileEdit": [
      { "command": "\"~/.bun/bin/bun\" \"...worker-service.cjs\" hook cursor file-edit" }
    ],
    "stop": [
      { "command": "\"~/.bun/bin/bun\" \"...worker-service.cjs\" hook cursor summarize" }
    ]
  }
}
```

## Como funciona

```
Cursor Agent
    │
    ▼ Eventos (MCP, shell, file edits, prompts)
Cursor Hooks System (hooks.json)
    │
    ▼ HTTP requests
Claude-Mem Worker (port 37777)
    │
    ▼
~/.claude-mem/claude-mem.db (SQLite + Chroma)
    │
    └── Compartilhado com Claude Code e Gemini CLI
```

| Hook Cursor           | Ação                                     |
|-----------------------|------------------------------------------|
| `beforeSubmitPrompt`  | Inicializa sessão, verifica worker        |
| `afterMCPExecution`   | Salva observação de tool MCP             |
| `afterShellExecution` | Salva observação de comando shell        |
| `afterFileEdit`       | Salva observação de edição de arquivo    |
| `stop`                | Gera resumo da sessão                    |

## Context Injection

Após cada sessão, o hook `stop` gera um resumo e atualiza o arquivo:
`.cursor/rules/claude-mem-context.mdc`

Esse arquivo é automaticamente incluído por todos os chats do Cursor (via Rules), eliminando a necessidade de reexplicar o contexto do projeto.

## Verificação

```bash
# Worker respondendo?
curl http://127.0.0.1:37777/api/readiness
# → {"status":"ready","mcpReady":true}

# Status dos hooks
cd /tmp/claude-mem-full && ~/.bun/bin/bun plugin/scripts/worker-service.cjs cursor status

# Web viewer com histórico de sessões
open http://localhost:37777
```

## MCP Tools no Cursor

Para ter as ferramentas de busca de memória disponíveis no Cursor, adicionar em `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "claude-mem": {
      "command": "/Users/kaicmurilo/.bun/bin/bun",
      "args": [
        "/Users/kaicmurilo/.claude/plugins/marketplaces/thedotmack/plugin/scripts/mcp-server.cjs"
      ]
    }
  }
}
```

Ferramentas disponíveis:
- `search(query)` — busca observações relevantes do passado
- `timeline(id)` — contexto cronológico em torno de uma entrada
- `get_observations([ids])` — detalhes completos por IDs

## Troubleshooting

**Hooks não executando:**
1. Cursor Settings → Hooks tab para ver erros
2. Reiniciar o Cursor após instalação
3. Verificar se o arquivo existe: `cat ~/.cursor/hooks.json`

**Worker não responde:**
```bash
# Reiniciar worker (via Claude Code ou diretamente)
~/.bun/bin/bun ~/.claude/plugins/marketplaces/thedotmack/plugin/scripts/worker-cli.js restart

# Ver logs
tail -f ~/.claude-mem/logs/worker-$(date +%Y-%m-%d).log
```

**Limitação vs Claude Code:**
- O Cursor não fornece o transcript da sessão, então os resumos têm menos detalhes que os gerados pelo Claude Code
- O hook `beforeSubmitPrompt` não permite injetar contexto direto no prompt — use as MCP tools para busca ativa

## Memória compartilhada entre IDEs

| IDE         | Lê memória | Escreve memória | Configuração                  |
|-------------|------------|-----------------|-------------------------------|
| Claude Code | ✅          | ✅               | Plugin nativo (`enabledPlugins`) |
| Gemini CLI  | ✅          | ✅               | `/install-extension thedotmack/claude-mem` |
| Cursor      | ✅ (via MCP) | ✅ (via hooks)  | `cursor install user` (manual) |

Todos apontam para o mesmo banco: `~/.claude-mem/claude-mem.db`
