# Prompt — Integração Livewire

Use quando for implementar, corrigir ou debugar integração de um plugin com Livewire 3 ou 4.

---

```
Vamos trabalhar na integração Livewire do ozi-ui.

## Referências
- Leia `ozi-ui-docs/dev/core/ozi-integrations/description.md`
- Leia `ozi-ui-docs/dev/components/ozi-select/description.md` — tem integração Livewire como referência

## Contexto: Como a integração funciona

### Arquitetura
```
Plugin (ozi-select) registra em OZI.integrations.registerPlugin()
Adapter (ozi-livewire.adapter.js) registra em OZI.integrations.registerAdapter()
→ casamento automático, independente de ordem
→ rescan() disparado no afterRender
```

### Arquivo de integração por plugin
Cada plugin que integra com Livewire tem um arquivo separado:
`components/ozi-{nome}/ozi-{nome}.plugin.js`

**IMPORTANTE:** Este arquivo DEVE estar registrado no `_pluginMap` do `ozi-conf.js`.
Se não estiver no `_pluginMap`, o loader não carrega e a integração silenciosamente não funciona.

### Livewire 3 vs Livewire 4 — diferença de hooks

```js
// Livewire 3
document.addEventListener('livewire:load', function() { ... });

// Livewire 4
document.addEventListener('livewire:init', function() { ... });
```

**Guard obrigatório:** sempre verificar qual versão está presente antes de registrar:
```js
if (window.Livewire && window.Livewire.hook) {
    // Livewire 4
} else {
    // Livewire 3 ou ausente
}
```

### jQuery 3.x — $(fn) é assíncrono
`$(function(){ })` no jQuery 3.x é resolvido assincronamente.
Se o código precisa rodar ANTES que o Livewire inicialize, não use `$(function())` — use
`document.addEventListener('DOMContentLoaded', ...)` ou o mecanismo de `OziAssets.php`.

### Padrão dispatch → setOptions
Para atualizar opções de um plugin via Livewire:
1. Livewire dispara um evento com os dados
2. O adapter captura e chama `OZI.components.{nome}.get(key).setOptions(data)`

## O que você quer fazer?
[DESCREVA: novo plugin, debug de integração existente, migrar LW3→LW4, etc.]

## Sandbox para testar
- Bootstrap 5 + Livewire 4: `dev-bs` (E:/xampp/www/ozi-ui/ozi-ui-dev-bs)
- Tailwind 4 + Livewire 4: `dev-tw` (E:/xampp/www/ozi-ui/ozi-ui-dev-tw) — futuro

## Checklist de debug de integração
- [ ] O arquivo `.plugin.js` está no `_pluginMap`?
- [ ] O adapter está registrado em `OZI.integrations`? (`OZI.integrations.getAdapters()`)
- [ ] O hook Livewire usa o formato correto para a versão instalada?
- [ ] `window.Livewire` está disponível quando o adapter inicializa?
- [ ] O `rescan()` está sendo chamado após o render do Livewire?
```
