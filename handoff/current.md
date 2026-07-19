# OZI-UI — Handoff de Sessão
**idDoc:** handoff-current | **Versão:** 1.3 | **Data:** 2026-07-18 (casa/E:) — F5-B Estágio 1 concluído: dev-tw criado, aceite cross-tema nos 3 sandboxes, fix do tema tailwind, e **pacote v2 gerado como pré-release `2.0.0-beta.1` (no Packagist)**

> Arquivo gravado pela IA ao encerrar cada sessão de trabalho no ozi-ui.
> Lido no início da sessão seguinte (casa ou trabalho).

---

# SESSÃO 2026-07-18 (casa/E:) — F5-B: sandboxes validados + beta.1 publicado

## 1. `ozi-ui-dev-tw` criado (sandbox Tailwind 4) 🆕

- Portadas as demos do `dev-bs` (Bootstrap) para **Tailwind 4**, reescrevendo todo o markup BS→TW: layout, 12 wrappers `tools/*`, 11 views `livewire/tools/*-demo`, páginas home/about/download. Rotas + 11 classes Livewire **idênticas** ao dev-bs (backend agnóstico). Consome o plugin v2 com `oziConf({ theme: 'tailwind' })`.
- **Gotchas (memória `dev-tw-setup.md`):** Tailwind via **CDN browser build** (`@tailwindcss/browser@4`), NÃO Vite — o classMap injeta utilitários em runtime que o build não escaneia. `@oziStyles` NÃO injeta o CSS do tema → o layout linka manualmente `themes/default/{tokens,overrides}` + `tailwind/{tokens,overrides}`. `composer require --no-scripts` deixou o `bootstrap/cache/packages.php` sem o pacote → `@oziScripts` virou literal; fix: `php artisan package:discover`. `.env` do dev-tw ajustado p/ rodar sem DB (este PHP só tem `pdo_mysql`): `SESSION/CACHE=file`, `QUEUE=sync`.

## 2. F5-B Estágio 1 — aceite cross-tema nos 3 sandboxes ✅

- **Os 3 sandboxes rodam o MESMO build v2 canônico** (dev-bs/dev-tw = dev-hard; `diff --strip-trailing-cr` = 0 diffs reais — divergência só de CRLF↔LF).
- **default (dev-hard):** `aceite.html` 10/10 + `aceite-temas.html` 14/14 (troca de tema em runtime). **bootstrap5 (dev-bs)** e **tailwind (dev-tw):** 9/9 componentes bootam via Edge headless, **0 erro JS**. **Varredura visual** (screenshots) do dev-tw: select/autocomplete/editor/audio/auth/check/toggle/search/loaddata **9/9 sem regressão**.
- Decisão #19 (mesmo JS, tema só troca classMap) provada em **apps Livewire reais**, não só nas páginas de aceite.

## 3. 🐛 Bug real do tema tailwind achado+corrigido (overrides.css → v1.0.1)

- `themes/tailwind/overrides.css` aplicava `display:block` em `.ozi-select-control`, **sobrescrevendo o `display:flex`** do componente → ícone/valor/ações empilhavam na vertical. Só o **dev-tw** carrega esse override (bootstrap5 usa classMap + Bootstrap CDN), por isso só ele quebrava. Fix: `display:block` fica **só no `.ozi-autocomplete-input`** (que é `<input>`); o control mantém flex. Persistiu após re-render Livewire (re-init via `ozi-hooks` OK). Corrigido na fonte (**dev-hard**) e propagado idêntico ao **pacote + dev-bs + dev-tw** (os 4 em v1.0.1). *Lição: override de tema com `display:block` quebra controle flex — validar layout VISUAL, não só boot; o dev-tw serve exatamente pra isso.*

## 4. Commits + push (todos em sync com origin)

