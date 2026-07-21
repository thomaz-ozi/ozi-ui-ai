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
│   ├── decisions.md            ← decisões arquiteturais com justificativas (#1–#20)
│   ├── regras-v2.md            ← 🆕 vocabulário/regras da IA no v2 (R1 "usar ozi-select"…)
│   └── aliases.md              ← atalhos dos repositórios
├── prompts/
│   ├── nova-sessao.md          ← iniciar qualquer sessão com contexto completo
│   ├── novo-componente.md      ← criar componente seguindo padrões do plugin
│   ├── novo-behavior.md        ← criar behavior com lifecycle correto
│   ├── novo-modulo.md          ← criar módulo com estrutura do plugin
│   ├── novo-lang.md            ← adicionar/ajustar dicionário i18n de um plugin
│   ├── novo-tema.md            ← criar/ajustar variante de tema
│   ├── integracao-livewire.md  ← implementar/debugar integração Livewire 3/4
│   ├── debug-plugin.md         ← investigar bug num plugin do ozi-ui
│   ├── review-plugin.md        ← checklist de review antes de publicar
│   └── sincronizar.md          ← sincronizar código/doc entre os repositórios
├── handoff/
│   └── current.md              ← estado de fim de sessão (lido no início da próxima)
└── logs/
    └── lessons-learned.md      ← aprendizados técnicos acumulados
```

---

## Como usar

1. **Nova sessão:** abra `prompts/nova-sessao.md` e cole o prompt completo no Claude Code
2. **Nova tarefa específica:** use o prompt da pasta `prompts/` que corresponde ao que você vai fazer
3. **Dúvidas de contexto:** consulte `context/` para referência rápida

---

*Alias do projeto: `-ai` | Local: `E:/xampp/www/ozi-ui/ozi-ui-ai`*
