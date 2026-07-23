# OZI-UI — Handoff de Sessão
**idDoc:** handoff-current | **Versão:** 1.7 | **Data:** 2026-07-23 (trabalho/C:) — F5-B Estágio 2. **3 bugs mexidos hoje:** (1) `lang` do `OziAssets` (dicionário não carregado pelo `@oziScripts`) — verificado + commit + push (`-pkg 7dd871a`); (2) **`ozi-auth` 4.0.1** — 5 chaves de lang faltando, caçado pelo piloto na revenda, corrigido na fonte + espelho + doc, commit + push (`-hard c4ae1fb`, `-pkg 9299c1e`, `-docs 5add4ce`); (3) `init(document)` já estava fechado. **Zero pendência de git.** ⚠️ **Piloto sendo testado no `master` (boot duplo falso) — o correto é `pilot-v2`** (§4). Bug do host à parte (`password_confirmation` no `distribuidora.revenda.edit`). Faltam (não-bugs): decisão `beta.2` × corte e refazer pág.2/3 no `pilot-v2`. Ver §SESSÃO 2026-07-23.

> Arquivo gravado pela IA ao encerrar cada sessão de trabalho no ozi-ui.
> Lido no início da sessão seguinte (casa ou trabalho).

---

# SESSÃO 2026-07-23 (trabalho/C:) — Fechamento dos 2 bugs do v2 (lang verificado + commit + push)

> Sessão curta e cirúrgica. `/session-start` → "vamos terminar os bugs". **Nenhum código novo** — o fix do `lang` já existia (sessão 21/07, uncommitted); esta sessão **verificou, commitou e pushou**.

## 1. Estado achado — 3 peças do bug do `lang` largadas sem commit (sessão 21/07)

A sessão de 21/07 (trabalho/C:) escreveu o fix do `lang` mas **não commitou**. Achado em 3 repos, todas coerentes entre si:
- **`-pkg`** `src/OziAssets.php` (o código) · **`-docs`** `dev/core/ozi-ui/changelog.md` (entrada) · **`-ai`** `logs/lessons-learned.md` (a lição).

## 2. ✅ Bug do `lang` — VERIFICADO (teste de controle) e fechado

**Natureza (recap):** cosmético — `[OZI:lang] t("select.valuePlaceholder"): chave nao encontrada em "en" nem "en"`. **Duas causas empilhadas:** (1) `immediateBoot` roda no HEAD e inicializa `OziLang` com conf vazia; o host chama `oziConf({lang})` **depois** do `@oziScripts` e o `oziConf` só fazia `OziConf.apply()`; (2) `@oziScripts` **não passa pelo `ozi-loader`** → nenhum dicionário de plugin era registrado (só `_baseDicts`).

**Fix (em `OziAssets.php`, vive SÓ no `-pkg`):** `oziConf()` re-inicializa o `OziLang` quando recebe `lang`/`fallbackLang`; novo `$availableLangs` + `resolveLocale()` (deriva do `app()->getLocale()`, normaliza p/ `pt-BR`/`en`/`es`); os `<script>` de dicionário saem **após o `immediateBoot`** (onde `OZI.lang` passa a existir) e **antes** dos plugins. Fica **fora** do `$availableScripts` de propósito (o `ozi:check` descarta `{lang}`).

**Verificação (o método que a própria lição prega — teste de controle):** harness PHP stubando `asset()`/`app()`/`public_path()`, renderizando `scripts()`. **28 checks PASS:** `resolveLocale` nos 5+ casos (`pt_BR`→`pt-BR`, `pt-PT`→`pt-BR`, `fr`→`en`, …), emissão dos 8 dicionários em `pt-BR`, **escopo por plugin** (`scripts(['auth'])` traz `shared`+`auth`, não select), **ordem** `immediateBoot < dicts < plugins`, copy/paste **excluídos**, locale `es`. `php -l` limpo. (Harness no scratchpad, descartável.)

**Confirmado também:** o `init(document)` do §SESSÃO 2026-07-20 já estava fechado/sincronizado — nenhuma ação a mais.

## 3. Commit + push (a pedido do usuário, nesta ordem)

| Repo | Commit | Branch | Push |
|---|---|---|---|
| `-pkg` | **7dd871a** | `v2` | `6780a1f..7dd871a` ✅ |
| `-docs` | **6544e4f** | `master` | `b4c4574..6544e4f` ✅ |
| `-ai` | **68b5ea7** | `main` | `9a2b5da..68b5ea7` ✅ |

