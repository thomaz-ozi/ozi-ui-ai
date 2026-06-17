# Prompt — Novo Tema / Ajuste de classMap

Use quando for criar uma nova variante de tema ou ajustar tokens do classMap existente.

---

```
Vamos trabalhar nos temas do ozi-ui.

## Referências
- Leia `ozi-ui-docs/dev/core/ozi-conf/description.md` — seção "Presets de Tema"
- Arquivo: `ozi-ui/core/ozi-conf.js` — objeto `_themePresets`

## Temas existentes
| Tema | Descrição |
|---|---|
| `default` | CSS próprio do ozi-ui (classes `ozi-*`) |
| `bootstrap5` | Integrado com Bootstrap 5 |
| `tailwind` | Integrado com Tailwind 4 |

## Tokens semânticos atuais
| Token | Uso |
|---|---|
| `invalid` | Estado de erro em campos |
| `valid` | Estado de sucesso em campos |
| `formValidated` | Classe no form após submit com validação |
| `feedback` | Mensagem de erro abaixo do campo |
| `button` | Classe padrão de botão |
| `disabled` | Estado desabilitado |

## O que você quer fazer?
- [ ] Adicionar novo tema (ex: `bulma`, `pico`)
- [ ] Adicionar novo token semântico
- [ ] Ajustar valores de token em tema existente
- [ ] Permitir override pontual via `oziConf({ classMap: { token: 'classe' } })`

## Para adicionar um novo tema
1. Adicionar entrada em `_themePresets` no `ozi-conf.js`
2. Documentar tokens em `ozi-ui-docs/dev/_meta/classMap-tokens.md`
3. Testar em `dev-hard` com todos os components

## Para adicionar novo token
1. Adicionar em `_themePresets` nos 3 temas existentes
2. Atualizar `_classMap()` nos plugins que usarão o token
3. Documentar em `ozi-ui-docs/dev/_meta/classMap-tokens.md`
```
