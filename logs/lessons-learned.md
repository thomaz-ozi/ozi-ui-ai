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

### 2026-07-03 — "Script error." opaco no headless é ruído do `file://` + script dinâmico, não um bug do componente

**Contexto:** F2 #3, página de aceite de `ozi-loaddata.js` (`aceite-loaddata.html`)

**Problema:** O aceite reportava um `"Script error."` sem `filename`/`lineno`/`stack` (evento `error` "opaco") logo no boot, antes de qualquer código de teste rodar. Todos os outros 17 checks passavam normalmente, incluindo os que dependiam da mesma lógica supostamente "quebrada" — sinal de que não era uma falha real.

**Solução:** Isolei o problema fora do componente: criei uma página mínima carregando **só** `oziConf({ plugins: ['select'] })` — um plugin v1 que eu nunca toquei nesta migração — e o mesmo `"Script error."` apareceu (3x). Conclusão: é um artefato do próprio mecanismo de carregamento dinâmico de plugins do `ozi.js` (`createElement('script')` + `appendChild`) quando a página é servida via `file://` no headless — o navegador "esconde" detalhes de erros de scripts carregados dinamicamente sob esse protocolo, mesmo sendo mesma origem/mesmo diretório. Ajustei a página de aceite para filtrar especificamente eventos `error` com `message === 'Script error.'` e `filename`/`lineno` vazios, mantendo a checagem de erros reais (que sempre vêm com atribuição completa).

**Armadilha:** Antes de reportar "achei um bug" a partir de um `"Script error."` sem detalhes no headless, isolar a mesma condição num componente **não relacionado** (ou numa página mínima) para confirmar se é ambiente ou código. Toda página de aceite feita a partir daqui em diante deve filtrar esse ruído conhecido (ver `aceite-loaddata.html` para o padrão) em vez de contar como falha ou, pior, tentar "consertar" um código que já está correto.

---

### 2026-07-03 — Adapter que já migrou precisa de opt-out explícito na ponte legada (`nativeElement: true`)

**Contexto:** F2 #4, `ozi-select.js` registrando adapter no `ozi-validate` (já migrado na F2 #1)

**Problema:** O `ozi-validate` v2.0.0 tinha `_wrapLegacy()` que envelopa **todo** elemento em jQuery antes de chamar qualquer adapter externo (ponte para os componentes ainda-v1: autocomplete/editor/audio). Ao migrar `ozi-select` para JS puro, seu adapter passou a esperar `Element` nativo — mas a ponte continuaria envelopando em jQuery, quebrando silenciosamente o adapter do select (`el.hasAttribute is not a function` num objeto jQuery).

**Solução:** Adicionada uma flag opcional no contrato de `registerAdapter()`: `{ name, nativeElement: true, match, isValid, getValue, setState }`. `_wrapLegacy()` virou `_adapterArg(adapter, el)` — só envelopa em jQuery se o adapter **não** declarar `nativeElement: true`. Adapters antigos continuam intactos (comportamento padrão inalterado); o select marca a flag e recebe `Element` puro.

**Armadilha:** Numa migração incremental onde um componente migrado (A) é **consumido** por outro já migrado (B) através de um registry/callback, a ponte de transição em A precisa de um mecanismo de opt-out por registro — não dá pra assumir "quem chama decide o formato" quando várias gerações (v1 e v2) competem pelo mesmo callback. Vale a mesma lição para qualquer módulo com padrão de adapter/plugin registry (`ozi-actions`, `ozi-integrations`) se algum consumidor migrar antes do próprio módulo.

---

### 2026-07-03 — `_parseBool()` não reconhece atributo booleano "vazio" em elementos não-nativos

**Contexto:** F2 #4, página de aceite `aceite-select.html` — testando que `ozi-validate.container()` marca um `ozi-select` obrigatório vazio como inválido

**Problema:** Escrevi o fixture de teste como `<div data-ozi-select="x" data-ozi-select-required data-ozi-required>` (atributo "presença apenas", sem valor) esperando que funcionasse como `required` HTML nativo. `container()` continuava retornando `isValid: true` mesmo com o campo vazio. Investigando, `_parseBool(el, attr, fallback)` tem dois caminhos: para atributos da lista `['required','disabled','checked','multiple','readonly']` ele lê a **propriedade IDL nativa** (`el[attr] === true`) — que não existe em elementos não-form como `<div>` (sempre `undefined`); para qualquer outro atributo (como `data-ozi-required`) ele exige que o **valor literal** seja `'true'`, `'1'`, `'required'` ou igual ao próprio nome do atributo — um atributo escrito sem valor (`data-ozi-required`) tem `getAttribute()` retornando `""` (string vazia), que não bate em nenhum desses casos e portanto avalia como `false`.

