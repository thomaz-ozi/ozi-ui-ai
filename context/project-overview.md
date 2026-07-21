# Project Overview — OZI-UI

**Última atualização:** 2026-07-05

---

## O que é

**ozi-ui** é um plugin JS distribuído via Composer e download direto. Fornece components, behaviors, modules e um core configurável compatível com qualquer stack PHP (vanilla, Laravel, etc.) e os principais frameworks CSS (Bootstrap 5, Tailwind 4, ou tema próprio).

- **v1** (tag `v1-final`, em produção no Central RH): plugin **jQuery/JS**.
- **v2** (branch `v2` do `dev-hard`): **JS puro, zero dependência de terceiros** — jQuery só em `integrations/` (shims), visual só em `themes/`.

## Estado da migração v2 (JS puro)

> **F0 ✅ · F1 ✅ · F2 ✅ (10/10) · F3 ✅ (temas) · F4 ✅ (integrações) · F5-A (doc) ✅ · F5-B pendente** (2026-07-05).
> **F4:** adapter Livewire → **v2.0.0** — `ozi:change → wire:model` via dispatch nativo `input`/`change` (modo A, respeita `.live`/`.debounce`/`.lazy`), `component.set()` (modo B) fallback; **guard por `e.detail.source==='api'`** evita o loop wire:model→setValue→ozi:change→set (contrato §1.3); aceite `aceite-livewire.html` 9/9. `ozi-hooks` v1.0.2 documentado como re-init oficial (LW3/LW4 com guard de formato de `commit`); bridge `zldHooks→OZI.hooks` unidirecional (marcada p/ remoção no corte); receitas Alpine copy/paste em `dev/_meta/receitas-alpine.md` (resolve dívida #11). Diferido p/ a leva do pacote: revisão do `OziAssets.php` (vive em `ozi-ui/src/`).
> 10 componentes de UI + 3 módulos internos migrados para JS puro no `ozi-ui-dev-hard` (branch `v2`). `ozi-audio` v4.0.0 fechou a F2 (reincluído revertendo a descontinuação da F0; aceite headless 26/26 via CDP). **F3 (temas):** criado `themes/tailwind/overrides.css` (única lacuna estrutural), `tailwind/dark.css`→v1.0.1 (`@media` completado), `_template/` reestruturado em 4 arquivos + README; validado que o mesmo build JS serve os 3 temas trocando só `oziConf({theme})` (aceite `aceite-temas.html` 14/14). O pacote distribuível `ozi-ui` ainda é placeholder v1.0.7 (sincroniza no 1º release v2).
>
> **Versionamento — major +1 por plugin** (não versão única): `ozi.js` (índice da geração) 1.0.7→**2.0.0**, `ozi-conf`→**3.0.0**, loaddata→5.0.0, select→6.0.0, autocomplete/editor/auth/search/**audio**→4.0.0, check/toggle→3.0.0, validate/actions/suggest→2.x, helpers→1.1.0. Descontinuados (uso zero): `ozi-copy`, `ozi-paste`.
>
> **Ordem de sync entre repos (decisão 2026-07-05, "uma coisa de cada vez"):** (1) `-ai` + `-hard` + `-docs` → (2) `-bs` → (3) `-tw` + `-pkg`/`ozi-ui`. Detalhe: `ozi-ui-docs/horizonte/roadmap/ozi-ui-v2-projeto-implementacao.md`.

---

## Repositórios GitHub

| Repositório | Visibilidade | Alias | Local | Propósito |
|---|---|---|---|---|
| `ozi-ui` | público / homologação | `-pkg` | `E:/xampp/www/ozi-ui/ozi-ui-pkg` | O plugin em si — distribuído via download e Composer |
| `ozi-ui-ai` | privado | `-ai` | `E:/xampp/www/ozi-ui/ozi-ui-ai` | Experiência da IA — context, logs, prompts, setup |
| `ozi-ui-website` | privado / produção | `-website` | `E:/xampp/www/ozi-ui/ozi-ui-website` | Website do plugin (oziui.com) |
| `ozi-ui-docs` | privado | `-docs` | `E:/xampp/www/ozi-ui/ozi-ui-docs` | Documentação interna, changelogs, decisões |
| `ozi-ui-dev-hard` | privado | `-hard` | `E:/xampp/www/ozi-ui/ozi-ui-dev-hard` | Sandbox principal — PHP puro, testes brutos do core |
| `ozi-ui-dev-bs` | privado | `-bs` | `E:/xampp/www/ozi-ui/ozi-ui-dev-bs` | Sandbox secundário — Laravel 13+ Livewire 4 + Bootstrap 5 |
| `ozi-ui-dev-tw` | privado | `-tw` | `E:/xampp/www/ozi-ui/ozi-ui-dev-tw` | Sandbox futuro — Laravel 13+ Livewire 4 + Tailwind 4 |

### Projeto host (consumidor em produção)

| Projeto | Local | Stack | Papel |
|---|---|---|---|
| **Central RH** | `C:/xampp_lite_8_4/www/centralrh/centralrh12` | Laravel 12 + Livewire 4 + Bootstrap 5, `ozi-ui/core ^1.0` via Composer | Aplicação real que consome o ozi-ui v1 — referência de compatibilidade da migração v2 (inventário: `ozi-ui-docs/horizonte/roadmap/ozi-ui-v2-f0-inventario-centralrh.md`) |

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
