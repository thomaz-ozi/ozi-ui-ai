# Project Overview — OZI-UI

**Última atualização:** 2026-06-16

---

## O que é

**ozi-ui** é um plugin jQuery/JS distribuído via Composer e download direto. Fornece components, behaviors, modules e um core configurável compatível com qualquer stack PHP (vanilla, Laravel, etc.) e os principais frameworks CSS (Bootstrap 5, Tailwind 4, ou tema próprio).

---

## Repositórios GitHub

| Repositório | Visibilidade | Alias | Local | Propósito |
|---|---|---|---|---|
| `ozi-ui` | público / homologação | `ozi-ui` | `E:/xampp/www/ozi/ozi-ui` | O plugin em si — distribuído via download e Composer |
| `ozi-ui-ai` | privado | `ozi-ai` | `E:/xampp/www/ozi/ozi-ui-ai` | Experiência da IA — context, logs, prompts, setup |
| `ozi-ui-website` | privado / produção | `ozi-website` | `E:/xampp/www/ozi/ozi-ui-website` | Website do plugin (oziui.com) |
| `ozi-ui-docs` | privado | `ozi-docs` | `E:/xampp/www/ozi/ozi-ui-docs` | Documentação interna, changelogs, decisões |
| `ozi-ui-dev-hard` | privado | `dev-hard` | `E:/xampp/www/ozi/ozi-ui-dev-hard` | Sandbox principal — PHP puro, testes brutos do core |
| `ozi-ui-dev-bs` | privado | `dev-bs` | `E:/xampp/www/ozi/ozi-ui-dev-bs` | Sandbox secundário — Laravel 13+ Livewire 4 + Bootstrap 5 |
| `ozi-ui-dev-tw` | privado | `dev-tw` | `E:/xampp/www/ozi/ozi-ui-dev-tw` | Sandbox futuro — Laravel 13+ Livewire 4 + Tailwind 4 |

---

## Ambiente de Desenvolvimento

- **Runtime local:** XAMPP Lite 8.4.6
- **PHP:** 8.4.6
- **MariaDB:** 11.4
- **SO:** Windows 11 Pro

## Deploy — oziui.com

- **Provedor:** KingHost
- **PHP:** 8.3
- **Restrições:** sem Node.js, sem Composer no servidor
- **Estratégia:** `public/build` commitado no git; assets compilados localmente antes do push

---

## Estrutura do Plugin (`ozi-ui/`)

```
ozi-ui/
├── ozi.js                      ← entry point (auto-detect urlBase)
├── OziAssets.php               ← entry point para Laravel/Blade
├── core/
│   ├── ozi-conf.js             ← configuração global, classMap, _pluginMap
│   ├── ozi-hooks.js            ← afterRender / beforeRender
│   ├── ozi-lang.js             ← i18n (t(), register())
│   ├── ozi-helpers.js          ← funções puras
│   ├── ozi-loader.js           ← carregamento dinâmico JS/CSS
│   └── ozi-integrations.js     ← registry plugin ↔ adapter
├── modules/
│   ├── ozi-validate/           ← motor de validação (adapter pattern)
│   ├── ozi-actions/            ← ações declarativas do backend
│   ├── ozi-suggest/            ← busca compartilhada (select + autocomplete)
│   ├── ozi-password-rules/     ← validação de senha (puro, sem DOM)
│   └── ozi-loaddata/           ← orquestrador fetch/render central
├── components/
│   ├── ozi-select/             ← select customizado (single/multiple/group)
│   ├── ozi-autocomplete/       ← input com dropdown de sugestões
│   ├── ozi-editor/             ← rich text (HTML + MD)
│   ├── ozi-audio/              ← player + gravador
│   ├── ozi-auth/               ← validação de senha em tempo real
│   ├── ozi-check/              ← checkboxes hierárquicos (3 níveis)
│   └── ozi-search/             ← filtro DOM em tempo real
└── behaviors/
    ├── ozi-copy/               ← copia para clipboard
    ├── ozi-paste/              ← cola texto fixo
    └── ozi-toggle/             ← show/hide declarativo
```

---

## Documentação Interna (`ozi-ui-docs/dev/`)

49 arquivos organizados em:
- `_meta/` — guias transversais (guia-novo-plugin, dívidas técnicas, lang-keys, classMap-tokens, estados-componentes)
- `core/` — um `description.md` + `changelog.md` por subsistema
- `modules/` — idem
- `components/` — idem
- `behaviors/` — idem
- `index.md` — contexto de projeto (repos, ambiente)
- `README.md` — índice navegável com conceitos arquiteturais chave
