# Decisões Arquiteturais — OZI-UI

Decisões tomadas com justificativas. Entender o "porquê" evita desfazer padrões estabelecidos.

---

## #1 — classMap em vez de classes hardcoded

**Decisão:** Plugins nunca escrevem `'is-invalid'` ou `'btn btn-primary'` no código. Usam `_classMap('invalid', 'ozi-invalid')`.

**Por quê:** O plugin precisa funcionar com Bootstrap 5, Tailwind 4 ou CSS próprio. Hardcodar classes de um framework quebraria todos os outros.

**Como funciona:** `ozi-conf.js` mantém presets de tema. Quando `theme: 'bootstrap5'` é configurado, `classMap.invalid` passa a ser `'is-invalid'`. Os plugins leem esse mapa em runtime.

---

## #2 — Singleton guard em todo arquivo

**Decisão:** Todo arquivo começa com `if (window.OziNome) return;`.

**Por quê:** Scripts de plugin podem ser incluídos múltiplas vezes (Blade partials, Livewire re-render, includes duplicados). O guard garante idempotência — o segundo carregamento é descartado silenciosamente.

---

## #3 — Delegação de eventos nos behaviors

**Decisão:** Behaviors fazem bind no `document`, não nos elementos.

```js
$(document).on('click.oziCopy', '[data-ozi-copy]', handler);
```

**Por quê:** Conteúdo dinâmico inserido via Livewire/AJAX não existia quando o behavior carregou. Bind delegado no `document` pega qualquer elemento, presente ou futuro.

**Consequência:** O hook `afterRender` de behaviors não precisa fazer nada — o bind já cobre tudo.

---

## #4 — `$(function(){ })` em vez de `OZI.isReady`

**Decisão:** Plugins usam `$(function(){ _boot(); })` para inicializar, nunca verificam `OZI.isReady`.

**Por quê:** O DOMReady do jQuery já garante que o DOM está pronto. `OZI.isReady` é para código externo que quer saber quando o *plugin inteiro* terminou de carregar — não para os próprios plugins.

---

## #5 — Adapter pattern no ozi-validate

**Decisão:** `ozi-validate` não conhece `ozi-select`, `ozi-autocomplete` ou qualquer outro component. Cada component registra um adapter que ensina o validate como ler seu valor e setar seu estado.

**Por quê:** Desacopla validação de UI. Novos components podem participar da validação sem modificar o validate.

---

## #6 — Registry plugin ↔ adapter no ozi-integrations

**Decisão:** Qualquer um dos dois pode ser registrado primeiro — o casamento acontece automaticamente quando ambos estão presentes.

**Por quê:** A ordem de carregamento de scripts não é garantida em todos os contextos. O registry resolve o problema de ordem sem coordinação explícita.

---

## #7 — Promise chain na boot sequence

**Decisão:** Core systems são carregados em cadeia de Promises, não em paralelo.

**Por quê:** `ozi-conf` precisa estar pronto antes de `ozi-hooks`, que precisa estar pronto antes de `ozi-lang`, etc. Promises garantem a sequência mesmo com scripts carregados dinamicamente.

---

## #8 — i18n com fallback embutido

**Decisão:** Cada plugin tem um helper local `_t(key, fallback)` com fallback PT-BR hardcoded.

**Por quê:** Se o sistema de lang não carregou ainda (ou não está presente), o plugin ainda funciona com textos em PT-BR. Nunca quebra por falta de tradução.

---

## #9 — `_pluginMap` centralizado no ozi-conf.js

**Decisão:** Toda entrada de novo plugin (deps, js, css, lang) é registrada em `_pluginMap` dentro de `ozi-conf.js`.

**Por quê:** Ponto único de verdade para dependências e arquivos. O `ozi-loader` não precisa saber de nada — só recebe a lista resolvida do `ozi-conf`.

**Armadilha:** Arquivos de integração Livewire também devem estar no `_pluginMap`. Esquecer disso impede que o adapter seja carregado.

---

## #10 — OziAssets.php para Laravel

**Decisão:** Projetos Laravel usam `OziAssets.php` + `@oziScripts` em vez de incluir `ozi.js` manualmente.

**Por quê:** Livewire 4 exige que os scripts Livewire rodem antes do boot do OZI (para `window.Livewire` estar disponível). `OziAssets.php` separa o boot em HEAD (imediato) + TAIL (DOMContentLoaded, depois do `@livewireScripts`).

**Padrão no Blade:**
```blade
@oziScripts
<script>oziConf({ theme: 'bootstrap5' });</script>
{{-- restante do head --}}
@livewireScripts
{{-- fim do body --}}
```

---

## #11 — Bridge de hooks unidirecional

**Decisão:** `ozi-loaddata` usa seu próprio sistema de hooks interno (`zldConf.zldHooks`). A bridge conecta esse sistema ao `OZI.hooks` — mas apenas em uma direção.

**Por quê:** Permite que renders disparados pelo `ozi-loaddata` acionem o `afterRender` global, re-inicializando todos os outros plugins. A flag `_running` evita loop infinito.

---

## #12 — Exposição dupla de API

**Decisão:** Todo plugin expõe `OZI.components.nome` E `window.OziNome`.

**Por quê:** `OZI.components.nome` é o acesso canônico moderno. `window.OziNome` existe para compat retroativa com código legado e para uso em contextos onde `OZI` pode não estar acessível.
