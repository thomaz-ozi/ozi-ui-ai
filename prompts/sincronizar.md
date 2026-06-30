# Prompt — Sincronizar Versão

Use este prompt ao iniciar uma sessão de sincronização entre repositórios.

---

## Fluxo oficial

```
dev-hard  →  ozi-ui  →  dev-bs
                ↓
            ozi-ui-docs  (changelogs)
                ↓
             ozi-ui-ai   (contexto + lições + logs)
```

**dev-hard** é a origem. Correções nascem e são validadas aqui (PHP puro, sem framework).
**ozi-ui** é o pacote oficial. Recebe o código validado.
**dev-bs** recebe de ozi-ui — `public/` manualmente, `vendor/` simula Composer.
**ozi-ui-docs** registra o que mudou por plugin, para consumidores e referência futura.
**ozi-ui-ai** garante que a próxima sessão começa com contexto completo.

---

## Prompt de sessão

```
Olá! Vamos sincronizar uma nova versão do ozi-ui.

Repositórios envolvidos:
- ozi-ui          → E:/xampp/www/ozi/ozi-ui
- ozi-ui-dev-hard → E:/xampp/www/ozi/ozi-ui-dev-hard
- ozi-ui-dev-bs   → E:/xampp/www/ozi/ozi-ui-dev-bs
- ozi-ui-docs     → E:/xampp/www/ozi/ozi-ui-docs
- ozi-ui-ai       → E:/xampp/www/ozi/ozi-ui-ai

Nova versão: [EX: 1.0.8]

Por favor, siga o checklist de sincronização abaixo.
```

---

## Checklist

### 1. Identificar o que mudou no dev-hard

- [ ] Quais arquivos JS foram alterados? (comparar cabeçalho `Ver:` com ozi-ui)
- [ ] Quais versões individuais subiram? (ex: ozi-select v5.0.2, ozi-validate v1.0.3)
- [ ] Existe entrada no `docs/changelog-externo.md` do dev-hard? Leia antes de prosseguir.

### 2. Sincronizar → ozi-ui

- [ ] Copiar cada JS alterado do dev-hard para o caminho correspondente em ozi-ui
- [ ] Atualizar `ozi-ui/composer.json` → `"version": "X.X.X"`
- [ ] Atualizar `ozi-ui/public/plugins/ozi-ui/ozi.js` → cabeçalho `Ver:` + `OZI.version`

### 3. Sincronizar → dev-bs

Para cada JS alterado:
- [ ] `dev-bs/public/plugins/ozi-ui/...` (cópia publicada)
- [ ] `dev-bs/vendor/ozi-ui/core/public/plugins/ozi-ui/...` (cópia Composer simulada)
- [ ] `dev-bs/public/plugins/ozi-ui/ozi.js` + `dev-bs/vendor/.../ozi.js`
- [ ] `dev-bs/composer.json` → `"ozi-ui/core": "X.X.X"`

### 4. Atualizar ozi-ui-docs (changelogs)

Para cada plugin alterado, atualizar `ozi-ui-docs/dev/{tipo}/{plugin}/changelog.md`:
- [ ] `dev/core/ozi-ui/changelog.md` (sempre que ozi.js mudar)
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

- [ ] `logs/lessons-learned.md` — adicionar armadilhas novas descobertas nesta versão
- [ ] `context/stack.md` — atualizar se houver mudança de padrão arquitetural
- [ ] `context/decisions.md` — adicionar se houver nova decisão relevante
- [ ] `context/project-overview.md` — atualizar versão corrente do pacote

---

## Verificação final

```
OZI.version em ozi.js        == composer.json version  ✓
dev-bs/composer.json         == nova versão             ✓
changelogs ozi-ui-docs       == entradas adicionadas    ✓
ozi-ui-ai/logs               == lições registradas      ✓
```
