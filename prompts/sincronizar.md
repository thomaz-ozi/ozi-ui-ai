# Prompt — Sincronizar Versão

Use este prompt ao iniciar uma sessão de sincronização entre repositórios.

---

## Fluxo oficial

```
dev-hard  →  ozi-ui (pacote)  →  dev-bs  +  dev-tw
                  ↓
              ozi-ui-docs  (changelogs)
                  ↓
               ozi-ui-ai   (contexto + lições + logs)
```

**`-hard`** é a origem. Correções nascem e são validadas aqui (PHP puro, sem framework).
**`-pkg`** é o pacote oficial. Recebe o código validado — **etapa obrigatória, nunca pule**:
é dele que sai o release, então um fix que não chegar aqui não chega ao consumidor.
**`-bs`** e **`-tw`** recebem do pacote — `public/` manualmente, `vendor/` simula Composer.
**`-docs`** registra o que mudou por plugin, para consumidores e referência futura.
**`-ai`** garante que a próxima sessão começa com contexto completo.

> ⚠️ **Invariante:** a branch `v2` do pacote é **espelho byte-idêntico** do `-hard`.
> Confira com `diff -rq --strip-trailing-cr` antes de fechar a sync.

---

## Dois tipos de sincronização

| Tipo | Quando usar | Versão do pacote |
|---|---|---|
| **A — Release** | Um conjunto de mudanças vira uma versão nova do pacote | **Sobe** (`composer.json` + `ozi.js`) |
| **B — Patch por plugin** | Fix pontual em 1+ plugins, fora de um release | **Não muda** — sobe só a versão do plugin (ex.: select 6.0.0 → 6.0.1) |

No **tipo B**, pule os itens marcados *(tipo A)* e decida à parte se vale gerar um
pré-release (ex.: `2.0.0-beta.2`) para os consumidores receberem o fix via Composer —
sem isso, só chega por cópia manual em `public/plugins`.

---

## Prompt de sessão

```
Olá! Vamos sincronizar uma nova versão do ozi-ui.

Repositórios envolvidos:
- -pkg   → E:/xampp/www/ozi-ui/ozi-ui-pkg
- -hard  → E:/xampp/www/ozi-ui/ozi-ui-dev-hard
- -bs    → E:/xampp/www/ozi-ui/ozi-ui-dev-bs
- -tw    → E:/xampp/www/ozi-ui/ozi-ui-dev-tw
- -docs  → E:/xampp/www/ozi-ui/ozi-ui-docs
- -ai    → E:/xampp/www/ozi-ui/ozi-ui-ai

Tipo: [A = release | B = patch por plugin]
Nova versão: [EX: 2.0.0 — só no tipo A]

Por favor, siga o checklist de sincronização abaixo.
```

---

## Checklist

### 1. Identificar o que mudou no -hard

- [ ] Quais arquivos JS foram alterados? (comparar cabeçalho `Ver:` com o `-pkg`)
- [ ] Quais versões individuais subiram? (ex: ozi-select v6.0.1, ozi-audio v4.0.1)
- [ ] Existe entrada no `docs/changelog-externo.md` do `-hard`? Leia antes de prosseguir.
- [ ] `node --check` em cada JS alterado

### 2. Sincronizar → -pkg (obrigatório, nunca pule)

- [ ] Copiar cada JS alterado do `-hard` para o caminho correspondente
- [ ] **Verificar o espelho:** `diff -rq --strip-trailing-cr` das árvores = **zero**
- [ ] *(tipo A)* `composer.json` → `"version": "X.X.X"`
- [ ] *(tipo A)* `public/plugins/ozi-ui/ozi.js` → cabeçalho `Ver:` + `OZI.version`

### 3. Sincronizar → -bs e -tw

Para **cada** sandbox e **cada** JS alterado:
- [ ] `public/plugins/ozi-ui/...` (cópia publicada — é a que a app realmente serve)
- [ ] `vendor/ozi-ui/core/public/plugins/ozi-ui/...` (cópia Composer simulada)
- [ ] *(tipo A)* `composer.json` → `"ozi-ui/core": "X.X.X"`

> ⚠️ `vendor/` é **gitignored** — não entra no commit, mas é o que mantém a simulação
> Composer honesta (é o motivo de o sandbox existir). Se o `vendor/` estiver uma
> geração inteira atrás (ex.: v1 enquanto `public/` já é v2), **não misture**:
> sincronize a árvore toda ou deixe consistente. Meio a meio é pior que desatualizado.

### 4. Atualizar ozi-ui-docs (changelogs)

Para cada plugin alterado, atualizar `ozi-ui-docs/dev/{tipo}/{plugin}/changelog.md`:
- [ ] `dev/core/ozi-ui/changelog.md` (sempre que `ozi.js` mudar)
- [ ] `dev/components/{plugin}/changelog.md`
- [ ] `dev/modules/{plugin}/changelog.md`
- [ ] `dev/behaviors/{plugin}/changelog.md`

Formato de entrada:
```markdown
## vX.X.X — AAAA-MM-DD

- **[TIPO]** Título da mudança.
  - **Causa raiz:** ...
  - **Fix:** ...
  - **Impacto:** ...
```

### 5. Atualizar ozi-ui-ai

- [ ] `logs/lessons-learned.md` — armadilhas novas descobertas nesta versão
- [ ] `context/stack.md` — se houver mudança de padrão arquitetural
- [ ] `context/decisions.md` — se houver nova decisão relevante
- [ ] `context/project-overview.md` — versão corrente do pacote *(tipo A)*
- [ ] `handoff/current.md` — estado de fim de sessão

---

## Verificação final

```
-pkg é espelho byte-idêntico do -hard    ✓   (diff -rq = 0)
-bs / -tw  public/ == -hard              ✓
OZI.version == composer.json version     ✓   (tipo A)
changelogs em -docs adicionados          ✓
-ai: lições + handoff registrados        ✓
commits feitos nos repos tocados         ✓
```

> **Lição registrada (2026-07-20):** numa sync que pulou o `-pkg`, o pacote ficou
> divergente do `-hard` — e o release sairia **com o bug**. Por isso o passo 2 é
> obrigatório e a verificação do espelho entrou no checklist final.