- **dev-hard** `a4b7f0f` + **pacote `ozi-ui`** `51f7f5e` → fix do tema, na branch **`v2`**.
- **dev-bs** `857fb0b` (sync plugin v1→v2, 39 arq., major +1 por plugin visível: ozi-conf 2.0.3→3.0.0, select 5.0.2→6.0.0) + **dev-tw** `8248567` (sandbox v2, 193 arq.) → na branch nova **`v2-sync`** (defaults `main`/`master` preservadas).
- **template-theme.css** (dev-hard): era resíduo untracked (cópia mal-nomeada do `_template/tokens.css`), mas o usuário o commitou em `53b5ad1 "up"` (agora tracked em origin/v2). Inócuo — deixado a critério (remover via commit ou manter).

## 5. 🎉 Pacote v2 gerado — pré-release `2.0.0-beta.1` (Packagist)

- Commit **`c2d6b9d`** na `v2`: `composer.json version` `2.0.0-dev`→**`2.0.0-beta.1`** (alinha com a tag e o cache-bust do `OziAssets`; o campo fixo conflitaria com a tag se ficasse `-dev`). Tag anotada **`v2.0.0-beta.1`** pushada.
- **Packagist sincronizado** (verificado via `repo.packagist.org/p2/ozi-ui/core.json`): versões `1.0.7` (v1) + **`2.0.0-beta.1`** (v2). Instalável: **`composer require ozi-ui/core:2.0.0-beta.1`** (a constraint de beta já libera a stability por-pacote — sem mexer no `minimum-stability` do host).
- **Reversível** — é pré-release, NÃO o corte. O `2.0.0` final continua sendo o Estágio 3, só depois do piloto. Isso melhora o piloto: dá pra consumir a v2 pela via real (Composer).

---

# SESSÃO 2026-07-17 (trabalho/C:) — F5-B iniciada

## 1. Revisão final dos docs de fases (publicada)

- `fases-implantacao-v2.md` reescrito em markdown válido + linha #10 audio na tabela F2 (dizia 10/10 mas listava 9); `ozi-ui-v2-projeto-implementacao.md` anotado (audio reincluído na linha histórica F0, "12 decisões"→época, links cross-repo viraram referência textual). **Todos os fatos verificados contra o código do dev-hard antes** (armadilha A1): versões de cabeçalho OK, guard verde (2 pendentes = copy/paste), 18 aceites presentes. Commits `69a2731` + `f3acce5` no ozi-docs.

## 2. Pacote sincronizado ✅ (pré-requisito do piloto)

- **Branch `v2` do `ozi-ui` = espelho byte-idêntico do dev-hard `a2e2013`** (`diff -rq` vazio; guard `check-camadas.sh` verde rodado NA árvore do pacote). Commit **`af0b52f`** pushado.
- `composer.json` → **`2.0.0-dev`** na branch (release `2.0.0` só no corte); keywords: `jquery` → `vanilla-js`/`livewire`.
- **`OziAssets.php` revisado** (fecha os diferidos F1/F4): copy/paste fora dos defaults; shims v1 como chaves opt-in + grupo **`shims-v1`** (removível no corte); `ozi-lang.js` no HEAD (com `OZI.lang`+`init()` no immediateBoot, espelhando o `ozi.js`); css de check/toggle que faltavam; listas na load order documentada; injeção de `urlBase` preservada.
- **`OziCheckCommand` reescrito** (fecha o item F1 "fonte única"): estava PODRE (paths v0.x, `ozi-addons/`). Agora `php artisan ozi:check` parseia o `_pluginMap` do `ozi-conf.js` publicado e **falha em qualquer deriva** com o OziAssets (2 sentidos; exceções documentadas: copy/paste descontinuados, `integrations/adapters/`+`shared/`, e `validate.css` — o `css:null` do map é INTENCIONAL, o css é visual do tema default, opt-in). Validado standalone: 37 paths map = 37 assets, zero deriva. `php -l` limpo.
- **Decisão #18 adaptada:** dev-bs/dev-tw não existem na máquina de trabalho — **ficam para a máquina de casa (E:)**.

