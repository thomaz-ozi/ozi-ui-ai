# Prompt — Novo Módulo

Use quando for criar um novo module (lógica compartilhada sem UI própria, consumido por components).

---

```
Vamos criar um novo módulo para o ozi-ui chamado `ozi-{nome}`.

## Referências
- Leia `ozi-ui-docs/dev/_meta/guia-novo-plugin.md`
- Leia `ozi-ui-docs/dev/modules/ozi-suggest/description.md` — módulo com AbortController + debounce como referência

## O que o módulo deve fazer
[DESCREVA A RESPONSABILIDADE]

## Quem vai consumir este módulo
[ex: ozi-select, ozi-autocomplete, outros]

## Tem dependências de outros módulos?
[ex: depende de ozi-validate]

## Diferenças dos components
- SEM UI própria — sem HTML gerado, sem estilos visuais
- SEM adapter de ozi-validate (a menos que o módulo seja o validate em si)
- SEM `afterRender` (não re-inicializa)
- Expõe `OZI.modules.{nome}` com API de funções

## Padrão de exposição de módulo

```js
var api = {
    metodo1: function() { ... },
    metodo2: function() { ... }
};

window.Ozi{Nome} = api;

$(function() {
    if (window.OZI) {
        window.OZI.modules = window.OZI.modules || {};
        window.OZI.modules['{nome}'] = api;
    }
});
```

## Arquivos a criar
- `modules/ozi-{nome}/js/ozi-{nome}.js`
- `modules/ozi-{nome}/lang/pt-BR.js` (se tiver textos)

## Registrar no `_pluginMap` do ozi-conf.js
```js
'{nome}': {
    deps: [],   // deps se houver
    js:   'modules/ozi-{nome}/js/ozi-{nome}.js',
    css:  null,
    lang: null
}
```
E adicionar na `_loadOrder` antes dos components que o consomem.

## Documentação a criar
- `ozi-ui-docs/dev/modules/ozi-{nome}/description.md`
- `ozi-ui-docs/dev/modules/ozi-{nome}/changelog.md`
```
