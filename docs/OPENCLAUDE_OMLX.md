# Guia Completo: OpenClaude + oMLX (Local LLM)

Este guia ensina como rodar o Claude Code localmente usando o oMLX para economizar tokens e ter privacidade.

## 1. Instalar o oMLX
- Baixe o oMLX em: https://github.com/Yidadaa/omlx
- Instale no macOS e abra o aplicativo.
- Vá em **Settings** -> **Auth** e defina uma API Key (ex: `k4icolas`).
- Baixe um modelo (ex: Gemma 2B) e configure o alias `claude-sonnet-4-6` nas configurações do modelo para enganar a validação do Claude Code.

## 2. Instalar o OpenClaude
O OpenClaude é um wrapper que permite usar qualquer LLM com o Claude Code.

```bash
npm install -g @gitlawb/openclaude
```

## 3. Configurar o Shell (~/.zshrc)
Adicione o seguinte alias ao seu `~/.zshrc`:

```bash
alias claude-local='export CLAUDE_CODE_USE_OPENAI=1 && export OPENAI_BASE_URL="http://localhost:8000/v1" && export OPENAI_API_KEY="k4icolas" && export OPENAI_MODEL="claude-sonnet-4-6" && openclaude'
```

Depois, atualize o shell:
```bash
source ~/.zshrc
```

## 4. Configurar Perfil Persistente
Crie o arquivo `~/.openclaude-profile.json` para que o OpenClaude saiba que deve usar o provedor OpenAI:

```json
{
  "currentProfile": "omlx",
  "profiles": {
    "omlx": {
      "provider": "openai",
      "model": "claude-sonnet-4-6",
      "baseUrl": "http://localhost:8000/v1",
      "apiKey": "k4icolas"
    }
  }
}
```

## 5. Rodar
Para usar o Claude oficial: `claude`
Para usar o Claude local (oMLX): `claude-local`
