# Stack — OZI-UI

---

## Linguagens e Runtime

| Camada | Tecnologia |
|---|---|
| Plugin (lógica) | JavaScript (ES5-compatible, jQuery 3.x) |
| Distribuição | PHP / Composer (`OziAssets.php` para Laravel) |
| Desenvolvimento | XAMPP PHP 8.4.6, MariaDB 11.4 |
| Sandboxes avançados | Laravel 13+ com Livewire 4 |

---

## Padrão de Arquivo JS

Todo arquivo do plugin segue o mesmo padrão **IIFE + 'use strict'**:

```js
(function ($, window, document) {
    'use strict';

    // [1] GUARD singleton
    if (window.OziNome) return;

    // [2] CONSTANTES
    // [3] CLASSMAP
    // [4] I18N
    // [5] INSTÂNCIAS (components) / lógica (behaviors)
    // [6] LÓGICA PRINCIPAL
    // [7] ADAPTER ozi-validate (components com campos)
    // [8] API PÚBLICA
    // [9] EXPOSIÇÃO + DOMReady + afterRender

})(jQuery, window, document);
```

---

## Namespace Global `window.OZI`

| Slot | Preenchido por |
|---|---|
| `OZI.version` | `ozi.js` |
| `OZI.conf` | `ozi-conf.js` |
| `OZI.helpers` | `ozi-helpers.js` |
| `OZI.modules` | plugins em `modules/` |
| `OZI.components` | plugins em `components/` |
| `OZI.behaviors` | plugins em `behaviors/` |
| `OZI.lang` | `ozi-lang.js` |
| `OZI.hooks` | `ozi-hooks.js` |
| `OZI.integrations` | `ozi-integrations.js` |
| `OZI.loader` | `ozi-loader.js` |
| `OZI.isReady` | `ozi.js` (após boot completo) |

---

## Temas (classMap)

| Token semântico | `default` | `bootstrap5` | `tailwind` |
|---|---|---|---|
| `invalid` | `ozi-invalid` | `is-invalid` | `border-red-500 text-red-600` |
| `valid` | `ozi-valid` | `is-valid` | `border-green-500 text-green-600` |
| `feedback` | `ozi-feedback` | `invalid-feedback` | `text-red-500 text-sm mt-1` |
| `button` | `ozi-btn` | `btn btn-primary` | `px-4 py-2 bg-blue-600 text-white rounded` |
| `disabled` | `ozi-disabled` | `disabled` | `opacity-50 cursor-not-allowed` |
| `formValidated` | `ozi-validated` | `was-validated` | `ozi-validated` |

**Regra de ouro:** plugins nunca usam classes de framework diretamente — sempre `_classMap('token', 'fallback-default')`.

---

## Boot Sequence

```
[ozi.js detecta urlBase]
    ↓
ozi-conf → ozi-hooks → ozi-lang → ozi-helpers → ozi-loader → ozi-integrations
    ↓
OziConf.init() + aplica oziConf() pendentes
    ↓
loader.loadPlugins() — carrega na ordem: validate → actions → suggest → password-rules → loaddata
                                         → select → autocomplete → editor → editor-md
                                         → audio → auth → check → search
                                         → copy → paste → toggle
    ↓
_bridgeHooks() — conecta zldConf.zldHooks → OZI.hooks (unidirecional)
    ↓
OZI.isReady = true → dispara OZI.ready() callbacks
```

---

## Integração com Laravel / Livewire

- **Entry point:** `OziAssets.php` substitui `ozi.js`
- **Blade helper:** `@oziScripts` emite HEAD + BODY + TAIL
- **Grupos:** `'default'` (plugins base) + `'livewire'` (adapters de integração)
- **Livewire 3:** hook `livewire:load` (formato antigo)
- **Livewire 4:** hook `livewire:init` (formato novo) — guard obrigatório pois formatos são diferentes
- **Arquivos de integração:** devem estar registrados no `_pluginMap` do `ozi-conf.js`

---

## i18n

- Chaves no formato `namespace.chave` (ex: `select.placeholder`, `common.required`)
- 3 locales mantidos: `pt-BR`, `en`, `es`
- Fallback embutido em cada plugin para PT-BR
- Acesso: `OZI.lang.t('chave', 'fallback')` ou helper local `_t(key, fallback)`

---

## Convenções de Nomenclatura

| Tipo | Arquivo | Exposição JS | Atributo HTML |
|---|---|---|---|
| Component | `components/ozi-{nome}/js/ozi-{nome}.js` | `OZI.components.{nome}` + `window.Ozi{Nome}` | `data-ozi-{nome}="key"` |
| Module | `modules/ozi-{nome}/js/ozi-{nome}.js` | `OZI.modules.{nome}` + `window.Ozi{Nome}` | — |
| Behavior | `behaviors/ozi-{nome}/js/ozi-{nome}.js` | `OZI.behaviors.{nome}` + `window.Ozi{Nome}` | `data-ozi-{nome}` |

---

## Dependências entre Plugins

```
loaddata → validate, actions
select   → loaddata, suggest, validate
autocomplete → loaddata, suggest, validate
editor-md → editor
audio    → loaddata
auth     → password-rules
```

Resolvidas automaticamente pelo algoritmo DFS do `ozi-conf.js`.
