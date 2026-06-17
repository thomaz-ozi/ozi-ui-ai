# Lessons Learned — OZI-UI

Aprendizados técnicos acumulados. Registre aqui armadilhas descobertas, soluções não-óbvias e decisões que pareciam erradas mas eram certas.

---

## Formato de entrada

```
### [DATA] — Título curto
**Contexto:** onde aconteceu (arquivo, feature, sandbox)
**Problema:** o que deu errado ou o que não era óbvio
**Solução:** o que resolveu
**Armadilha:** o que evitar na próxima vez
```

---

## Entradas

---

### 2026-06-16 — Arquivos de integração Livewire fora do _pluginMap

**Contexto:** `ozi-select` + integração Livewire

**Problema:** O arquivo `ozi-select.plugin.js` de integração Livewire não estava registrado no `_pluginMap` do `ozi-conf.js`. O loader nunca o carregava, e a integração silenciosamente não funcionava — sem erro no console.

**Solução:** Adicionar entrada no `_pluginMap` para cada arquivo `.plugin.js` de integração.

**Armadilha:** Qualquer novo arquivo de integração (Livewire, Alpine, etc.) DEVE estar no `_pluginMap`. Se estiver ausente, o loader não sabe da existência do arquivo.

---

### 2026-06-16 — jQuery 3.x $(fn) é assíncrono

**Contexto:** Inicialização de adapters Livewire

**Problema:** `$(function(){ ... })` no jQuery 3.x é resolvido de forma assíncrona. Código dentro desse bloco pode rodar DEPOIS que o Livewire já inicializou, perdendo a janela para registrar hooks.

**Solução:** Para código que precisa rodar ANTES da inicialização do Livewire, usar `document.addEventListener('DOMContentLoaded', ...)` diretamente, ou deixar o `OziAssets.php` gerenciar a ordem via `deferredBoot`.

**Armadilha:** Não assumir que `$(fn)` === DOMContentLoaded em timing. Em jQuery 3.x, podem chegar na mesma microtask queue mas com ordem diferente.

---

### 2026-06-16 — Livewire 3 vs Livewire 4: formato de hook diferente

**Contexto:** Adapters de integração

**Problema:** `livewire:load` (LW3) e `livewire:init` (LW4) são eventos diferentes. Um adapter escrito para LW3 silenciosamente não funciona em LW4 e vice-versa.

**Solução:** Guard explícito baseado em `window.Livewire && window.Livewire.hook` para detectar LW4. Caso contrário, assumir LW3 ou ausente.

**Armadilha:** Nunca escrever adapter sem guard de versão — a falha é sempre silenciosa.

---

### 2026-06-16 — Deploy KingHost: sem Node/Composer

**Contexto:** Deploy do oziui.com (ozi-ui-website)

**Problema:** KingHost PHP 8.3 não tem Node.js nem Composer disponíveis no servidor. Tentar rodar `npm run build` ou `composer install` no servidor falha.

**Solução:** Compilar assets localmente (XAMPP), commitar `public/build/` no git e fazer deploy via push. O servidor só serve arquivos estáticos pré-compilados.

**Armadilha:** Nunca adicionar `public/build/` ao `.gitignore` do ozi-ui-website. A constraint de plataforma é permanente.
