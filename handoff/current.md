# OZI-UI — Handoff de Sessão
**idDoc:** handoff-current | **Versão:** 1.0 | **Data:** 2026-07-03 (máquina: trabalho C:)

> Arquivo gravado pela IA ao encerrar cada sessão de trabalho no ozi-ui.
> Lido no início da sessão seguinte (casa ou trabalho).

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
- **Escopo fechado:** 9 componentes (select, autocomplete, loaddata, validate, auth, editor, search, check, toggle) + 3 módulos internos; **descontinuados** audio/copy/paste (uso zero)
- **Decisão #13** (nova em `context/decisions.md`): reset global só estrutural (box-sizing+svg) — `ozi-reset.css` v1.0.2
- **Tags `v1-final` publicadas de verdade (2026-07-03, correção pós-auditoria)**: `ozi-ui` (pacote/ozi-core, homologação-produção) em `b5b218c` + branch `v2` criada a partir do `master`, ambas no `origin`; `ozi-ui-dev-hard` em `02893ae`, no `origin`. *A sessão anterior tinha registrado esse item como concluído, mas nem a tag nem a branch `v2` do pacote existiam — só foram criadas/pushadas agora.*
- Unificação `2.0.0-dev`: **adiada** para pós-testes de ambiente (decisão do arquiteto); trabalho permanece no dev-hard
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

## F2 — Migração ✅ **COMPLETA (9 componentes + 3 módulos, 2026-07-04, branch `v2`)**

> **F2 fechada (2026-07-04):** os 9 componentes E os 3 módulos internos do escopo v2 estão em JS puro (zero jQuery). Guard verde (**3 pendentes — só descontinuados**). Varredura de regressão: todos os aceites PASSOU; consumidores (select/autocomplete/loaddata) revalidados após a migração dos módulos-dep. Dívida #12 (suggest/actions) **resolvida**.

| Item | Status | jQuery | Aceite |
|---|---|---|---|
| #1 validate · #2 toggle · #3 loaddata · #4 select | ✅ | 0 | 20/20 · 20/20 · 18/18 · 28/28+5/5 |
| #5 autocomplete · #6 check · #7 search | ✅ | 0 | 27/27 · 17/17+3/3 · 29/29 |
| #8 auth · #9 editor(+md) | ✅ | 0 | 31/31 · 30/30+15/15 |
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
- Guard `check-camadas.sh`: `PENDING_V1` em **3 arquivos** (era 17 no início da F1). Restam **APENAS os descontinuados** — `ozi-audio`, `ozi-copy`, `ozi-paste` (não entram na v2; seguem no guard até a decisão formal de arquivamento no corte F5). **Todo o escopo v2 está fora do guard.**
- Docs espelhados em `ozi-ui-docs/dev/` para todos os 9 migrados (description.md + changelog.md com a v4/v2 de cada, incluindo editor + editor-md).

## Próximo passo recomendado

**🎉 FASE 2 FECHADA** — todo o escopo v2 (9 componentes + 3 módulos) em JS puro, guard só com descontinuados, todos os aceites verdes. Consolidação/revisão feita; dívidas #13/#14 seguem abertas (baixa prioridade), #12 resolvida.

**→ F3 — Temas/CSS** (próxima fase; baixa dificuldade, paralelizável):
- `tokens.css` como **fonte única** de variáveis por tema (default/bootstrap5/tailwind).
- Criar `themes/tailwind/overrides.css` (única lacuna estrutural identificada no plano).
- Revisar `dark.css` dos 3 temas.
- Validar: o **mesmo build JS** serve os 3 temas trocando só `oziConf({ theme })` — os componentes já usam `_classMap`/tokens, então isso deve "só funcionar"; a F3 confirma e preenche lacunas de CSS.
- Atualizar `themes/_template/` como guia de tema novo.
- Arquivos-base: `public/plugins/ozi-ui/themes/{default,bootstrap5,tailwind}/{tokens,overrides,dark}.css` + `classmap.js`.

**Fases seguintes:**
- **F4 — Integrações:** adapter Livewire (`ozi:change`→`wire:model`, timing/debounce); guard LW3 vs LW4; `ozi-hooks` como re-init oficial.
- **F5 — Docs/piloto/corte:** piloto no Central RH (2–3 páginas v2 lado-a-lado com v1), boot duplo, migrar os 2 `x-ui.select2` + os 2 pontos de `ozi:change` jQuery-posicional, corte (remover shims/aliases zld, publicar 2.0.0, arquivar v1, mover projeto p/ `genesis/`). Ponto de não-retorno só aqui.

## ⚠️ Pendências fora do git do ozi-ui

- ~~`centralrh12` UNCOMMITTED na máquina de TRABALHO (C:)~~ **✅ RESOLVIDO (verificado 2026-07-04 na casa/E:)**: o update do pacote chegou e está commitado em `E:/xampp/www/centralrh/centralrh12` (git limpo, branch `master`). `composer.lock` = `ozi-ui/core 1.0.7` (commit `f5e2d205`); `public/plugins/ozi-ui/` publicado (commit `4e402f88`); `shared/css/ozi-reset.css` = v1.0.2. A casa recebeu o update — nada pendente aqui.
- **Mapa real de consumidores `ozi:change` no Central RH** (mais fino que o inventário F0, verificado 2026-07-04): 2 pontos **jQuery-posicionais** `(event, items)` que quebram sem shim ao encerrar o dual-dispatch — `livewire/empresa/modulo/vagas/candidate-list.blade.php:754` e `profile/edit.blade.php:388`; os demais já são **formato v2 (`detail`)**: `components/ui/ozi-autocomplete.blade.php:138` (`addEventListener` nativo) e `livewire/profissional/curriculo/dados.blade.php:73,92` (Alpine `$event.detail.value`). Isso é insumo do piloto F5.
- Boot duplo no `app.blade.php` do Central RH: ainda pendente (fluxo do host; 8 views incluem `footer-vendor-scripts` direto, 4 via `x-app-layout`).
- Validação headless nesta máquina: `msedge --headless=new --virtual-time-budget=8000 --dump-dom <url>` funciona (binário em `C:/Program Files (x86)/Microsoft/Edge/Application/msedge.exe`).
