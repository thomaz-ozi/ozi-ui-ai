# Prompt — Nova Chave de Tradução (i18n)

Use quando precisar adicionar, corrigir ou auditar chaves de tradução em qualquer plugin.

---

```
Vamos trabalhar nas traduções do ozi-ui.

## Referência obrigatória
- Leia `ozi-ui-docs/dev/_meta/lang-keys.md` — 70 chaves em 10 namespaces (estado atual completo)

## Localização dos arquivos de lang

Cada plugin tem seus próprios arquivos:
{tipo}/ozi-{nome}/lang/pt-BR.js
{tipo}/ozi-{nome}/lang/en.js
{tipo}/ozi-{nome}/lang/es.js

Shared (chaves comuns entre plugins):
shared/lang/pt-BR.js
shared/lang/en.js
shared/lang/es.js

## Estrutura de um arquivo de lang

```js
(function (window) {
    'use strict';
    var lang = window.OZI && window.OZI.lang;
    if (!lang || !lang.register) return;
    lang.register('pt-BR', {
        {namespace}: {
            chave: 'Texto traduzido'
        }
    });
})(window);
```

## Convenção de chaves

- Namespace = nome do plugin sem `ozi-` (ex: `select`, `validate`, `copy`)
- Chave em inglês, snake_case (ex: `placeholder`, `no_results`, `required`)
- Acesso: `OZI.lang.t('namespace.chave')` ou helper local `_t('namespace.chave', 'fallback PT')`

## Interpolação de variáveis

```js
// definição
{ mensagem: 'Máximo de {{max}} itens' }

// uso
OZI.lang.t('select.mensagem', '', { max: 10 })
// → 'Máximo de 10 itens'
```

## O que você quer fazer?
- [ ] Adicionar nova chave em plugin existente (pt-BR + en + es)
- [ ] Corrigir tradução existente
- [ ] Adicionar novo idioma
- [ ] Auditar chaves sem tradução em algum idioma

## Após qualquer alteração
- Atualizar `ozi-ui-docs/dev/_meta/lang-keys.md` com as chaves novas/alteradas
- Verificar se o helper `_t(key, fallback)` no plugin ainda tem o fallback PT correto
```