Os 6 commits que o handoff v1.6 (casa/E:) listava como "não pushados" **já estavam no origin** nesta máquina (C:) — só os 3 do `lang` estavam pendentes aqui. **Os 3 repos em sync com origin.** (Não toquei `-hard`/`-bs`/`-tw` — nada mudou neles.)

## 4. ⚠️ Piloto pág.2 testado na BRANCH ERRADA (`master`, não `pilot-v2`) — falso "console limpo"

O usuário testou a **página 2** (`empresa/vagas/{id}/candidates`) e reportou "nada anormal", mas o console tinha o sintoma do **boot duplo**: `[OZI:integrations] registerPlugin: "X" já registrado.` para os 9 plugins **duas vezes**, com **`copy`** entre eles.

**Diagnóstico (verificado no host `C:/.../centralrh/centralrh12`):**
- O checkout estava no **`master`** (working tree limpo), **não** no `pilot-v2`. Só existe `origin/pilot-v2` (HEAD `54ef4dd`) — **sem branch local**.
- No `master`: `app.blade.php:61` tem `@oziScripts` **E** `footer-vendor-scripts.blade.php:21` ainda carrega `<script src=".../ozi.js" data-navigate-once>` (auto-boot) → **boot duplo**. O `copy` aparece porque o auto-boot do `ozi.js` carrega tudo do `_pluginMap` publicado (que ainda tem copy) — o `@oziScripts` (v2) não emite copy.
- **CORREÇÃO (2026-07-23, verificado por versão de arquivo):** o publicado do `master` é **v1 INTEIRO** — `ozi.js 1.0.7`, `select 5.0.2`, `auth 3.0.1`, `autocomplete 3.0.3`. (Eu tinha dito "é v2" olhando só o `core/`, que tem arquivos comuns às duas gerações — ERRADO.) Logo o `registerPlugin já registrado` do `master` é o **boot duplo da v1**, e todo teste feito no `master` foi **da v1** (produção), não do piloto v2. O publicado v2 (4.0.x) vive só no `pilot-v2`.
- **`origin/pilot-v2` corrige** (confirmado): footer sem `ozi.js` (comentário `{{-- boot único --}}`), `oziConf` movido p/ o `app.blade.php` depois do `@oziScripts`, os 2 `ozi:change` migrados, workaround removido, `composer 2.0.0-beta.1`.

**Conclusão:** não voltou **nenhum bug do plugin** — o sintoma é wiring do host já resolvido no `pilot-v2`. O teste da pág.2 **não vale**; refazer no `pilot-v2`.

**Ruído do host a ignorar no console:** fontes Poppins (sanitizer), `NS_ERROR_UNEXPECTED` do TinyMCE, `toastr.min.js.map` 404 — nada disso é do ozi.

## 5. 🐛 Bug REAL do plugin caçado no piloto (revenda) — `ozi-auth` 4.0.1 (i18n)

Testando `revenda/empresa/form` (edit + add), o console mostrou `[OZI:lang] t("auth.remaining"): chave nao encontrada em "pt-BR" nem "en"`. **Diferente do bug do `lang` de 21/07:** ali o dicionário não era carregado (`@oziScripts`); aqui o locale está certo (pt-BR ativo, dict carregado) mas **a chave não existe no dicionário**.

**Auditoria (fonte dev-hard):** o `ozi-auth.js` referencia **5 chaves** via `_t()` que **não estavam** em nenhum dos 3 dicionários — caíam no fallback PT embutido (funcionava, mas `en`/`es` viam português + console avisava): `auth.mailRequired` (js:193), `auth.remaining` (448), `auth.exceeded` (450), `auth.summaryValid` (473), `auth.summaryInvalid` (353/474). Resíduo da migração v4.0.0.

**Fix + sync (protocolo `-hard → -pkg → -docs`; bs/tw não existem no C:):**
| Repo | Commit | Push |
|---|---|---|
| `-hard` (fonte) | **c4ae1fb** | ✅ `v2` |
| `-pkg` (espelho byte-idêntico) | **9299c1e** | ✅ `v2` |
| `-docs` (changelog auth) | **5add4ce** | ✅ `master` |