## 3. Pré-verificação do piloto no Central RH (raiz registrada: `C:/xampp_lite_8_4/www/centralrh/`)

- Workspace próprio (`centralrh12` + `centralrh12-ai` + `centralrh12-docs`, protocolo `/session-start`). ⚠️ `centralrh12` tem **trabalho uncommitted em andamento** (correções RD-1..RD-5 Revenda/Empresa) — piloto não pode misturar com esse diff.
- ✅ Os 2 pontos `ozi:change` jQuery-posicional confirmados nas mesmas linhas (`candidate-list.blade.php:754`, `profile/edit.blade.php:388`).
- 🎉 **Os 2 `x-ui.select2` já saíram** — zero consumidores (matches restantes são exemplos no comentário do componente). Item do piloto já resolvido pelo host; componente pode ser removido.
- 🔍 **Causa raiz do boot duplo identificada:** `app.blade.php` usa `@oziScripts` (linha 61, boot manual) **E** `footer-vendor-scripts.blade.php:21` carrega `ozi.js` (boot automático, `data-navigate-once`) = dois boots por design; + as 8 views que incluem o footer direto (4 via `x-app-layout` = dobro). Corrigir no piloto: escolher UM mecanismo.
- 🔍 **Achado novo:** `livewire/revenda/empresa/form.blade.php:~330-347` tem workaround v1 de re-init (`OziSelect.destroy()` manual + `jQuery.removeData('ozi-select-initialized')` + re-dispatch) — desnecessário na v2 (init idempotente via WeakMap); incluir na limpeza do piloto.

---

# PROJETO ATIVO — Migração v2 (jQuery → JS puro)

**Documento canônico:** `ozi-ui-docs/horizonte/roadmap/ozi-ui-v2-projeto-implementacao.md`
**Inventário do host:** `ozi-ui-docs/horizonte/roadmap/ozi-ui-v2-f0-inventario-centralrh.md`
**Contratos F1:** `ozi-ui-dev-hard/docs/ozi-ui-v2-contratos.md`
**Princípio (decisão do arquiteto):** v2 sem dependência de terceiros; frameworks só em `integrations/`, visual só em `themes/`.

## F0 — Concluída ✅ (2026-07-03)

