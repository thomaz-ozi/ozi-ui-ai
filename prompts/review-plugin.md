# Prompt — Review de Plugin

Use antes de publicar, mergear ou considerar um plugin "pronto".

---

```
Faça um review completo do plugin `ozi-{nome}` antes de publicar.

## Arquivo principal
`ozi-ui/{tipo}/ozi-{nome}/js/ozi-{nome}.js`

## Checklist obrigatório (do guia-novo-plugin.md)

### Estrutura
- [ ] IIFE com `'use strict'`
- [ ] Singleton guard (`if (window.OziNome) return`)
- [ ] Namespace defensivo (`OZI.{tipo} = OZI.{tipo} || {}`)

### Sem hardcode de framework
- [ ] NENHUMA ocorrência de `'is-invalid'`, `'is-valid'`, `'invalid-feedback'`
- [ ] NENHUMA ocorrência de `'btn btn-primary'`, `'btn-success'`, etc.
- [ ] NENHUMA ocorrência de classes Tailwind hardcoded (`border-red-500`, etc.)
- [ ] Todas as classes vêm de `_classMap(token, fallback)`

### i18n
- [ ] NENHUMA string em PT hardcoded no JS principal
- [ ] Todos os textos usam `_t(key, fallback)` ou `OZI.lang.t(key)`
- [ ] Arquivos `lang/pt-BR.js`, `lang/en.js`, `lang/es.js` existem

### Boot e hooks
- [ ] Boot via `$(function(){ _boot(); })` — sem `if (!OZI.isReady)`
- [ ] `afterRender` registrado como `'{tipo}:{nome}'`

### Components (extra)
- [ ] Adapter registrado em `OZI.modules.validate`
- [ ] Guard de dupla-init no `_init()` (ex: `if (_instances[key]) return`)
- [ ] `destroy()` limpa eventos e DOM gerado

### Behaviors (extra)
- [ ] Bind com namespace: `.off('click.ozi{Nome}').on('click.ozi{Nome}')`
- [ ] Evento nativo disparado com `bubbles: true`

### Registro
- [ ] Entrada no `_pluginMap` do `ozi-conf.js` com deps corretas
- [ ] Posição correta na `_loadOrder`

### Documentação
- [ ] `ozi-ui-docs/dev/{tipo}/ozi-{nome}/description.md` existe e está atualizado
- [ ] `ozi-ui-docs/dev/{tipo}/ozi-{nome}/changelog.md` tem a entrada da versão atual
- [ ] Dívidas resolvidas: marcadas como `Resolvida` em `_meta/dividas-tecnicas.md` com versão
- [ ] Novas dívidas identificadas: adicionadas em `_meta/dividas-tecnicas.md` com ID sequencial
- [ ] Se aprendizado não óbvio: registrado em `ozi-ui-ai/logs/lessons-learned.md`

## Perguntas de revisão
1. O plugin funciona sem jQuery carregado? (não deve — é esperado que dependa)
2. O plugin funciona com `plugins: 'all'` E com seleção manual?
3. O plugin funciona nos 3 temas? (default, bootstrap5, tailwind)
4. O plugin re-inicializa corretamente após Livewire render?
5. O plugin pode ser carregado mais de uma vez sem efeitos colaterais?
```
