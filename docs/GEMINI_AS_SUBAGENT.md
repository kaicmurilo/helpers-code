# Gemini CLI como Subagente no Claude Code

Use o Gemini CLI rodando localmente como subagente dentro do Claude Code para delegar tarefas simples e economizar tokens do Claude.

Combinado com o [claude-mem](https://github.com/thedotmack/claude-mem), o Gemini compartilha a mesma memória persistente do Claude Code — projetos, decisões e histórico de trabalho ficam disponíveis para ambos automaticamente.

## Como funciona

```
Claude Code (você fala)
     ↓ delega tarefas simples via MCP
Gemini CLI (roda na sua máquina)
     ↑↓ lê/escreve memória compartilhada
  claude-mem (~/.claude-mem)
     ↑↓ lê/escreve memória compartilhada
Claude Code
```

O pacote `gemini-mcp-tool` expõe o Gemini CLI local como um servidor MCP (Model Context Protocol), permitindo que o Claude Code chame o Gemini diretamente como ferramenta.

## Pré-requisitos

- [Claude Code](https://claude.ai/code) instalado
- [Gemini CLI](https://github.com/google-gemini/gemini-cli) instalado e autenticado
- Node.js (para o `npx`)
- (Recomendado) [claude-mem](https://github.com/thedotmack/claude-mem) instalado em ambos os IDEs

### Instalar o Gemini CLI

```bash
npm install -g @google/gemini-cli
gemini  # faz login na primeira execução
```

## Configuração

### Opção A: Projeto específico (recomendado)

Crie ou edite o arquivo `.claude.json` na raiz do projeto (ou em `~/.claude.json` para uso global via Claude Code desktop):

```json
{
  "mcpServers": {
    "gemini-cli": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "gemini-mcp-tool"],
      "env": {}
    }
  }
}
```

> No Claude Code, os projetos ficam em `~/.claude.json` dentro da chave `projects.<caminho-do-projeto>.mcpServers`.

### Opção B: Via Claude Code CLI

```bash
claude mcp add gemini-cli -- npx -y gemini-mcp-tool
```

### Verificar se está funcionando

Peça ao Claude: *"ping o servidor gemini-cli"* ou simplesmente pergunte algo e veja se ele usa o Gemini.

## Ferramentas disponíveis após configuração

| Ferramenta | Descrição |
|-----------|-----------|
| `ask-gemini` | Envia prompt para o Gemini CLI com suporte a arquivos via `@` |
| `brainstorm` | Modo brainstorming com o Gemini |
| `fetch-chunk` | Busca chunks de resposta longas |
| `ping` | Verifica se o servidor está ativo |

## Como instruir o Claude a usar o Gemini

Após configurar, você pode dizer ao Claude:

> "Use o Gemini para tarefas simples como leitura de arquivos grandes, resumos e análises básicas para economizar tokens."

Ou adicionar isso ao seu `CLAUDE.md`:

```markdown
## Política de delegação

Para tarefas simples (resumos, leitura de arquivos, perguntas factuais),
use o servidor MCP `gemini-cli` com `model: "gemini-2.5-flash"` para
economizar tokens. Use seus próprios tokens apenas para raciocínio complexo.
```

## Modelos disponíveis

| Modelo | Uso ideal |
|--------|-----------|
| `gemini-2.5-flash` | Tarefas rápidas e baratas (resumos, análises simples) |
| `gemini-2.5-pro` | Tarefas mais complexas (padrão) |

Exemplo de prompt para o Claude:
```
ask-gemini com model="gemini-2.5-flash": resuma o arquivo @src/index.ts
```

## Casos de uso ideais para delegar ao Gemini

- Leitura e resumo de arquivos grandes
- Análise de logs
- Perguntas factuais sobre código
- Explicações simples de funções
- Qualquer tarefa que não dependa do fio da conversa atual

## Contexto compartilhado via claude-mem

O [claude-mem](https://github.com/thedotmack/claude-mem) é um plugin de memória persistente para Claude Code e Gemini CLI que armazena observações de cada sessão em um banco local (`~/.claude-mem`).

Quando instalado em ambos os IDEs apontando para o **mesmo diretório**, Claude Code e Gemini CLI passam a compartilhar automaticamente:
- Histórico de trabalho de sessões anteriores
- Decisões de arquitetura e contexto de projetos
- Bugs, soluções e aprendizados acumulados

### Instalar o claude-mem no Claude Code

Adicione em `~/.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "claude-mem@thedotmack": true
  },
  "extraKnownMarketplaces": {
    "thedotmack": {
      "source": {
        "source": "github",
        "repo": "thedotmack/claude-mem"
      }
    }
  }
}
```

### Instalar o claude-mem no Gemini CLI

```bash
gemini  # abra o Gemini CLI
/install-extension thedotmack/claude-mem
```

O Gemini CLI adiciona automaticamente os hooks em `~/.gemini/settings.json` para ler e escrever no mesmo `~/.claude-mem`.

### Resultado

O Gemini já sabe o que aconteceu em sessões passadas. O único contexto que ele **não** tem é o fio da **conversa atual** — e isso é fácil de contornar passando as informações relevantes no prompt:

```
ask-gemini: "Estamos migrando o auth middleware por requisito de compliance.
Analise @src/auth/middleware.ts e liste os pontos que precisam mudar."
```

## Limitações

- Não tem acesso ao fio da conversa atual — passe contexto relevante no prompt quando necessário
- Requer autenticação ativa do Gemini CLI (`~/.gemini/oauth_creds.json`)

## Referências

- [gemini-mcp-tool no npm](https://www.npmjs.com/package/gemini-mcp-tool)
- [Gemini CLI](https://github.com/google-gemini/gemini-cli)
- [claude-mem](https://github.com/thedotmack/claude-mem) — memória persistente compartilhada entre IDEs
- [Model Context Protocol](https://modelcontextprotocol.io)
- [Claude Code MCP docs](https://docs.anthropic.com/en/docs/claude-code/mcp)
