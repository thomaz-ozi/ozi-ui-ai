# Prompt — Nova Sessão

Cole este prompt no início de qualquer sessão de trabalho no ozi-ui.

---

## Versão Completa

```
Olá! Vamos trabalhar no projeto ozi-ui.

## Contexto do projeto

**ozi-ui** é um plugin jQuery/JS distribuído via Composer e download direto.
Fornece components, behaviors e modules com suporte a 3 temas (default, Bootstrap 5, Tailwind 4)
e integração com Livewire 3/4.

### Repositórios
- `ozi-ui` (E:/xampp/www/ozi/ozi-ui) — o plugin, público
- `ozi-ui-docs` (E:/xampp/www/ozi/ozi-ui-docs) — documentação interna, 49 arquivos em dev/
- `ozi-ui-dev-hard` (E:/xampp/www/ozi/ozi-ui-dev-hard) — sandbox PHP puro
- `ozi-ui-dev-bs` (E:/xampp/www/ozi/ozi-ui-dev-bs) — sandbox Laravel 13 + Livewire 4 + BS5
- `ozi-ui-dev-tw` (E:/xampp/www/ozi/ozi-ui-dev-tw) — sandbox Laravel 13 + Livewire 4 + TW4

### Ambiente
- Windows 11, XAMPP PHP 8.4.6, MariaDB 11.4
- Deploy: KingHost PHP 8.3, sem Node/Composer no servidor

### Regras arquiteturais essenciais
1. Nunca hardcodar classes de framework — sempre `_classMap('token', 'fallback')`
2. Todo arquivo começa com singleton guard `if (window.OziNome) return`
3. Boot via `$(function(){ _boot(); })` — nunca `if (!OZI.isReady)`
4. Behaviors: bind delegado no `document`, não nos elementos
5. Novos plugins precisam de entrada no `_pluginMap` do `ozi-conf.js`
6. Arquivos de integração Livewire também devem estar no `_pluginMap`

### Documentação de referência
- Arquitetura geral: `ozi-ui-docs/dev/README.md`
- Guia novo plugin: `ozi-ui-docs/dev/_meta/guia-novo-plugin.md`
- Dívidas técnicas abertas: `ozi-ui-docs/dev/_meta/dividas-tecnicas.md` ← leia antes de mexer em qualquer arquivo
- Cada plugin: `ozi-ui-docs/dev/{tipo}/{plugin}/description.md`
- Aprendizados acumulados: `ozi-ui-ai/logs/lessons-learned.md`

Tarefa de hoje: [DESCREVA AQUI O QUE VOCÊ QUER FAZER]
```

---

## Versão Curta (sessões de continuação)

```
Continuando o ozi-ui. Plugin jQuery/JS com classMap por tema, boot em Promise chain,
behaviors com bind delegado no document. Docs em ozi-ui-docs/dev/.

Tarefa: [DESCREVA AQUI]
```

---

## Onboarding (primeira vez com um colaborador)

```
Preciso de ajuda no ozi-ui — um plugin jQuery/JS.

Antes de começar, leia:
1. ozi-ui-docs/dev/README.md (visão geral + conceitos arquiteturais)
2. ozi-ui-docs/dev/_meta/guia-novo-plugin.md (padrões que DEVEM ser seguidos)

Os pontos mais importantes:
- Nunca hardcodar classes CSS de framework (Bootstrap/Tailwind) — usar classMap
- Singleton guard obrigatório em todo arquivo
- Behaviors usam delegação de eventos no document

Tarefa: [DESCREVA AQUI]
```
