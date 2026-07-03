# TODO — Discussão parada: absorver superpowers no think-ahead?

> **Status:** discussão adiada, aguardando decisão. Documento de contexto para retomar a conversa.
> **Não é lido automaticamente pelo Claude** — vive em `_inactive/`, que está em `claudeMdExcludes` no `settings.json`, e não é `@`-importado por nenhum arquivo. Serve só como fonte para reabrir o assunto manualmente.
> **Criado em:** 2026-07-02

---

## Pergunta em aberto

O que fazer com o plugin **superpowers** (v6.1.0, `enabledPlugins` no `settings.json`) frente ao meu sistema próprio de regras (`think-ahead` + `decision-protocol` + `observations` + skills `ahead:*`)? Algumas skills do superpowers são redundantes ou conflitam com o meu sistema. A ideia levantada foi **copiar/adaptar/melhorar manualmente as skills e instruções do superpowers para dentro do think-ahead** (um fork seletivo ou total).

## Como a discussão chegou aqui

1. Dúvida inicial: o superpowers traz *instruções sempre ativas + skills* ou só skills?
   - **Resposta:** ambos. A única coisa sempre-ativa é a skill `using-superpowers`, injetada por um **hook `SessionStart`** do plugin (`hooks/hooks.json` → `run-hook.cmd session-start`). O resto são skills sob demanda (`superpowers:*`). A parte sempre-ativa é essencialmente a meta-regra "se há 1% de chance de uma skill se aplicar, você DEVE invocá-la" — mas o próprio texto reconhece que instruções de usuário (CLAUDE.md/rules) têm **precedência** sobre skills.

2. Pergunta: dá para desativar skills individuais que conflitam?
   - **Resposta (confirmada via claude-code-guide):** **não existe** controle nativo por-skill no Claude Code atual.
     - Sem chave `disabledSkills` no `settings.json`. Só `disableBundledSkills` (todas as nativas de uma vez) e toggle de plugin inteiro (`/plugin disable superpowers@...`).
     - `disable-model-invocation: true` no frontmatter de um `SKILL.md` funciona (bloqueia auto-invocação, mantém slash manual), **mas** exige editar o arquivo — e editar dentro de `~/.claude/plugins/cache/.../superpowers/6.1.0/` **é sobrescrito em cada update** do plugin.
     - O hook `SessionStart` não tem supressão seletiva — só `disableAllHooks: true` (global) ou desligar o plugin.

3. Ideia atual (a decidir): trazer manualmente o conteúdo do superpowers para dentro do think-ahead.

## Reframe importante (não esquecer ao retomar)

- **`think-ahead` é sistema de *regras* — sempre carregado** (via `@`-imports no `CLAUDE.md`); cada linha custa contexto em toda sessão.
- **Skills do superpowers são *procedimentos sob demanda*** — só entram no contexto quando invocadas.
- Portanto: copiar o **corpo** de uma skill para dentro de uma **regra** = pagar o custo de contexto dela *sempre*. O lar correto de uma skill copiada é **outra skill** (`ahead:*`), não uma regra. Só a meta-regra de *quando usar* vira regra.

## Mapa das skills do superpowers vs. meu sistema

| Skill superpowers | Relação | Nota |
|---|---|---|
| `brainstorming` | **Conflita** | Provavelmente usa `AskUserQuestion`, proibido pelo `decision-protocol.md` (a confirmar lendo o corpo) |
| `writing-plans` / `executing-plans` | Redundante | Plan mode + meu fluxo já cobrem |
| `finishing-a-development-branch` | Redundante | Sobrepõe `ahead:mr` + convenções de commit |
| `requesting-code-review` / `receiving-code-review` | Redundante | Sobrepõe `/code-review` e o plugin `code-review` |
| `subagent-driven-development` / `dispatching-parallel-agents` | Complementar c/ lacuna | Minha regra "Parallel Analysis for Large Tasks" é fina; pode haver ideia a absorver |
| `systematic-debugging` | **Complementar** | Alta qualidade, sem conflito |
| `test-driven-development` | **Complementar** | Alinhado com meu "Bug Fix Workflow: Test First" |
| `verification-before-completion` | **Complementar** | Reforça "Testing Honesty"/"Honest About Limitations" — tem a disciplina "evidence before assertions" que eu não explicito |
| `using-git-worktrees` | Neutro | Não tenho; é tooling (worktree nativo existe) |
| `writing-skills` | Redundante | Sobrepõe `ahead:instructions` |

## Opções em cima da mesa

1. **Fork wholesale** — copiar tudo para skills/regras próprias e desligar o plugin.
   - Ganho: controle total, durabilidade, mata o hook sempre-ativo.
   - Custo: **imposto de fork** alto — superpowers é ativamente desenvolvido (RELEASE-NOTES de 82KB); forkar congela na 6.1.0 e cada update vira merge manual. Adaptar bem ~12 skills é trabalho real e recorrente. Risco de *piorar* skills maduras e de criar dois sistemas meio-sobrepostos.

2. **Regra de precedência** (mais barato) — uma regra em `~/.claude/rules/` que nomeia as skills conflitantes e declara que meu sistema ganha. Aproveita que skills já cedem a instruções de usuário. Durável, imune a update. Não mexe no plugin.

3. **Absorção em três camadas (recomendação atual)** — tratar cada skill pela relação:
   - **Conflitantes → absorver/reescrever** como skill `ahead:*` alinhada (poucas: `brainstorming`).
   - **Redundantes → não copiar**, referenciar ou ignorar (evita drift).
   - **Complementares de alta qualidade → deixar como plugin**; se tiverem uma *ideia* que falta, absorver **só a ideia** para a regra relevante — não o arquivo.
   - Lema: **minerar o superpowers por técnicas, não por arquivos.**

4. **Não fazer nada** — conviver com a redundância; o custo real de contexto/conflito é baixo hoje.

## Decisões a tomar quando retomar

- [ ] Confirmar o conflito real do `brainstorming` (e de outras suspeitas) **lendo o corpo das skills**, não a descrição.
- [ ] Escolher entre Opção 2 (só precedência) e Opção 3 (absorção seletiva) — provavelmente uma combinação.
- [ ] Decidir se `verification-before-completion` merece virar regra explícita ("evidence before assertions") ou se fica como plugin.
- [ ] Se absorver algo: garantir que vira **skill `ahead:*`** (sob demanda), não regra sempre-carregada, salvo a meta-regra de quando usar.
- [ ] Invocar `ahead:instructions` antes de escrever qualquer regra/skill nova.

## Próximo passo sugerido

Levantamento barato: ler o corpo das skills das camadas "conflitante" e "complementar de alta qualidade" (`brainstorming`, `systematic-debugging`, `test-driven-development`, `verification-before-completion`, `subagent-driven-development`) e devolver, por skill: *conflita / redundante / tem ideia que falta em qual regra minha*. Com isso, decidir Opção 2 vs 3 com base no conteúdo real.
