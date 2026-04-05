# Gemini CLI como Subagente no Claude Code

Use o Gemini CLI rodando localmente como subagente dentro do Claude Code para delegar tarefas simples e economizar tokens do Claude.

## Como funciona

```
Claude Code (você fala)
     ↓ delega tarefas simples
Gemini CLI (roda na sua máquina)
     ↓ responde
Claude Code (integra a resposta)
```

O pacote `gemini-mcp-tool` expõe o Gemini CLI local como um servidor MCP (Model Context Protocol), permitindo que o Claude Code chame o Gemini diretamente como ferramenta.

## Pré-requisitos

- [Claude Code](https://claude.ai/code) instalado
- [Gemini CLI](https://github.com/google-gemini/gemini-cli) instalado e autenticado
- Node.js (para o `npx`)

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
- Buscas em arquivos que não precisam de contexto da conversa

## Limitações

- O Gemini não tem acesso ao histórico da conversa com o Claude
- Para tarefas que dependem de contexto acumulado, o Claude deve responder diretamente
- Requer autenticação ativa do Gemini CLI (`~/.gemini/oauth_creds.json`)

## Referências

- [gemini-mcp-tool no npm](https://www.npmjs.com/package/gemini-mcp-tool)
- [Gemini CLI](https://github.com/google-gemini/gemini-cli)
- [Model Context Protocol](https://modelcontextprotocol.io)
- [Claude Code MCP docs](https://docs.anthropic.com/en/docs/claude-code/mcp)
