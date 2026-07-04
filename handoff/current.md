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

## F2 — Migração dos componentes 🟠 *(6/9 concluídos, 2026-07-04, branch `v2` do dev-hard)*

- **#5 `ozi-autocomplete`** ✅ commit `df20083` — reaproveitou todos os padrões do `ozi-select` (`_make`, `_classListOp`, delegação nativa, `nativeElement:true`, `emit()` com `source`, o mesmo shim genérico de compat — nenhum shim novo precisou ser criado). Toast do grupo "unique" migrado para Web Animations API. Aceite dedicado `public/teste-v2/aceite-autocomplete.html`: **27/27 de primeira** (os fixtures já nasceram com `id` + `data-ozi-required="true"` explícito, lições da migração anterior).
- **#6 `ozi-check`** ✅ commit `57f978d` — motor funcional (sem instâncias OOP), hierarquia switch→group→item + tristate preservados. Achado novo: o payload original usava a chave `source` para indicar o **nível** (switch/group/item) — colidia com o `source` do contrato v2 (`user`/`api`); renomeada para `level`. O evento jQuery customizado `'oziCheck:initFetched'` (não é `CustomEvent`, só existe no mundo jQuery) saiu do componente e virou shim opcional: `integrations/adapters/ozi-check-v1-events.shim.js`. 2 aceites dedicados: `aceite-check.html` (17/17) + `aceite-check-shim.html` (3/3), ambos de primeira.


- **#1 `ozi-validate`** ✅ commit `7658f23` (+ `v2.1.0` no commit `efc15fd`, ver #4) — motor 100% nativo (querySelectorAll/closest/classList); adapters ainda-v1 (autocomplete/editor/audio) continuam recebendo o elemento envelopado em jQuery via ponte `_wrapLegacy()` (`guard-ok`); adapters v2 (select) marcam `nativeElement: true` para receber Element puro. Aceite dedicado `public/teste-v2/aceite-validate.html`: 20/20.
- **#2 `ozi-toggle`** ✅ commit `30da5a7` — slide/fade migrados de `$.animate`/`slideDown`/`slideUp` para **Web Animations API** (`Element.animate`); fim do dual-dispatch (`_emit` só via `OZI.helpers.emit`, sem shim — `ozi:toggle-*` não é consumido no Central RH). Aceite dedicado `public/teste-v2/aceite-toggle.html`: 20/20 (ressalva de teste — ver lessons-learned).
- **#3 `ozi-loaddata` (+collector)** ✅ commit `ec0096b` — o mais crítico para dados (ZLD); AJAX já era via `fetch`, migração foi DOM/delegação. `zldSafeById` passa a retornar Element nativo (resolve divergência com o alias homônimo do `ozi-helpers`). Não emite CustomEvent próprio. Aceite dedicado `public/teste-v2/aceite-loaddata.html`: 18/18.
- **#4 `ozi-select`** ✅ commit `efc15fd` — o caso-símbolo (1.106 linhas, 62 jQuery), maior risco de regressão de UX. Motor 100% nativo (DOM/delegação consolidada num único listener por instância); `:visible`→`isVisible()`; fim do dual-dispatch com `source:'user'|'api'` novo no `emit()`. **Criado o primeiro shim de compat**: `integrations/adapters/ozi-change-v1-compat.shim.js` (re-emite `ozi:change` como jQuery posicional `(event, items, instance)` para os 2 consumidores do Central RH — opcional, não carregado pelo boot). Exigiu ajuste mínimo no `ozi-validate` (`v2.1.0`: `registerAdapter({ nativeElement: true })`). 2 aceites dedicados: `aceite-select.html` (28/28) + `aceite-select-shim.html` (5/5).
- Guard `check-camadas.sh`: `PENDING_V1` em **10 arquivos** (era 17 no início da F1).
- Docs espelhados em `ozi-ui-docs/dev/{behaviors/ozi-toggle,modules/ozi-loaddata,components/ozi-select,components/ozi-autocomplete,components/ozi-check}/`; `ozi-validate`/`ozi-loaddata`/`ozi-select`/`ozi-autocomplete` também documentam no próprio `dev-hard` (CHANGELOG/README ao lado do JS) — `ozi-check` só tem o espelho em `ozi-ui-docs` (nunca teve README/CHANGELOG próprio).

## Próximo passo recomendado

**F2 #7 — migrar `ozi-search`** (590 linhas, 41 jQuery) na branch `v2`:
TreeWalker/paginação já são nativos — jQuery é praticamente casca de DOM/eventos por cima. Checklist no projeto consolidado §F2 (sem jQuery; emissão só via `OZI.helpers.emit`; init idempotente + destroy; classes `ozi-*`; i18n; espelhar doc; página de aceite). Reaproveitar os padrões consolidados (`_make`, `_classListOp`, `nativeElement:true` no adapter se registrar, `id` + `data-ozi-required="true"` nos fixtures de teste, filtrar `"Script error."` opaco nas checagens de erro). Ao concluir: remover do `PENDING_V1` do guard. Depois: auth (#8, único sem CustomEvent hoje).

## ⚠️ Pendências fora do git do ozi-ui

- **`centralrh12` tem mudanças UNCOMMITTED na máquina de TRABALHO (C:)**: `composer.lock` (1.0.7) + `public/plugins/ozi-ui/` sincronizado + reset.css v1.0.2. Mensagem sugerida: `OZI-UI | atualiza pacote v1.0.5 → v1.0.7 (composer + assets publicados; reset.css v1.0.2)`. Sem esse commit, a casa não recebe o update do host.
- Validação headless nesta máquina: `msedge --headless=new --virtual-time-budget=8000 --dump-dom <url>` funciona.