- Central RH registrado: `www/centralrh/centralrh12` (Laravel 12 + LW4 + BS5, `ozi-ui/core ^1.0`)
- Inventário: só `ozi:change` é escutado (jQuery posicional em **2 arquivos**: `candidate-list.blade.php:754`, `profile/edit.blade.php:388`); `ozi:auth-*` não é consumido (shim dispensável); ZLD = maior consumo (55+ urls, alimenta modal/offcanvas)
- Release drift resolvido: v1.0.7 já existia no Packagist → `composer update` no Central RH (lock 1.0.5→1.0.7) + publicado sincronizado (preservado depois oficializado o hotfix do reset)
- **Escopo fechado:** 9 componentes (select, autocomplete, loaddata, validate, auth, editor, search, check, toggle) + 3 módulos internos; **descontinuados** audio/copy/paste (uso zero) — *nota (2026-07-05): `ozi-audio` foi reincluído no escopo e migrado (decisão #17); descontinuados restam só `ozi-copy`/`ozi-paste`*
- **Decisão #13** (nova em `context/decisions.md`): reset global só estrutural (box-sizing+svg) — `ozi-reset.css` v1.0.2
- **Tags `v1-final` publicadas de verdade (2026-07-03, correção pós-auditoria)**: `ozi-ui` (pacote/ozi-core, homologação-produção) em `b5b218c` + branch `v2` criada a partir do `master`, ambas no `origin`; `ozi-ui-dev-hard` em `02893ae`, no `origin`. *A sessão anterior tinha registrado esse item como concluído, mas nem a tag nem a branch `v2` do pacote existiam — só foram criadas/pushadas agora.*
- Unificação `2.0.0-dev`: **descartada** (decisão #16, 2026-07-05) — versionamento passou a ser **major +1 por plugin**, não versão única; trabalho permanece no dev-hard
- Boot duplo no `app.blade.php` do Central RH: pendente (tarefa do fluxo centralrh; 8 views incluem `footer-vendor-scripts` direto, 4 via `x-app-layout` = footer em dobro)

## F1 — ~80% concluída ✅ (2026-07-03, branch `v2` do dev-hard, commit `0865333`)

- **Contratos** escritos e implementados (`docs/ozi-ui-v2-contratos.md`): eventos (CustomEvent/`detail`/`emit()`), camadas, helpers de transição, init/destroy
- **`ozi-helpers` v1.1.0**: zero jQuery; `toElement`/`toElements` aceitam Element|jQuery|seletor; **`OZI.helpers.emit(el, name, detail)`** = ponto único de emissão do contrato; `runBatch` depreciado p/ v2
- **Guard**: `tools/check-camadas.sh` — jQuery fora de `integrations/` = falha; lista `PENDING_V1` (17 arquivos v1) encolhe a cada migração F2, deve zerar antes do corte
- **Aceite PASSOU** (validado em Edge headless): `public/teste-v2/aceite.html` (raiz) + `teste-v2/tools/aceite-aninhada.html` — 10 checks: isReady, zero jQuery, zero requests, urlBase, zero erros, dual-accept, emit bubbles+detail
- **Achado:** o boot do `ozi.js` NUNCA carregou jQuery (só distribui `core/jquery-3.7.1.min.js`); hooks/loader já eram limpos — as "1 ocorrência" da análise eram comentários

### Falta na F1 (itens menores)
1. `OziAssets.php` derivar do `_pluginMap` (fonte única) — mexe no repo do pacote; fazer junto com o 1º release v2
2. Decisão #12 (exposição dupla `window.OziX`) — recomendação: manter via shim, remover no corte
3. Reavaliar `horizonte/roadmap/pluginconf-melhorias.md` contra o contrato

## F2 — Migração ✅ **COMPLETA (10 componentes + 3 módulos, 2026-07-05, branch `v2`)**

> **F2 fechada (2026-07-05):** os 10 componentes E os 3 módulos internos do escopo v2 estão em JS puro (zero jQuery). `ozi-audio` v4.0.0 (reincluído pela decisão #17) foi o 10º e fechou a fase — aceite headless 26/26 (via CDP, `MediaRecorder`/`getUserMedia` stubados). Guard verde (**2 pendentes — só descontinuados `ozi-copy`/`ozi-paste`**). Varredura de regressão: todos os aceites PASSOU; consumidores (select/autocomplete/loaddata) revalidados após a migração dos módulos-dep. Dívida #12 (suggest/actions) **resolvida**.

| Item | Status | jQuery | Aceite |
|---|---|---|---|
| #1 validate · #2 toggle · #3 loaddata · #4 select | ✅ | 0 | 20/20 · 20/20 · 18/18 · 28/28+5/5 |
| #5 autocomplete · #6 check · #7 search | ✅ | 0 | 27/27 · 17/17+3/3 · 29/29 |
| #8 auth · #9 editor(+md) | ✅ | 0 | 31/31 · 30/30+15/15 |
| #10 audio (v4.0.0, 2026-07-05) | ✅ | 0 | 26/26 (headless CDP) |
| módulo password-rules · suggest · actions | ✅ | 0 | (já vanilla) · isolado · 17/17 |

> **Nota da migração dos módulos (2026-07-04):** `ozi-suggest` v2.0.0 (3 jQuery triviais → querySelector/getAttribute/readyState). `ozi-actions` v2.0.0 (adapters de tema em DOM nativo; **removido o fallback `$(el).modal()` BS4** — modal/offcanvas usam SÓ a API nativa `window.bootstrap.Modal/Offcanvas`; toast default troca `$.fadeOut` por transição de opacidade). Aceites isolados novos: `aceite-suggest.html`, `aceite-actions.html`.

### Detalhe dos componentes migrados

- **#9 `ozi-editor` (+ editor-md)** ✅ (editor.js v4.0.0, editor-md.js v2.0.0) — o maior/último (1566+478 linhas, 57+2 jQuery → zero). **Feito faseado:** editor-md.js primeiro (trivial — só o branch de boot `window.jQuery($fn)`; conversores já eram vanilla), aceite **isolado** `aceite-editor-md.html` **15/15**; depois editor.js. O **motor já era nativo** (Selection/Range/execCommand/contentEditable/sanitização) — a migração trocou o encanamento: build via `createElement`, delegação por `addEventListener`+`closest` rastreada por instância p/ `destroy()`, `.data()`→WeakMap, `:visible`→`style.display`, `.closest(sel,ctx)` 2-arg→`closest`+`contains()`+guard text-node. Fim do dual-dispatch: `ozi:change` só CustomEvent via `emit()`, payload no contrato `{component,name:key,value,type,source}` (antes `{key,value,type}` sem `component` + trigger jQuery). **Sem shim** (verificado: nenhum consumidor jQuery-posicional do `ozi:change` do editor no Central RH). +`ozi:init`/`ozi:destroy`. Adapter validate `nativeElement:true`. `registerConverters` defere `init('md')` via flag `_booted` (sem a fila `$(fn)`). Aceite `aceite-editor.html` (html + integração md, **sem jQuery na página**): **30/30** — inclui o check-símbolo "zero requests a jQuery". **Achado (não corrigido):** `_cleanMd` do md.js usa `/^\n+|\n+$/` sem flag `g` → `htmlToMd` deixa `\n\n` final; comportamento herdado.
- **#8 `ozi-auth`** ✅ (v4.0.0) — 610 linhas, 28 jQuery → zero. **Lacuna do contrato corrigida:** era o único componente sem CustomEvent — os eventos eram jQuery-only (`$form.trigger('ozi:auth-*',[payload])`); agora são CustomEvent via `emit()` com payload no contrato (`value:access`, `source:user|api`), + adicionados `ozi:init`/`ozi:destroy`. **Sem shim** (`ozi:auth-*` não consumido no Central RH). Delegação `input/change/focusout/submit` nativa no document (singleton; `blur`→`focusout`); `$.data`→WeakSet/WeakMap; wrap/after/toggleClass→DOM nativo. `destroy(root)` novo na API. Motor `_oziAuth` inalterado (já era puro — **auth NÃO consome o módulo ozi-password-rules**, tem engine própria; o "depende de password-rules" do plano era só conceitual, sem acoplamento). **Achado:** `pluginConf.auth` tem defaults (`passMin:12`,`passMax:64`,`userCaracter:4`) que reprovam forms sem `data-ozi-auth-pass-min` nem campo user — comportamento config-driven idêntico à v1. Aceite dedicado `public/teste-v2/aceite-auth.html`: **31/31**.
- **#7 `ozi-search`** ✅ (v4.0.0) — 590 linhas, 41 jQuery → zero. Delegação de `input` nativa; TreeWalker/regex já eram nativos; `$.data` → WeakMap/WeakSet (item removido pelo `setItems` não deixa estado órfão); `:visible` → `_isVisible`; `$.contains`→`Node.contains`. Fim do dual-dispatch (`ozi:search-filtered` só CustomEvent via `emit()`, payload adere ao contrato com `value:query`+`source:user|api`; **sem shim** — não é consumido no Central RH, mesma decisão do toggle). i18n: aria-labels da paginação passam por `_t()`. **Achado/FIX:** bug latente da v1 — grupo (`data-ozi-search-group`) ocultado numa busca não reaparecia ao ampliar o termo (`:visible` do item retornava false porque o grupo pai ainda estava `display:none` — chicken-egg); agora decide pelo `style.display` do próprio item. Aceite dedicado `public/teste-v2/aceite-search.html`: **29/29** (achei o boolean-attr armadilha nos fixtures — `data-ozi-search-words`/`-no-filter` exigem `="true"`, idêntico à v1).

- **#5 `ozi-autocomplete`** ✅ commit `df20083` — reaproveitou todos os padrões do `ozi-select` (`_make`, `_classListOp`, delegação nativa, `nativeElement:true`, `emit()` com `source`, o mesmo shim genérico de compat — nenhum shim novo precisou ser criado). Toast do grupo "unique" migrado para Web Animations API. Aceite dedicado `public/teste-v2/aceite-autocomplete.html`: **27/27 de primeira** (os fixtures já nasceram com `id` + `data-ozi-required="true"` explícito, lições da migração anterior).
- **#6 `ozi-check`** ✅ commit `57f978d` — motor funcional (sem instâncias OOP), hierarquia switch→group→item + tristate preservados. Achado novo: o payload original usava a chave `source` para indicar o **nível** (switch/group/item) — colidia com o `source` do contrato v2 (`user`/`api`); renomeada para `level`. O evento jQuery customizado `'oziCheck:initFetched'` (não é `CustomEvent`, só existe no mundo jQuery) saiu do componente e virou shim opcional: `integrations/adapters/ozi-check-v1-events.shim.js`. 2 aceites dedicados: `aceite-check.html` (17/17) + `aceite-check-shim.html` (3/3), ambos de primeira.


- **#1 `ozi-validate`** ✅ commit `7658f23` (+ `v2.1.0` no commit `efc15fd`, ver #4) — motor 100% nativo (querySelectorAll/closest/classList); adapters ainda-v1 (autocomplete/editor/audio) continuam recebendo o elemento envelopado em jQuery via ponte `_wrapLegacy()` (`guard-ok`); adapters v2 (select) marcam `nativeElement: true` para receber Element puro. Aceite dedicado `public/teste-v2/aceite-validate.html`: 20/20.
- **#2 `ozi-toggle`** ✅ commit `30da5a7` — slide/fade migrados de `$.animate`/`slideDown`/`slideUp` para **Web Animations API** (`Element.animate`); fim do dual-dispatch (`_emit` só via `OZI.helpers.emit`, sem shim — `ozi:toggle-*` não é consumido no Central RH). Aceite dedicado `public/teste-v2/aceite-toggle.html`: 20/20 (ressalva de teste — ver lessons-learned).
- **#3 `ozi-loaddata` (+collector)** ✅ commit `ec0096b` — o mais crítico para dados (ZLD); AJAX já era via `fetch`, migração foi DOM/delegação. `zldSafeById` passa a retornar Element nativo (resolve divergência com o alias homônimo do `ozi-helpers`). Não emite CustomEvent próprio. Aceite dedicado `public/teste-v2/aceite-loaddata.html`: 18/18.
- **#4 `ozi-select`** ✅ commit `efc15fd` — o caso-símbolo (1.106 linhas, 62 jQuery), maior risco de regressão de UX. Motor 100% nativo (DOM/delegação consolidada num único listener por instância); `:visible`→`isVisible()`; fim do dual-dispatch com `source:'user'|'api'` novo no `emit()`. **Criado o primeiro shim de compat**: `integrations/adapters/ozi-change-v1-compat.shim.js` (re-emite `ozi:change` como jQuery posicional `(event, items, instance)` para os 2 consumidores do Central RH — opcional, não carregado pelo boot). Exigiu ajuste mínimo no `ozi-validate` (`v2.1.0`: `registerAdapter({ nativeElement: true })`). 2 aceites dedicados: `aceite-select.html` (28/28) + `aceite-select-shim.html` (5/5).
- Guard `check-camadas.sh`: `PENDING_V1` em **2 arquivos** (era 17 no início da F1; `ozi-audio` saiu ao ser migrado em 2026-07-05). Restam **APENAS os descontinuados** — `ozi-copy`, `ozi-paste` (não entram na v2; seguem no guard até o arquivamento formal no corte F5). **Todo o escopo v2 está fora do guard.**
- Docs espelhados em `ozi-ui-docs/dev/` para todos os 10 migrados (description.md + changelog.md com a v4/v2 de cada, incluindo editor + editor-md e audio v4.0.0).

## F3 — Temas/CSS ✅ **CONCLUÍDA (2026-07-05, decisão #19)**

- **Tema = dados, nunca código:** cada tema são 4 arquivos em `themes/{tema}/` — `tokens.css` (variáveis `--ozi-*`; `default` é a fonte única de fallbacks), `overrides.css` (estiliza classes `ozi-*` **só com tokens**), `dark.css` (remapeia tokens via `[data-ozi-theme="dark"]` **e** `@media prefers-color-scheme`), `classmap.js` (tokens → classes do framework).
- Criado `themes/tailwind/overrides.css` (**única lacuna estrutural** do plano); `tailwind/dark.css` → v1.0.1 (bloco `@media` que estava incompleto); `_template/` reestruturado nos 4 arquivos + README.
- **Validado:** o **mesmo build JS** serve os 3 temas trocando só `oziConf({ theme })`. Aceite `aceite-temas.html` **14/14** (overrides tailwind aplicado + classMap troca em runtime).
- ⚠️ Armadilha registrada (A3): os DOIS blocos do `dark.css` (atributo + `@media`) devem ter os MESMOS remapeamentos — o do tailwind havia divergido, corrigido aqui.

## F4 — Integrações ✅ **CONCLUÍDA (2026-07-05, decisão #20)**

- Adapter Livewire `integrations/adapters/ozi-livewire.adapter.js` → **v2.0.0**, dois modos:
  - **A (preferido):** `data-ozi-livewire-native` → seta o valor e dispara `input`+`change` nativos no input `wire:model` (o Livewire trata pela própria máquina, respeitando `.live`/`.debounce`/`.lazy` — resposta ao "timing/debounce" do roadmap).
  - **B (fallback):** `data-ozi-livewire-model` → `component.set()`.
- **Anti-loop pelo contrato:** ignora `e.detail.source === 'api'` (mudança programática) — exatamente o que o campo `source` do contrato v2 (#15) existe para permitir.
- **Re-init pós-morph = `ozi-hooks`** (fontes `livewire3`/`livewire4` com guard entre os formatos de `commit`), nunca o adapter; **não** instalar `MutationObserver` (regra R5). Bridge `zldHooks→OZI.hooks` unidirecional, **marcada para remoção no corte** (F5).
- Substitutos dos behaviors descontinuados: **receitas Alpine** copy/paste (`ozi-ui-docs/dev/_meta/receitas-alpine.md`) — resolvem a dívida técnica #11.
- Aceite `aceite-livewire.html` **9/9** (Livewire simulado, headless).
- *Diferido para a leva do pacote:* revisão do `OziAssets.php` (vive em `ozi-ui/src/`).

## F5-A — Documentação ✅ **CONCLUÍDA (2026-07-05)**

- Guia de migração v1→v2 (`ozi-ui-docs/dev/_meta/guia-migracao-v1-v2.md`) — breaking changes, antes/depois.
- Contrato v2 oficial (`ozi-ui-docs/dev/_meta/contrato-v2.md`) — camadas + eventos (§emit, §source).
- Regras para a IA (`ozi-ui-ai/context/regras-v2.md`) — R1–R14 + armadilhas A1–A5.

## Próximo passo recomendado — **F5-B Estágio 2: piloto no Central RH**

Estágios 0/1 **concluídos** (2026-07-18): 4 ambientes prontos, os 3 sandboxes validados no mesmo build v2, e **`2.0.0-beta.1` publicado no Packagist**. O piloto agora pode consumir a v2 pela via real (Composer).

**Piloto no Central RH** (`E:/xampp/www/centralrh/centralrh12`, workspace próprio `/session-start` — o tracking detalhado vive no `centralrh12-ai`/`-docs`; decisões do plugin voltam pro `ozi-ui-ai`). 2–3 páginas em v2:
- **Como consumir a v2 (2 opções):** (a) **lado-a-lado** — copiar o build v2 p/ `public/plugins/ozi-ui-v2/` + layout de piloto que carrega `plugins/ozi-ui-v2/ozi.js` (auto-detecta urlBase) só nessas páginas, resto v1; ou (b) **switch via composer** numa branch do host — `composer require ozi-ui/core:2.0.0-beta.1` (troca tudo de uma vez).
- **Boot duplo:** manter **`@oziScripts`** (decisão #10) e **remover** o `<script ozi.js>`+`oziConf` do `partials/footer-vendor-scripts.blade.php:21-23` (9 views incluem o footer). Nas páginas-piloto, condicionar o layout p/ um único boot (v1 OU v2).
- **Migrar** os 2 `ozi:change` jQuery-posicional p/ `addEventListener`+`e.detail`: `livewire/empresa/modulo/vagas/candidate-list.blade.php:754` e `profile/edit.blade.php:388` (decisão do usuário: migrar, sem shim).
- **Remover** o workaround de re-init `livewire/revenda/empresa/form.blade.php:337` (`jQuery(rootEl).removeData('ozi-select-initialized')`) — desnecessário na v2.
- **Pré-check:** `centralrh12` (E:) está limpo em `master`, lock `1.0.7`; ⚠️ o `vendor/ozi-ui/core/composer.json` mostra `0.19.3-alpha` (lock diz 1.0.7) — rodar `composer install` de sanidade antes.
- **Aceite do piloto:** zero regressão de dados (ZLD, 55+ URLs), zero listener duplicado (um só `OZI.isReady`).

**Estágio 3 — Corte (ponto de não-retorno, só depois do piloto passar):** tag **`2.0.0`** final (major +1 por plugin, decisão #16); remover shims/aliases `zld` + grupo `shims-v1` + bridge `zldHooks→OZI.hooks`; host → `composer require ozi-ui/core:^2.0`; arquivar v1; mover o projeto p/ `genesis/`.

**Plano detalhado da F5-B:** `C:/Users/Thomaz/.claude/plans/cheerful-greeting-cosmos.md`.

**Dívidas técnicas:** #11 (receitas Alpine, F4) e #12 (suggest/actions, F2) **resolvidas**; #13/#14 seguem abertas (baixa prioridade).

## ⚠️ Pendências fora do git do ozi-ui

- ~~`centralrh12` UNCOMMITTED na máquina de TRABALHO (C:)~~ **✅ RESOLVIDO (verificado 2026-07-04 na casa/E:)**: o update do pacote chegou e está commitado em `E:/xampp/www/centralrh/centralrh12` (git limpo, branch `master`). `composer.lock` = `ozi-ui/core 1.0.7` (commit `f5e2d205`); `public/plugins/ozi-ui/` publicado (commit `4e402f88`); `shared/css/ozi-reset.css` = v1.0.2. A casa recebeu o update — nada pendente aqui.
- **Mapa real de consumidores `ozi:change` no Central RH** (mais fino que o inventário F0, verificado 2026-07-04): 2 pontos **jQuery-posicionais** `(event, items)` que quebram sem shim ao encerrar o dual-dispatch — `livewire/empresa/modulo/vagas/candidate-list.blade.php:754` e `profile/edit.blade.php:388`; os demais já são **formato v2 (`detail`)**: `components/ui/ozi-autocomplete.blade.php:138` (`addEventListener` nativo) e `livewire/profissional/curriculo/dados.blade.php:73,92` (Alpine `$event.detail.value`). Isso é insumo do piloto F5.
- Boot duplo no `app.blade.php` do Central RH: ainda pendente (fluxo do host; 8 views incluem `footer-vendor-scripts` direto, 4 via `x-app-layout`).
- Validação headless nesta máquina: `msedge --headless=new --virtual-time-budget=8000 --dump-dom <url>` funciona (binário em `C:/Program Files (x86)/Microsoft/Edge/Application/msedge.exe`).
