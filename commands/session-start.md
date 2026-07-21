# /session-start — Iniciar Sessão OZI-UI

Ao receber este comando, execute as etapas abaixo **em ordem e sem pular nenhuma**, antes de responder qualquer outra coisa.

---

## Etapa 1 — Handoff anterior

Leia `ozi-ui-ai/handoff/current.md` (é um log — a linha 1 é o resumo e o primeiro bloco `# SESSÃO ...` é o mais recente; não precisa ler as sessões antigas abaixo, só se for relevante à tarefa de hoje).
Se o arquivo não existir, registre "sem handoff anterior" e continue.

## Etapa 2 — Contexto do projeto

Leia os arquivos abaixo (nesta ordem):

1. `ozi-ui-ai/context/project-overview.md` — repositórios, ambiente, estado da migração v2
2. `ozi-ui-ai/context/stack.md` — padrão de arquivo JS, convenções

## Etapa 3 — Regras e armadilhas (obrigatório antes de tocar em qualquer código do plugin)

Leia `ozi-ui-ai/context/regras-v2.md` — regras R1–R14 + armadilhas A1–A5. Sem isso, não editar `core/`, `modules/`, `components/` ou `behaviors/`.

## Etapa 4 — Pendências abertas

Leia `ozi-ui-docs/fases-implantacao-v2.md` (painel condensado — vá direto à fase em aberto, não leia o histórico completo das fases já concluídas).

## Etapa 5 — Relatório de início de sessão

Após ler tudo, responda com exatamente este formato:

---

**Sessão iniciada** — [data de hoje]

**Estado anterior (handoff):**
[O que estava em andamento, o que ficou pela metade, alertas — ou "sem handoff anterior"]

**Fase atual (migração v2):**
[fase em aberto conforme `fases-implantacao-v2.md`, ex.: "F5-B — piloto Central RH + corte"]

**Pendências prioritárias:**
- [item 1]
- [item 2]
- [item 3]

**Contexto carregado:**
- [x] handoff/current.md
- [x] context/project-overview.md
- [x] context/stack.md
- [x] context/regras-v2.md
- [x] fases-implantacao-v2.md

**Lembretes:**
- Duas máquinas, mesmos repos via junction: trabalho = `C:/xampp_lite_8_4/www/ozi-ui/...`, casa = `E:/xampp/www/ozi-ui/...`. Resolver a partir da máquina atual — não "corrigir" caminhos `E:/` na doc.
- Fonte da v2 é `ozi-ui-dev-hard` (branch `v2`); `ozi-ui`/`-pkg` é espelho sincronizado, não a origem (armadilha A1).
- Código do **Central RH** (host do piloto) não é deste repositório — confirmar antes de alterar blades/host.

**Pronto.** Pode descrever a tarefa de hoje.

---

$ARGUMENTS
