# Prompt — Debug de Plugin

Use quando um plugin não está funcionando como esperado.

---

```
Preciso debugar o plugin `ozi-{nome}` no ozi-ui.

## Sandbox de teste
[ex: dev-hard (PHP puro) | dev-bs (Laravel + Livewire 4 + BS5)]

## O que deveria acontecer
[descreva o comportamento esperado]

## O que está acontecendo
[descreva o comportamento atual — erro no console, sem resposta, comportamento errado]

## Referências
- Leia `ozi-ui-docs/dev/{tipo}/ozi-{nome}/description.md`
- Leia `ozi-ui-docs/dev/{tipo}/ozi-{nome}/changelog.md`

---

## Checklist de diagnóstico — comece por aqui

### 1. O plugin carregou?
```js
console.log(window.OziNome);         // undefined = não carregou
console.log(OZI.components['{nome}']); // undefined = não registrou
console.log(OZI.isReady);            // false = boot não terminou
```

### 2. O plugin está no _pluginMap?
Verificar em `ozi-ui/core/ozi-conf.js` se existe entrada com o nome correto.
Se não estiver, o loader nunca vai carregá-lo.

### 3. As dependências foram resolvidas?
```js
console.log(OZI.conf);               // ver plugins carregados
OziConf.getLoadList();               // lista na ordem de carregamento
```

### 4. O atributo HTML está correto?
```html
<!-- component -->
<div data-ozi-{nome}="minha-key"></div>

<!-- behavior -->
<button data-ozi-{nome}="valor"></button>
```

### 5. Dupla inicialização?
O singleton guard (`if (window.OziNome) return`) pode estar bloqueando.
Verificar se o arquivo foi incluído mais de uma vez.

### 6. Conteúdo dinâmico (Livewire/AJAX)?
O `afterRender` está registrado? Após render, rodar manualmente:
```js
OZI.hooks.afterRender.run(document.getElementById('container'));
```

### 7. Integração Livewire não dispara?
- O arquivo `.plugin.js` está no `_pluginMap`?
- O hook usa o formato correto para a versão instalada? (LW3: `livewire:load` / LW4: `livewire:init`)
- `window.Livewire` estava disponível quando o adapter inicializou?

### 8. classMap retornando vazio?
```js
console.log(OZI.conf.classMap);      // ver tokens resolvidos
console.log(OZI.conf.theme);         // ver tema ativo
```

---

## Logs úteis
Ativar log do OZI para ver o boot completo:
```js
oziConf({ core: { log: true } });
```
```
