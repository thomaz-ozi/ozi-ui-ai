# Prompt — Novo Componente

Use quando for criar um novo component (widget visual que substitui/envolve um elemento HTML).

---

```
Vamos criar um novo component para o ozi-ui chamado `ozi-{nome}`.

## Referências obrigatórias antes de começar
- Leia `ozi-ui-docs/dev/_meta/guia-novo-plugin.md` — template completo e checklist
- Leia `ozi-ui-docs/dev/components/ozi-select/description.md` — component mais completo como referência

## O que o component deve fazer
[DESCREVA A FUNCIONALIDADE]

## Elemento base
[ex: `<div>`, `<input>`, `<select>` nativo]

## Atributo de identificação
[ex: `data-ozi-{nome}="key"`]

## Tem campos de formulário? (precisa de adapter ozi-validate?)
[SIM / NÃO]

## Checklist que o código final DEVE ter
- [ ] IIFE com `'use strict'`
- [ ] Singleton guard `if (window.Ozi{Nome}) return`
- [ ] Namespace defensivo `OZI.components = OZI.components || {}`
- [ ] `_classMap(token, fallback)` — sem classes BS5/TW hardcoded
- [ ] `_t(key, fallback)` — sem strings PT hardcoded
- [ ] Boot via `$(function(){ _boot(); })`
- [ ] `OZI.hooks.afterRender.register('component:{nome}', fn)`
- [ ] Adapter registrado em `OZI.modules.validate` (se tiver campos)
- [ ] `OZI.components.{nome} = api`
- [ ] `window.Ozi{Nome} = api`
- [ ] Lang em `lang/pt-BR.js`, `lang/en.js`, `lang/es.js`
- [ ] Entrada no `_pluginMap` do `ozi-conf.js`

## Arquivos a criar
- `components/ozi-{nome}/js/ozi-{nome}.js`
- `components/ozi-{nome}/css/ozi-{nome}.css` (se tiver estilos)
- `components/ozi-{nome}/lang/pt-BR.js`
- `components/ozi-{nome}/lang/en.js`
- `components/ozi-{nome}/lang/es.js`

## Documentação a criar
- `ozi-ui-docs/dev/components/ozi-{nome}/description.md`
- `ozi-ui-docs/dev/components/ozi-{nome}/changelog.md`
```
