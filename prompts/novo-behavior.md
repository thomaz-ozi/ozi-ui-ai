# Prompt — Novo Behavior

Use quando for criar um novo behavior (comportamento declarativo ativado por atributo, sem substituição de elemento).

---

```
Vamos criar um novo behavior para o ozi-ui chamado `ozi-{nome}`.

## Referências
- Leia `ozi-ui-docs/dev/_meta/guia-novo-plugin.md` — seção Template Behavior
- Leia `ozi-ui-docs/dev/behaviors/ozi-copy/description.md` — referência

## O que o behavior deve fazer
[DESCREVA O COMPORTAMENTO]

## Atributo trigger
[ex: `data-ozi-{nome}` ou `data-ozi-{nome}-target`]

## Evento que dispara a ação
[ex: click, change, input]

## Diferenças dos components (behaviors são mais simples)
- SEM instâncias por elemento — singleton puro
- SEM adapter de ozi-validate
- Bind delegado no `document` — funciona com conteúdo dinâmico automaticamente
- Hook `afterRender` registrado mas sem ação (bind delegado já cobre tudo)

## Checklist que o código final DEVE ter
- [ ] IIFE com `'use strict'`
- [ ] Singleton guard `if (window.Ozi{Nome}) return`
- [ ] Bind: `$(document).off('click.ozi{Nome}', SELECTOR).on('click.ozi{Nome}', SELECTOR, handler)`
- [ ] `_t(key, fallback)` para textos
- [ ] `_emit` duplo: `$(el).trigger(event)` + `CustomEvent` nativo com `bubbles: true`
- [ ] Namespace defensivo `OZI.behaviors = OZI.behaviors || {}`
- [ ] `OZI.behaviors.{nome} = api`
- [ ] `window.Ozi{Nome} = api`
- [ ] `OZI.hooks.afterRender.register('behavior:{nome}', function() {})` (vazio)
- [ ] Entrada no `_pluginMap` do `ozi-conf.js`

## Feedback visual (se aplicável)
Behaviors com feedback (copy, paste) usam tooltip com:
- `fadeOut(1000ms)` após exibir
- `remove()` após `2200ms`
- Ícones via `OZI.helpers.icon()` (não FA4 hardcoded — dívida #11)

## Arquivos a criar
- `behaviors/ozi-{nome}/js/ozi-{nome}.js`
- `behaviors/ozi-{nome}/css/ozi-{nome}.css` (se tiver estilos)
- `behaviors/ozi-{nome}/lang/pt-BR.js` (se tiver textos)

## Documentação a criar
- `ozi-ui-docs/dev/behaviors/ozi-{nome}/description.md`
- `ozi-ui-docs/dev/behaviors/ozi-{nome}/changelog.md`
```
