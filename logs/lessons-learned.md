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

### 2026-07-03 — Auditar de verdade antes de estimar: o "jQuery no core" eram comentários

**Contexto:** F1 da migração v2 (jQuery → JS puro), dev-hard branch `v2`

**Problema:** A análise estática apontava jQuery no boot do core (`ozi.js`, hooks, loader) — o que dimensionaria a F1 como um refactor pesado do núcleo. Ao inspecionar linha a linha, o boot do `ozi.js` **nunca carregou jQuery** (ele apenas distribui `core/jquery-3.7.1.min.js`; as páginas o incluem por conta própria) e as "1 ocorrência" em hooks/loader eram **comentários**.

**Solução:** O core já estava limpo. O trabalho real da F1 era só **helpers + contrato + verificação** — e a fase encurtou. Feito isso, o aceite passou.

**Armadilha:** `grep`/análise estática conta comentários e strings como "ocorrências". Antes de dimensionar uma fase com base em contagem de matches, confirmar no código se são chamadas reais. Uma auditoria de 10 min mudou o escopo da fase inteira.

---

### 2026-07-03 — Convivência v1↔v2: helpers aceitam Element nativo OU jQuery (dual-accept)

**Contexto:** `ozi-helpers` v1.1.0 (`toElement`/`toElements`)

**Problema:** Durante a F2, componentes v1 (jQuery) e v2 (vanilla) rodam lado a lado. Se os helpers aceitassem só um dos dois tipos, a migração incremental quebraria — todo componente teria que migrar de uma vez.

**Solução:** `toElement()`/`toElements()` normalizam a entrada aceitando **Element nativo, objeto jQuery ou seletor string**. Assim v1 e v2 chamam o mesmo helper sem adaptação, e a migração pode ser feita um componente por vez.

**Armadilha:** Numa migração incremental, a camada compartilhada (helpers) precisa ser **bi-compatível** desde o primeiro passo — senão vira big-bang. `runBatch` ficou depreciado para v2 mas funcional para v1 pelo mesmo motivo.

---

### 2026-07-03 — Contrato de eventos: `emit()` como ponto único + guard de camadas com lista que encolhe

**Contexto:** `docs/ozi-ui-v2-contratos.md`, `OZI.helpers.emit()`, `tools/check-camadas.sh`

**Problema:** Sem disciplina, cada componente emitiria eventos com formato próprio (nomes, payload) e o jQuery vazaria de volta para as camadas já migradas ao longo da F2.

**Solução:**
1. **`OZI.helpers.emit(el, name, detail)`** = ponto único de emissão. Só `CustomEvent` com `bubbles: true` e payload único em `detail: { component, name, value, items?, source }`; nomes `ozi:*` preservados. Valida o payload quando o log está ativo.
2. **Guard `check-camadas.sh`**: falha se jQuery aparecer fora de `integrations/`, mantendo uma lista `PENDING_V1` (17 arquivos v1 tolerados) que **encolhe a cada migração** e deve zerar antes do corte v2.

**Armadilha:** Regressão de arquitetura é silenciosa. Um guard executável + uma allowlist decrescente transforma "não use jQuery aqui" em teste que quebra o build — e mede o progresso da migração objetivamente.

---

### 2026-07-03 — Adapters ainda-v1 exigem ponte explícita (`_wrapLegacy`), não migração forçada em cascata

**Contexto:** F2 #1, `ozi-validate.js` — componentes ainda-v1 (`ozi-select`, `ozi-autocomplete`, `ozi-editor`, `ozi-audio`) registram adapters em `OZI.modules.validate.registerAdapter()` esperando `$el` (chamam `.is()`, `[0]`).