`ozi-auth` **4.0.0 → 4.0.1** (só lang, JS sem mudança de lógica). Verificado: `node --check` limpo, re-auditoria referenciadas × definidas = **0 faltando** nos 3 locales. Nota: `auth.userMin` segue **definido-mas-não-referenciado** (chave morta, inócua). **A mesma lacuna existe na v1** (auth 3.0.1 — foi onde apareceu, pois o teste rodou no `master`/v1; a v1 é congelada, não corrige). **Ainda não propagado ao host:** o `pilot-v2` publica auth **v2 4.0.0** (sem o fix, commitado antes dele) → pra testar, copiar os 4 arquivos v2 (4.0.1) para o publicado do `pilot-v2` **após** o `git switch`, ou levar via `beta.2`/corte.

## 6. Bug do HOST (não é do ozi) — anotar pro Central RH

`revenda edit`: `Livewire: wire:model="password_confirmation" property does not exist on component: [distribuidora.revenda.edit]`. O blade tem `wire:model.defer="password_confirmation"` mas o componente Livewire não declara a propriedade. O `data-ozi-auth-confirm` é só validação client-side do ozi-auth — o `wire:model` é fiação do host. Erraria igual na v1. **Fica pro dev do Central RH** (sessão `centralrh12`), não é regressão da v2.

## ⏳ Retomar por aqui (próxima sessão) — nenhum bug de plugin aberto; falta piloto + decisão