**Solução:** Não é bug — confirmei que essa é a **mesma lógica exata da v1** (só trocando `$el.prop`/`$el.attr` por `el[attr]`/`el.getAttribute`). Corrigi o fixture para `data-ozi-required="true"` (valor explícito). Também descobri, no mesmo teste, que o `container()` só coleta um campo se ele tiver `name` **ou** `id` (`if (!name) return;`) — a raiz do `ozi-select` não ganha nenhum dos dois automaticamente, então o dev precisa dar um `id` (convenção comum: igual à chave do `data-ozi-select`) para o adapter ser exercitado via validação de formulário inteiro.

**Armadilha:** Ao escrever fixtures de teste (ou documentação) para atributos `data-ozi-*` booleanos consumidos por `_parseBool()`, sempre usar valor explícito (`="true"`), nunca a forma "presença apenas" do HTML nativo — essa lib não trata os dois formatos como equivalentes fora da lista de booleanos nativos. E lembrar que qualquer componente customizado (`<div data-ozi-*>`) que queira participar da validação de formulário via `container()` precisa de `id` ou `name` no elemento raiz.

---

### 2026-07-04 — Payload legado com chave `source` genérica colide com o `source` do contrato v2

**Contexto:** F2 #6, `ozi-check.js` — payload de `_emit()` usava `source: 'switch'|'group'|'item'` para indicar qual nível da hierarquia disparou a mudança.

**Problema:** O contrato v2 reserva `detail.source` para `'user'|'api'` (distinguir interação humana de chamada programática, usado por adapters Livewire para evitar loop). O `ozi-check` v1 já usava a mesma chave `source` com um significado completamente diferente (nível: switch/group/item). Copiar o payload direto para o `detail` sobrescreveria um pelo outro.

**Solução:** Renomeada a chave original para `level` no `detail` (`{ component, name, value, source: 'user', level: 'switch'|'group'|'item', ... }`), preservando a informação sem colidir com o campo reservado do contrato.

**Armadilha:** Antes de simplesmente repassar um payload v1 para dentro do `detail` do contrato v2, checar se alguma chave do payload original colide semanticamente com `component`/`name`/`value`/`items`/`source` (as reservadas pelo contrato). Migrações futuras (`ozi-search`, `ozi-editor`) devem fazer essa checagem antes de montar o `emit()`.

---

### 2026-07-04 — Evento jQuery customizado (não-DOM) exige shim dedicado, não dá pra só trocar por CustomEvent

**Contexto:** F2 #6, `ozi-check.js` escutava `$(document).on('oziCheck:initFetched', fn)` — um evento **puramente jQuery** (disparado via `$(document).trigger(...)`, não relacionado a nenhum evento DOM nativo), documentado como compat para hosts legados.

**Problema:** Diferente de `ozi:change`/`ozi:toggle-*` (que already são `CustomEvent` nativos com dual-dispatch — dá pra parar de emitir via jQuery e manter só o nativo), este evento **nunca existiu como CustomEvent** — é 100% um mecanismo interno do jQuery. Não existe "versão nativa equivalente" para simplesmente continuar escutando sem jQuery; se um host antigo ainda faz `$(document).trigger('oziCheck:initFetched', [root])`, só um listener registrado via `jQuery(document).on(...)` vai recebê-lo.

**Solução:** Removido o listener do componente (que teria que carregar jQuery só para isso, violando o contrato de camadas). Criado `integrations/adapters/ozi-check-v1-events.shim.js` — arquivo opcional, no-op se jQuery ausente, que escuta o evento legado e delega para `OZI.components.check.refresh()`. Documentado no `description.md` do componente.

**Armadilha:** Ao migrar um componente, distinguir dois tipos de "evento jQuery legado": (1) dual-dispatch de um evento que **também** é `CustomEvent` nativo — aqui basta parar de emitir via jQuery, o shim genérico de `ozi:change` já cobre; (2) evento **exclusivamente jQuery** (custom, sem contraparte `CustomEvent`) — esse exige um shim próprio e específico, não dá pra reaproveitar o shim genérico. `ozi-editor`/`ozi-search`/`ozi-auth` podem ter o mesmo padrão — verificar antes de assumir que "remover jQuery" é só trocar o mecanismo de disparo.

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