**Problema:** Migrar o motor de `ozi-validate` para JS puro mudaria naturalmente o que é passado para `adapter.match/isValid/getValue/setState` — mas os 4 componentes que ainda não migraram (F2 #4, #5, #8 e o descontinuado #audio) quebrariam silenciosamente se passassem a receber `Element` nativo em vez de jQuery.

**Solução:** `_wrapLegacy(el)` — função interna que envelopa o elemento em `jQuery(el)` **só** na hora de chamar adapters registrados externamente (o adapter nativo interno sempre recebe `Element` puro). Marcada com comentário `guard-ok` (mesmo mecanismo do `runBatch` do `ozi-helpers`), então o guard `check-camadas.sh` não acusa violação de camada mesmo referenciando `window.jQuery`.

**Armadilha:** Numa migração componente-por-componente, todo módulo que **consome** outros componentes ainda-v1 via callback/adapter precisa de uma ponte simétrica à do `ozi-helpers` (`toElement`/`toElements`) — não só o motor interno precisa ficar puro, a *interface* com quem ainda não migrou também. Sem isso, a ordem "risco crescente" do plano quebraria componentes fora de ordem.

---

### 2026-07-03 — Web Animations API (`Element.animate`) não é testável via headless `--virtual-time-budget`

**Contexto:** F2 #2, `ozi-toggle.js` — `slideDown`/`slideUp`/fade de ícone migrados de `$.animate` para `Element.animate()` nativo.

**Problema:** A página de aceite (`public/teste-v2/aceite-toggle.html`) travou esperando o evento `ozi:toggle-change` do fluxo com slide — o `anim.onfinish` (e também `anim.finished.then()`) nunca disparava sob `msedge --headless=new --virtual-time-budget=N --dump-dom`, mesmo com `--run-all-compositor-stages-before-draw`. Isolei o problema fora do componente (animação mínima de teste): `anim.currentTime` avança e `anim.playState` chega a `'finished'`, mas o evento/promise de conclusão não dispara de forma confiável — e o resultado nem é determinístico entre execuções (uma rodada terminou `'finished'`, outra ficou `'running'` no mesmo teste).

**Solução:** Não é bug do código migrado (o motor funciona — `playState`/`currentTime` provam isso). Ajustei a página de aceite para verificar só o efeito **síncrono** do slide (o `display` já é setado antes da animação começar) e usar um trigger **sem** animação para os cliques delegados testados via evento. Documentei a ressalva no CHANGELOG do componente: conclusão visual da animação deve ser conferida manualmente em navegador real antes do piloto (F5).

**Armadilha:** Qualquer migração futura que troque `$.animate`/`slideDown`/`slideUp`/transições CSS por `Element.animate()` (candidatos: `ozi-select`, `ozi-autocomplete`, `ozi-editor` se tiverem esse padrão) vai esbarrar na mesma limitação de teste automatizado headless. Testar o efeito síncrono/estado final via DOM, não o callback de conclusão da animação — e marcar para verificação manual.

---

### 2026-07-03 — Aceite de "zero jQuery" mede requests, não só `window.jQuery`

**Contexto:** `public/teste-v2/aceite.html` (+ variante em rota aninhada), Edge headless

**Problema:** Verificar `window.jQuery === undefined` **não prova** que a página está livre de jQuery — o script pode ter sido requisitado e falhado, ou carregado e limpo depois.

**Solução:** O aceite roda em **Edge headless** (`msedge --headless=new --virtual-time-budget=8000 --dump-dom <url>`) e checa 10 pontos, incluindo **zero requests a jQuery via Performance API** (`performance.getEntriesByType('resource')`), `OZI.isReady`, urlBase auto-detectado, dual-accept dos helpers e o `emit()` borbulhando até o `document` com o payload do contrato. Rodado na raiz E em rota aninhada.

**Armadilha:** Prova de ausência precisa olhar a camada de rede, não só o estado final do `window`. E testar em rota aninhada além da raiz pega o bug clássico de `urlBase` relativo.

---

### 2026-06-20 — urlBase auto-detectado era ignorado por condição errada no bootstrap

**Contexto:** `ozi.js` + `ozi-conf.js` — todos os ambientes (dev-hard, dev-bs, website)

**Problema:** O `ozi.js` já auto-detecta o `_urlBase` correto pelo atributo `src` do próprio `<script>` tag (bloco `[2]`). Porém a condição de fallback no bootstrap era `urlBase === './plugins/ozi-ui/'`, enquanto o default do `ozi-conf.js` era `'/plugins/ozi-ui/'` (sem `./`). A condição nunca batia — o `_urlBase` detectado era sempre descartado, e o path hardcoded do default vencia silenciosamente.

**Solução:**
1. `ozi-conf.js` — `urlBase: null` nos defaults, sinalizando que o valor deve vir externamente
2. `ozi.js` — capturar `_userSetUrlBase` antes do `init()` (verifica se o dev passou `core.urlBase` no `_pendingConf`); se não passou, o `_urlBase` auto-detectado sempre vence

**Armadilha:** A falha era silenciosa — o plugin carregava, mas com o path errado dependendo do ambiente. Em `dev-hard` funcionava por coincidência (default `/plugins/ozi-ui/` == path real). Em ambientes com rotas aninhadas ou `@oziScripts`, o conflito aparecia. Sempre que houver uma condição de fallback verificando um string literal de path, confirmar se o default atual ainda corresponde ao literal.

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

### 2026-06-16 — OziAssets.php não injeta urlBase — SVGs quebram em rotas aninhadas

**Contexto:** `OziAssets.php` + qualquer rota fora da raiz (ex: `/tools/editor`)

**Problema:** `OziConf.init()` define `urlBase` com o default relativo `'./plugins/ozi-ui/'`. Em `/tools/editor`, esse path resolve para `/tools/plugins/ozi-ui/` → 404 em todos os assets (SVGs, CSS). O `ozi.js` não tem esse problema pois auto-detecta o `urlBase` pelo atributo `src` do script tag (sempre absoluto).

**Solução:** Adicionar `OziConf.apply({ core: { urlBase: asset('plugins/ozi-ui') } })` no `$immediateBoot` do `OziAssets.php`, logo após o `OziConf.init()`.

**Armadilha:** Qualquer novo entry point PHP que não carregue `ozi.js` precisa injetar `urlBase` explicitamente. O path relativo só funciona se a página estiver na raiz `/`.

---

### 2026-06-16 — editor-md ausente do $availableScripts do OziAssets.php

**Contexto:** `data-ozi-editor-md` usando `@oziScripts` (Laravel)

**Problema:** `ozi-editor-md.js` não estava na lista `$availableScripts` do `OziAssets.php`. O arquivo nunca era carregado, `registerConverters()` nunca era chamado, e instâncias `[data-ozi-editor-md]` ficavam órfãs silenciosamente — sem erro no console, editor simplesmente não inicializava.

**Solução:** Adicionar `'editor-md' => 'components/ozi-editor/js/ozi-editor-md.js'` no `$availableScripts` logo após `editor`.

**Armadilha:** `OziAssets.php` tem sua própria lista de scripts, independente do `_pluginMap` do `ozi-conf.js`. Qualquer arquivo novo que funciona via `ozi.js` precisa ser adicionado manualmente ao `$availableScripts` para funcionar via `@oziScripts`.

---

### 2026-06-16 — Deploy KingHost: sem Node/Composer

**Contexto:** Deploy do oziui.com (ozi-ui-website)

**Problema:** KingHost PHP 8.3 não tem Node.js nem Composer disponíveis no servidor. Tentar rodar `npm run build` ou `composer install` no servidor falha.

**Solução:** Compilar assets localmente (XAMPP), commitar `public/build/` no git e fazer deploy via push. O servidor só serve arquivos estáticos pré-compilados.

**Armadilha:** Nunca adicionar `public/build/` ao `.gitignore` do ozi-ui-website. A constraint de plataforma é permanente.