1. **Decisão de arquiteto (pendência #1):** como o fix do `lang` chega ao piloto? O `vendor` do `centralrh12` roda `2.0.0-beta.1` **sem** o fix. Opções: publicar `2.0.0-beta.2` (piloto recebe via `composer update`), ou levar no **corte** `2.0.0`. Até lá o piloto não tem o fix do `lang` (mas é **cosmético**, não bloqueia).
2. **Piloto — ⚠️ ANTES DE TUDO: `git switch -c pilot-v2 origin/pilot-v2`** (não há branch local; testar no `master` dá boot duplo falso — ver §4). DB up + `php artisan serve` (usa `:8080`) + `composer install` se o vendor não estiver no `beta.1` + `php artisan view:clear`. Terminar o teste interativo: **pág.2 refazer** (`empresa/vagas/{id}/candidates`, filtro de badges) + pág.3 (`revenda/empresa/form`, `plano_token`). **Aceite:** `registerPlugin já registrado` + `copy` **sumirem** (boot único). Rodar pela sessão do `centralrh12` (protocolo).
3. **Aceite do piloto verde → corte** (§7 da sessão 19/07): **A1 já feito ✅**, plugin `2.0.0`, host `^2.0`, remover rede v1 (guard → 0), docs → `genesis/`.
4. Pendência 2 do handoff v1.6 (`vendor/` dos sandboxes em v1) segue aberta — baixa prioridade, decidir junto do corte.

---

# SESSÃO 2026-07-20 (casa/E:) — Reorganização dos repos + BUG REAL do v2 achado pelo piloto

> Sessão longa e produtiva, encerrada com o usuário cansado — o alinhamento final (sync + docs + este handoff) foi feito pela IA a pedido dele. **Nada foi pushado.**

## 1. Reorganização de pastas + nova convenção de alias

- `E:/xampp/www/ozi/` → **`E:/xampp/www/ozi-ui/`** (diferencia do projeto irmão `ozi-arch`).
- Pasta do pacote `ozi-ui/ozi-ui` → **`ozi-ui/ozi-ui-pkg`** (nome reflete a finalidade: repositório de publicação). O pacote Composer **segue `ozi-ui/core`**, inalterado.
- **Aliases agora são só o sufixo/papel**, project-agnostic: `-pkg`, `-ai`, `-docs`, `-website`, `-hard`, `-bs`, `-tw`. Motivo do usuário: não ter que lembrar a inicial de cada projeto (`ozi-docs` vs `sn-docs`) — o papel vem do alias, o projeto vem do contexto.
- Propagado em `-ai` (README, aliases, project-overview, prompts, mcp-config) e `-docs` (index, dev/index). Commits: `-ai` **7985366**, `-docs` **ed51040**.
- ⚠️ **Preservado:** `ozi-core` continua sendo o **nome do plugin** na doc técnica (`dev/core/ozi-core/`) — não confundir com o alias antigo. Uma troca em massa teria corrompido a doc.

## 2. 🐛 BUG REAL do v2 — o piloto justificou sua existência

**Sintoma:** a cada `wire:navigate`, 3 erros no console do Central RH:
`DOMException: Document.querySelector: '[object HTMLDocument]' is not a valid selector`
em `ozi-select.js:1189`, `ozi-autocomplete.js:764`, `ozi-audio.js:926`.

**Causa raiz:** `ozi-hooks.js:138` faz `_registry[id](root || document, ctx)` — **converte root nulo em `document`**. Os `init()` decidiam "elemento ou seletor?" por `nodeType`: `(scope.nodeType === 1) ? scope : document.querySelector(scope)`. Como `document.nodeType === 9` (e não 1), caía no ramo de seletor e o `querySelector(document)` estourava.

**Fix:** resolução por **tipo**, não por `nodeType` — `string` → `querySelector`; nó com `querySelectorAll` (Element/Document/Fragment) → escopo direto; senão `null`. No `get()` do select, não-Element retorna `null` (depende de `getAttribute`).

**4 pontos / 3 componentes:** select `init`+`get` → **v6.0.1**; autocomplete `init` → **v4.0.1**; audio `init` → **v4.0.1**.

**`ozi-hooks` NÃO foi alterado** — `toggle`/`auth`/`editor`/`search`/`validate` recebem o mesmo `document` no hook e tratam corretamente; o defeito era só na resolução de escopo desses 3. (Validação cruzada que bateu exatamente com o console.)

**Validado:** `node --check` nos 3 + **erro confirmado eliminado pelo usuário** em `/profile`.

⚠️ **Armadilha de teste (importante):** os erros persistiram após o fix por **cache do browser** — o `?v=2.0.0-beta.1` não muda, então o Ctrl+Shift+R não bastou. O diagnóstico decisivo foi o **número da linha**: o browser reportava 1189 (pré-fix), mas o arquivo corrigido tem o `init` na **1197**. *Sempre conferir o número da linha antes de concluir que o fix falhou.*

## 3. Sincronização completa — protocolo oficial seguido

`-hard` → `-pkg` → `-bs`/`-tw` → `-docs` → `-ai`

| Repo | Commit | Nota |
|---|---|---|
| `-hard` (fonte) | **01c81e4** | origem do fix |
| `-pkg` | **6780a1f** | espelho byte-idêntico restaurado (`diff -rq` = 0) |
| `-bs` | **bb7f6a5** | `public/` idêntico ao `-hard` |
| `-tw` | **2e4deff** | `public/` idêntico ao `-hard` |
| `-docs` | **cf169bb** | changelogs 6.0.1 / 4.0.1 / 4.0.1 |
| `-ai` | *(este commit)* | lessons-learned + protocolo + handoff |

> Nota de processo: a primeira tentativa sincronizou `-bs`/`-tw` **direto do `-hard`, pulando o `-pkg`** — o que deixou o pacote divergente. Como o release sai do pacote, o `2.0.0` teria saído **com o bug**. Corrigido na mesma sessão e o protocolo foi endurecido (ver §4).

## 4. Protocolo de sincronização atualizado (`prompts/sincronizar.md`)

- Passou a incluir o **`-tw`** (o doc só conhecia o `dev-bs`).
- Distinguidos **tipo A (release)** × **tipo B (patch por plugin)** — o nosso caso era B, e o checklist antigo assumia sempre A.
- Passo 2 (`-pkg`) marcado como **obrigatório**, com verificação do espelho no checklist final.
- Alerta sobre a cópia `vendor/` dos sandboxes (ver pendência 2).

## 5. ✅ A1 do corte — PRÉ-VERIFICADO

O re-grep do host inteiro (item 1 do corte, §7 da sessão 19/07) já está feito: **o host está limpo de rede v1**.

- `.on('ozi:` → **zero**
- `removeData('ozi-` → só um **comentário**
- **todos** os listeners de `ozi:change` em formato v2 (`e.detail` / Alpine `$event.detail`)
- superfície de API OZI no host = **3 chamadas** (`OziSelect.get` ×2, `OziSelect.init` ×1) — todas verificadas seguras (chave string e `querySelector` → Element)

## 6. 🚧 Pendências e decisões do arquiteto

1. **Versão do pacote** — publicar `2.0.0-beta.2` para o piloto receber o fix pela via real (Composer), ou levar direto no corte `2.0.0`? Hoje o piloto só tem o fix por **cópia manual** em `public/plugins`, que por isso diverge do `vendor/`. `composer.json` do `-pkg` **continua `2.0.0-beta.1`** (não alterei).
2. **`vendor/` dos sandboxes está em v1** — `select 5.0.2`, `audio 3.0.2`, composer `1.0.0` (bs) / `1.0.7` (tw), enquanto `public/` está v2. **Pré-existente**, desde a sync de 18/07 (que só fez `public/`). Decidir: sincronizar a árvore toda ou deixar consistente — **não misturar** v2 solto dentro de árvore v1.
3. **Bug do `lang`** (cosmético, **não bloqueia** o piloto) — `[OZI:lang] t("select.valuePlaceholder"): chave nao encontrada em "en" nem "en"`. Causa: `OziAssets.php:98` faz `OziLang.init(_c.lang || "en", _c.fallbackLang || "en")` lendo `OZI.conf` **no boot imediato**, mas o host chama `oziConf({lang:'pt-BR'})` **depois** do `@oziScripts` (app.blade.php: 61 vs 66) → conf vazia → `en`/`en`. Decidir onde/como configurar o lang (antes do boot, ou fazer `oziConf` re-inicializar o lang).
4. **Push pendente** — **6 commits locais** não pushados: `-ai`, `-docs`, `-hard`, `-bs`, `-tw`, `-pkg`.

## ⏳ Retomar por aqui (próxima sessão)

1. Decidir as pendências **1** e **3** acima (a 2 pode esperar).
2. Piloto: testar **página 2** (`empresa/vagas/{id}/candidates` — filtro de badges) e **página 3** (`revenda/empresa/form` — `plano_token`).
3. Aceite do piloto verde → **corte** (§7 da sessão 19/07): **A1 já está feito ✅**, falta plugin `2.0.0`, host `^2.0`, remover a rede v1 (guard tem que ir a 0), docs → `genesis/`.
4. Sugestão registrada na lição: criar um aceite que simule `OZI.hooks.afterRender.run()` **sem root** — teria pegado este bug.

---

# SESSÃO 2026-07-19 (casa/E:) — F5-B Estágio 2 destravado: pré-check do host + decisão "piloto rápido → corte"

> Sessão curta de alinhamento/verificação — **nenhum código do plugin ou do host foi alterado**. Saídas: pré-check do host, decisões de sequência, e o corte enfileirado. O piloto em si roda na **sessão do `centralrh12`**.

## 1. Session-start + estado dos repos
- Contexto relido (handoff + docs dos 2 repos). **`ozi-ui-ai` (`main`) e `ozi-ui-docs` (`master`): limpos e em sync com origin** — nada pendente de git neles.

## 2. Pré-check do host (Central RH, E:) ✅ — verificado, não confiado (A1)
- **`centralrh12` git LIMPO** em `master`, em sync com origin. **O "uncommitted RD-1..RD-5" que o handoff v1.3 alertava está RESOLVIDO** — commitado em `3b326768` ("RD-1 a RD-6 | SEGURANÇA Revenda/Empresas"). O piloto **não** vai misturar com aquele diff.
- `composer.lock` = **`ozi-ui/core 1.0.7`** ✅.
- ⚠️ **`vendor/ozi-ui/core/composer.json` = `0.19.3-alpha`** (diverge do lock) → **rodar `composer install` de sanidade ANTES de qualquer edição** (é o passo 0 do piloto).
- `public/plugins/ozi-ui` publicado (v1); constraint `^1.0`.
- `centralrh12` tem **workspace próprio completo** (`centralrh12-ai`/`-docs`, `/session-start` próprio, handoff de 48KB).

## 3. Decisão de workspace — o piloto roda no centralrh12
- A **execução** do piloto (editar blades do host, boot, migrar os `ozi:change`) roda na **sessão do `centralrh12`** (abrir o projeto lá + `/session-start` p/ carregar CLAUDE.md/memória/handoff próprios do host). Esta sessão do **ozi-ui** é onde **as decisões do plugin voltam** e onde **o corte é executado**.

## 4. Estado do "estável" — verificado (o usuário propôs lançar a v2)
- **dev-hard (`v2`):** guard `check-camadas.sh` **VERDE** (2 `PENDING_V1` = só `ozi-copy`/`ozi-paste` descontinuados). Escopo v2 todo fora do guard.
- **pacote (`v2`):** `composer.json` = `2.0.0-beta.1`, tag `v2.0.0-beta.1`. **Packagist** serve `2.0.0-beta.1` + `1.0.7`.
- **Distinção registrada:** "estável nos sandboxes + aceites headless" **≠** "validado em produção". Os **4 riscos que SÓ o piloto enxerga**: (a) boot duplo real (`@oziScripts`+`ozi.js` no footer), (b) os 2 `ozi:change` jQuery-posicionais quebrarem sem shim, (c) **ZLD em escala** (55+ URLs → modal/offcanvas), (d) re-init do Livewire pós-morph real.

## 5. 🧭 DECISÃO do usuário — "piloto rápido → corte" (não pular o piloto)
- Lançar a v2 **via piloto**, não por cima dele. Código já pronto → piloto curto (2–3 páginas). **Se voltar verde → publicar `2.0.0` stable + remover shims + migrar host logo em seguida, com evidência real.** O gate fica, mas curto.
- Reversível × irreversível ficou claro: publicar `2.0.0` *disponível* é barato (ninguém em `^1.0` é forçado); o **corte** (remover shims, migrar host p/ `^2.0`, arquivar v1, mover p/ `genesis/`) é o ponto de não-retorno.

## 6. Briefing do piloto (inputs do lado-plugin — colar na sessão do centralrh12)
0. **Sanidade:** `composer install` (vendor `0.19.3-alpha` → lock `1.0.7`).
1. **Consumir a v2:** (a) lado-a-lado — build v2 do dev-hard → `public/plugins/ozi-ui-v2/` + layout de piloto só nas páginas; ou (b) `composer require ozi-ui/core:2.0.0-beta.1` numa branch do host.
2. **Boot duplo:** manter `@oziScripts`; **remover** `<script ozi.js>`+`oziConf` de `partials/footer-vendor-scripts.blade.php:21-23`.
3. **Migrar** (sem shim) os 2 `ozi:change` posicionais → `addEventListener`+`e.detail`: `livewire/empresa/modulo/vagas/candidate-list.blade.php:754` e `profile/edit.blade.php:388`.
4. **Remover** workaround de re-init: `livewire/revenda/empresa/form.blade.php:337`.
5. Regras: R1 · R3 · R5. **Aceite do piloto:** zero regressão ZLD + zero listener duplicado (um só `OZI.isReady`).

## 7. 🔪 Corte enfileirado (dispara NESTA sessão quando o piloto voltar verde) — ordem importa
1. **A1 antes de remover shim:** re-grep o host inteiro (`\.on\('ozi:`, `ozi:change` posicional, `removeData\('ozi-`) — confirmar que só os 3 pontos do piloto dependiam da rede v1.
2. **Plugin → `2.0.0`:** `composer.json` `2.0.0-beta.1`→`2.0.0` na `v2`; **[sub-decisão em aberto: destino do `master`]**; tag `v2.0.0`; push → Packagist.
3. **Host → `composer require ozi-ui/core:^2.0`**.
4. **Plugin → remover a rede v1:** shims/aliases `zld` + grupo `shims-v1` + bridge `zldHooks→OZI.hooks`; arquivar `ozi-copy`/`ozi-paste` → **guard tem que ir a 0**.
5. **Docs/contexto:** `fases-implantacao-v2` → F5-B/corte concluídos; mover roadmap v2 `horizonte/roadmap/` → `genesis/`; registrar decisão do corte em `decisions.md` (#21?) + `lessons-learned.md` + handoff.

## 8. 🚧 PILOTO EXECUTADO nesta sessão — `centralrh12` branch `pilot-v2` (WIP commit `54ef4dd3`)

> Nota: executei o piloto a partir DESTA sessão (ozi-ui), não da sessão do centralrh12 como §3 previa — mas está tudo commitado e **isolado na branch `pilot-v2`** (`master` intacto em `3b326768`, que já tem o RD commitado). Espelhar no `centralrh12-ai` depois, se quiser.

**Abordagem:** switch via Composer (§6 opção 1b). `composer require ozi-ui/core:2.0.0-beta.1` (resolveu o vendor `0.19.3-alpha`→`2.0.0-beta.1`). Assets v2 publicados em `public/plugins/ozi-ui` via **cópia** de `vendor/.../public/plugins/ozi-ui` (o `vendor:publish` precisa do DB).

**Implementado (5 arquivos):** §2 boot duplo — bloco OZI removido de `footer-vendor-scripts.blade.php`; `oziConf({lang,theme})`+re-init do autocomplete movidos p/ `app.blade.php` **depois** do `@oziScripts`. §3 os 2 `ozi:change` → `addEventListener('ozi:change', e => e.detail.items)` com remove/re-bind seguro (`candidate-list` + `profile/edit`; **atenção:** o usuário também modificou `profile/edit` — minha migração segue intacta). §4 workaround `jQuery.removeData` removido (`revenda/empresa/form`).

**2 bugs achados+corrigidos (lições):**
- **(A) `@diretiva` em COMENTÁRIO Blade é EXECUTADA.** Escrevi `@oziScripts`/`@livewireScripts` em comentário `<!-- -->` → o Blade os rodou → boot duplo + HTML quebrado no `/vagas`. Fix: comentário `{{-- --}}` **sem `@`**. *(nunca escrever `@nome-de-diretiva` em comentário)*
- **(B) `autocomplete.init(document)` quebra na v2.** `document.nodeType===9` (não 1) → o `init` trata como seletor → `querySelector(document)` estoura. Fix no host: `init()` **sem arg** (aí o escopo já vira `document`). **➡️ Follow-up plugin (dev-hard):** endurecer `autocomplete/select .init()` p/ aceitar `document` (nodeType 9) sem quebrar — mesma classe de bug nos dois.

**Verificado:** ✅ server-side tudo verde — sintaxe (`view:cache`), app 200, **boot único** (`app.blade.php` compilado tem `OziAssets->scripts()`=1) + `oziConf` presente, `/vagas` (público) limpo (0 ozi), **API imperativa v2 compatível** (`OziSelect.init/get/destroy`, `clearSelection` — docblock diz "API pública inalterada"). ✅ **Página 1 (`profile`) funcional:** selecionar badges gera os `input[name="badges[]"]` (AL, AP) — prova que o `ozi:change` migrado entrega `e.detail.items`. *(os badges azuis são `badge bg-primary` do host via `renderBadges()`, não do ozi — estilo é customização do Dev, OK.)*

**⚠️ Pré-req operacional:** o **MySQL/MariaDB do host TEM que estar de pé** (3306) — o `AppServiceProvider` faz query no boot; sem DB, `artisan`/`serve`/`view:cache` falham. (Nesta sessão o usuário subiu o DB + `php artisan serve` na :8000.)

## ⏳ Retomar por aqui (próxima sessão) — terminar o teste interativo do piloto
1. **No `centralrh12` (branch `pilot-v2`, DB up + `php artisan serve`):**
   - recarregar `/profile` (Ctrl+Shift+R) e confirmar que o erro `autocomplete.init` **sumiu** (já recompilei o view:cache com o fix);
   - **pegar os DEMAIS erros de console** que o usuário tem (não mostrados) e corrigir — padrão provável: "API v2 chamada como v1";
   - testar **página 2** (filtro de badges em `empresa/vagas/{id}/candidates`) e **página 3** (`plano_token` em `revenda/empresa/form`).
2. **Aceite do piloto:** zero regressão ZLD + zero listener duplicado. Verde → **corte** (§7): re-grep A1, plugin `2.0.0`, host `^2.0`, remover rede v1, docs.
3. **Sub-decisão pendente p/ o corte:** destino do `master` do pacote — virar v2 e arquivar a v1 (tag `v1-final` já existe), ou manter v1 em paralelo.
- **Commit WIP do piloto:** `54ef4dd3` na `pilot-v2` (local; push a critério — remote `oswaldopaulo/centralrh12`). `master` intacto.

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

> **Atualização 2026-07-19** (ver sessão no topo): decisão firmada **"piloto rápido → corte"**. Pré-check do host feito; **falta o `composer install` de sanidade** (vendor `0.19.3-alpha` × lock `1.0.7`). O piloto roda na **sessão do `centralrh12`**; o **corte** já está enfileirado nesta sessão do ozi-ui.

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
