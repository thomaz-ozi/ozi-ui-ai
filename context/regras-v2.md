# Regras para a IA — trabalhando no OZI-UI v2

Vocabulário e regras que a IA deve seguir ao desenvolver **no** `ozi-ui` (o plugin) ou **com** ele
(apps que o consomem). Complementa [`decisions.md`](decisions.md) e [`stack.md`](stack.md).
Docs oficiais: `ozi-ui-docs/dev/_meta/` (contrato v2, guia de migração).

---

## 🧭 Regras de uso (apps que consomem o ozi-ui)

- **R1 — Nunca crie um componente próprio que o ozi-ui já fornece.** Precisa de select? Use
  `ozi-select`. Input com sugestões? `ozi-autocomplete`. Busca/filtro? `ozi-search`. Checkboxes
  hierárquicos? `ozi-check`. Editor? `ozi-editor`. Mostrar/ocultar? `ozi-toggle`. Áudio? `ozi-audio`.
  Não escreva um `<select>` custom, um dropdown à mão, nem um "meu-autocomplete".
- **R2 — Markup declarativo primeiro.** Configure por atributos `data-ozi-*` / `data-zld-*`; só caia
  na API JS quando precisar de controle imperativo.
- **R3 — Consuma eventos por `addEventListener` + `e.detail`** (contrato v2), nunca por
  `$(el).on('ozi:change', (e, items, inst) => …)` (formato v1 posicional — só existe via shim).
- **R4 — Copy/paste saíram.** Use as **receitas Alpine** (`dev/_meta/receitas-alpine.md`), não
  reintroduza `ozi-copy`/`ozi-paste`.
- **R5 — Livewire:** prefira o **modo A** (`data-ozi-livewire-native` → dispatch nativo em
  `wire:model`); deixe o re-init pós-morph com o `ozi-hooks` (não instale `MutationObserver`).

---

## 🛠️ Regras de desenvolvimento (mexer no plugin)

- **R6 — Zero jQuery fora de `integrations/`.** `core`/`modules`/`components`/`behaviors`/`themes`
  não podem referenciar `$`/`jQuery`. Framework (jQuery/Alpine/Livewire) só em `integrations/`;
  visual só em `themes/`. Rode `tools/check-camadas.sh` — tem que passar.
- **R7 — Emita evento SÓ por `OZI.helpers.emit(el, name, detail)`.** Nada de `dispatchEvent`
  direto nem `$(el).trigger()`. `detail` na base `{ component, name, value, items?, source }`;
  `source: 'user'` na interação, `'api'` em `setValue`/programático.
- **R8 — Boot nativo:** `document.readyState`/`DOMContentLoaded`. Nunca `$(function(){})` (o `$(fn)`
  do jQuery 3 é assíncrono — classe de bug eliminada na v2).
- **R9 — Estado por-elemento em `WeakMap`/`WeakSet`**, nunca `$.data()`. `init(root?)` idempotente
  pós-morph (marker `el.__oziXInitialized`) + `destroy()` que remove listeners e emite `ozi:destroy`.
- **R10 — classMap/tokens, nunca classe de framework hardcoded.** `_classMap('invalid','ozi-invalid')`
  + `_classListOp` (split para múltiplas classes do tailwind). Visual novo → token `--ozi-*` no tema.
- **R11 — i18n com fallback embutido** (`_t(key, fallback)` PT-BR) em todo texto visível.
- **R12 — Adapter de validação com `nativeElement: true`** (recebe `Element` nativo).
- **R13 — Espelhe a doc oficial** (`ozi-ui-docs/dev/{tipo}/{plugin}/`) na MESMA sessão em que mexer
  no código. A doc não é tarefa de fim.

---

## 🔢 Versionamento

- **R14 — Major +1 por plugin** no corte para JS puro (NÃO versão única 2.0.0). Ex.: loaddata 4→5,
  select 5→6, check 2→3. `ozi.js` (índice da geração) → **2.0.0**; `ozi-conf` → **3.0.0**.
  Descontinuados (copy/paste) mantêm a linha v1.

---

## 🗺️ Onde as coisas vivem

- **Código v2:** `ozi-ui-dev-hard` (branch `v2`). É a fonte da verdade da migração.
- **Pacote distribuível `ozi-ui`** (alias `ozi-core`): só recebe a v2 no **corte/release**.
- **Doc oficial:** `ozi-ui-docs/dev/` (espelho) + `horizonte/roadmap/` (projeto).
- **Ordem de sync entre repos** (decisão do usuário, "uma coisa de cada vez"):
  `ozi-ai + dev-hard + ozi-docs` → `dev-bs` → `dev-tw + ozi-core`. Nunca tudo de uma vez.

---

## ⚠️ Armadilhas (não repita)

- **A1 — Não confie na memória/doc sobre "está feito"; verifique o código.** Já houve branch `v2`
  do pacote que era placeholder idêntico ao master (o código v2 vive no `dev-hard`, não no pacote).
- **A2 — `data-ozi-required` precisa de valor** (`="true"`), não bare, para o `ozi-validate`
  considerar required.
- **A3 — Os dois blocos do `dark.css`** (`[data-ozi-theme]` e `@media prefers-color-scheme`) devem
  ter os MESMOS remapeamentos (divergiram no tailwind — corrigido na F3).
- **A4 — Arquivo de integração fora do `_pluginMap` falha silenciosamente.**
- **A5 — LW3 `commit (component, succeed)` vs LW4 `commit (payload)`** — precisam de guard de formato.

---

*Fonte oficial das regras: `ozi-ui-docs/dev/_meta/contrato-v2.md` + `guia-migracao-v1-v2.md`.
Lições novas entram em [`../logs/lessons-learned.md`](../logs/lessons-learned.md) na hora.*
