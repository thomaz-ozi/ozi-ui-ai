# ozi-ui-ai

Repositório de experiência da IA para o projeto **ozi-ui**.

Contém o contexto acumulado, prompts reutilizáveis e aprendizados técnicos que permitem retomar qualquer sessão de desenvolvimento com contexto completo.

---

## Estrutura

```
ozi-ui-ai/
├── README.md                   ← este arquivo
├── setup/
│   ├── mcp-config.md           ← guia de configuração MCP no Windows
│   ├── settings.json           ← template .claude/settings.json
│   └── settings.local.json     ← template de permissões locais
├── context/
│   ├── project-overview.md     ← repositórios, ambientes, aliases, fluxo
│   ├── stack.md                ← stack técnica, arquitetura, convenções
│   └── decisions.md            ← decisões arquiteturais com justificativas
├── prompts/
│   ├── nova-sessao.md          ← iniciar qualquer sessão com contexto completo
│   ├── novo-componente.md      ← criar componente seguindo padrões do plugin
│   ├── novo-behavior.md        ← criar behavior com lifecycle correto
│   ├── novo-modulo.md          ← criar módulo com estrutura do plugin
│   ├── novo-tema.md            ← criar/ajustar variante de tema
│   ├── integracao-livewire.md  ← implementar/debugar integração Livewire 3/4
│   └── review-plugin.md        ← checklist de review antes de publicar
└── logs/
    └── lessons-learned.md      ← aprendizados técnicos acumulados
```

---

## Como usar

1. **Nova sessão:** abra `prompts/nova-sessao.md` e cole o prompt completo no Claude Code
2. **Nova tarefa específica:** use o prompt da pasta `prompts/` que corresponde ao que você vai fazer
3. **Dúvidas de contexto:** consulte `context/` para referência rápida

---

*Alias do projeto: `ozi-ai` | Local: `E:/xampp/www/ozi/ozi-ui-ai`*
