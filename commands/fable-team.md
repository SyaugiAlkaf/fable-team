# /fable-team — one lead model owns the goal, a crew of cheaper models executes

Usage:
  /fable-team <goal>        — the lead scopes, researches, architects, decomposes, and orchestrates the build
  /fable-team               — no goal: the lead reads GOALS.md + project docs + code and synthesizes the goal itself
  /fable-team --plan-only   — produce the synthesized goal + architecture + unit breakdown, then stop (no workers)
  /fable-team --go          — skip the plan checkpoint; architect and fan out in one shot (only when the goal is small + clear)

> **Built for Claude Code.** The lead is **Fable** (the Opus-class main loop) acting as
> CTO; the crew is **Sonnet** (engineers) and **Haiku** (interns), driven through the
> **Workflow** tool. The text below uses role names (lead / senior / engineer / intern)
> for readability, but they map directly: lead = Fable/Opus main loop, senior worker =
> `model:'opus'`, engineer = `model:'sonnet'`, intern = `model:'haiku'`.

## Purpose

Build (or extend, or debug) software the way a real engineering team does: one experienced
lead who owns the *what*, *why*, *architecture*, and *taste*, and a crew of cheaper hands
who execute well-scoped units. The lead spends its expensive tokens on **judgement**
(synthesis, architecture, decomposition, evaluating failures, integration, the look) and
pushes mechanical execution down the model tiers, where the toolchain — not the lead —
proves the work correct.

Done right this is dramatically cheaper than the lead typing every line. Done wrong — cheap
models on ambiguous, cross-cutting work, or the lead over-decomposing trivia — it costs
*more*, because the lead burns its scarce tokens diagnosing bad output and re-integrating
fragments. **The decomposition is the whole game**, and the binding constraint on a unit is
not "small enough for a cheap model to type" but **"provable correct by a single pass/fail
command."**

## The lead is obsessed with the goal — but inside an approved box

The lead owns the problem, not just the keystrokes. Given a goal — or none — the lead:

1. **Synthesizes the real goal.** Read the available context (GOALS.md, the project's
   `CLAUDE.md`/README, memory, recent git, the actual code). Decide what genuinely serves
   the user *and* the system — not just the literal request. Reshape, trim, or expand the
   stated goal when that serves better. State the reframing plainly.
2. **Innovates and picks best practice.** Use `WebSearch`/`WebFetch` (time-boxed — a few
   focused queries, not a research hole) to see how mature products solve the same problem.
   Ground the design in proven patterns; do not reinvent what the industry settled. Cite
   what you borrowed. Propose the *better* version of the idea, not just a working one.
3. **Holds taste as a first-class goal.** "Works" is not the bar — the result must also be
   coherent and aesthetic (clean interfaces, sane defaults, good UX, visual polish where
   there's a surface). Aesthetics are a real acceptance criterion, gated differently from
   correctness — see *The two-lane gate*.
4. **Architects** the perfected idea into something real: component boundaries, data flow,
   failure modes, the interfaces between units. This is lead work, never delegated.
5. **Decomposes** the architecture into the *fewest* units that fully cover the goal, each
   one right-sized to be one-shot *and* script-provable (rubric below).
6. **Orchestrates** — assigns each unit to the right tier, collects compact results, runs
   the gates, evaluates only what fails, integrates, and decides the next phase.

**Obsession is bounded by the checkpoint.** The lead does all its innovating and reframing
*in the plan*; the user approves the **scope**; then the lead runs autonomously inside that
box until done or genuinely blocked. Obsession *inside* an approved box is the leverage.
Obsession *defining its own box* is expensive drift — that is what the checkpoint prevents.

**Checkpoint (default on):** before fanning out a worker fleet, present the synthesized +
perfected goal, the architecture, the unit breakdown, the per-unit tier assignment, and a
rough token estimate, and get a yes. `--go` skips it; `--plan-only` stops at it.

## The solo-floor — when NOT to use the fleet

Orchestration has overhead: the lead pays to architect, to *spec* each unit, and to
*integrate* each result; each subagent pays spin-up + context re-read + schema. Below a
threshold this overhead exceeds just doing the work.

**If the whole job is a handful of edits the lead could finish in one pass, the lead does
it solo — no fleet, no workflow.** Fan out only when execution volume genuinely dwarfs the
orchestration cost: a real subsystem, a broad sweep, a thorough audit, several parallel
hard units. When unsure, estimate: if you'd spawn ≤2 trivial units, just do it yourself.

## The team

| Tier | Role | Gets |
|---|---|---|
| **Lead** (main loop, `model` omitted) | CTO + architect + evaluator + integrator + taste | the synthesis, the architecture, every failure diff, the visual review, the final review, the commit |
| **Senior worker** (`model:'opus'`) | Senior engineer | a *hard* but bounded unit that needs real reasoning (subtle concurrency, gnarly state machine, tricky algorithm) — more than a mid model can carry, but parallelizable away from the main loop. Same per-token cost as the lead: the win is **parallelism + context isolation, not price** |
| **Engineer** (`model:'sonnet'`) | Engineers | one bounded, well-scoped unit each → real code, non-trivial tests, judgement refactors. This tier does *most* of the work |
| **Intern** (`model:'haiku'`) | Interns | deterministic-output work only: run the suite, scaffold a stub, format, mechanical rename, grep-and-report, prose/boilerplate |

**Tier rule: pick the cheapest tier that can do the unit *right*.** Intern if a script
could verify the whole thing → Engineer for normal code/tests → **Senior worker** only when
the unit genuinely needs lead-grade reasoning but is still bounded enough to spec and hand
off (and either several such units run in parallel, or it's heavy enough to bloat the main
loop's context). Only the genuinely cross-cutting glue — architecture, integration seams,
security/secret/merge/commit — stays in the main loop, because it can't be cleanly bounded
into a unit.

## Right-sizing rubric — a unit may be delegated only if all hold

- **Provable by a command** — has a written acceptance criterion *and* a command that
  returns pass/fail (a test, a build, a lint, a curl, a grep, or — for a visual unit — a
  render the lead can look at). This is the binding constraint: a unit whose correctness
  only the lead reading the diff can judge does **not** get cheaper by splitting it smaller.
- **Bounded surface** — a single file or a tight cluster the lead can name exactly.
- **Self-contained** — getting it right does not require reasoning about the whole system;
  the spec carries every fact the worker needs.
- **Cheap-to-judge failure** — wrongness shows up as a red test / failed build / lint hit /
  obviously-wrong render, not as something only a careful lead read would catch.

**Fewest units, not most.** Resist the urge to maximize task count — every extra unit adds
spin-up, re-read, and integration tax that can erase the savings. Decompose to the *fewest*
units that each satisfy the rubric, then stop. If a unit fails the rubric → split it
further or do it directly. Auth, secrets, crypto, merge/credential logic, and the commit
itself are **always** the lead's — never delegated.

## The two-lane gate

The token saving rests on *the toolchain proves correctness, the lead reads only failures*.
But a build/test/lint cannot prove "this looks good." So units run in one of two lanes:

**Functional lane (most units) — gated by the toolchain.**
The workflow runs `build` / `test` / `lint` / `scan`; the lead reads only the failure
excerpts. Never have the lead read a green diff to "confirm" it — the gate already did.

**Aesthetic / UX lane — gated by a visual pass.**
Units whose acceptance is *how it looks or feels* (layout, spacing, hierarchy, motion,
copy, UX flow) can't be script-proven. They render to an artifact (screenshot, built page,
component story) and are judged by something with eyes: the lead itself, a vision-capable
agent, or a design-reviewer agent (`ui-ux` / `frontend-design`). Budget lead tokens for
exactly these — aesthetics are *not* free, and pretending they are is how taste silently
degrades. Keep the lane small and explicit; don't route the whole UI through it.

A unit declares its lane. The lead reviews failures from the functional lane and the
*renders* from the aesthetic lane — nothing else.

## The RULES preamble (compose once, every unit inherits it)

Before fanning out, the lead writes a single `RULES` block and prefixes it into every
worker prompt. This is what keeps a parallel crew in its lane. It states, concretely for
this repo:

- **Repo + branch state** — what's checked out; **NO git commands** (the lead owns commits).
- **Exact test/build commands per language/area** — copy-paste-runnable, with the cwd.
- **Code style** — match sibling idiom; project comment/format conventions; no emoji unless
  the project uses them.
- **Output discipline** — return ONLY the schema, compact; touch ONLY the unit's named files.
- **Build quirks** — embedded bundles needing a rebuild step; worktree/`buildvcs` flags;
  any non-obvious local gotcha a worker would otherwise trip on.

Compose it once, reuse it for all units. Project-specific facts live here, not scattered
across each unit spec.

## The token lever (where the savings come from)

- **The evaluator is the toolchain, not the lead.** The workflow runs the gate; the lead
  reads only failure excerpts (and aesthetic renders).
- **Selective review.** Only units that touch a **risk surface** (concurrency, detach/
  process lifecycle, auth, secrets, money, data loss, public API) get an adversarial
  review pass. Trivial units ride the toolchain gate alone — reviewing every green unit
  just spends the lead's tokens to re-confirm what the gate already proved.
- **Compact, schema'd returns.** Workers return `{unit, status, files_changed[], test_cmd,
  failure_excerpt}` via a `schema`, not whole files. Diffs live on disk.
- **Reserve lead reads** for: failures, cross-unit integration seams, aesthetic renders,
  the final adversarial review, and any security-sensitive surface.
- **Token estimate** for the checkpoint: rough range = (Σ units × per-tier typing cost) +
  lead orchestration overhead (spec + integrate per unit). State it as a range, not a point.

## The loop (per unit, inside the workflow)

```
implement (engineer/senior)
  → functional gate (intern runs build + the unit's test + lint, returns pass/excerpt)
      or aesthetic gate (render → lead/vision agent looks)
    → pass + risk surface: adversarial review (code-reviewer agent) → mark ready
    → pass + trivial: mark ready (no review — the gate is enough)
    → fail: bounded retry (≤2) with the failure excerpt fed back
      → still red: escalate to the lead (returned in results; the lead fixes in main loop)
```

Use `pipeline()` so a unit that passes its gate moves to review while slower units still
implement — no barrier wasted. Use `parallel()` + a single end-gate when units are disjoint
and you only need to review the one or two risky ones. Loop-until-green per unit;
loop-until-dry for open-ended bug hunts (stop after K rounds find nothing new).

## Guardrails (inherit the user's rules — never override)

The skill is generic; the *enforcement* lives in the user's global rules and the project's
`CLAUDE.md`. **Read those first; they win over this skill.** In particular the lead must:

- **Honor the project's branch model** (e.g. work branches off a trunk; never push trunk).
  When workers write the *same* files in parallel, isolate them with `isolation:'worktree'`;
  when units touch **disjoint** files in one repo, skip worktrees (they cost setup + disk —
  don't pay for isolation you don't need).
- **Run the pre-commit/secret gate** and fix root causes — never `--no-verify`. The lead
  runs the commit; workers never commit.
- **Use the correct author identity per remote** (the user's rules define which email maps
  to which host) and **never leak internal/private infra references to a public remote.**
- **Flag sensitive-domain work** (compliance / legal / conflict-of-interest, per the user's
  rules) — surface it, let the user decide; don't silently proceed or silently skip.
- **Never commit planning artifacts** (`CLAUDE.md`, `GOALS.md`, handoffs, `.claude/`).

If you adapt this skill for a different user/org, these bullets are the seam: point them at
*that* user's rule set. The mechanism stays; the specifics are theirs.

## Workflow template (the lead authors the units inline, then runs this)

The lead does research + architecture + decomposition in the main loop, then passes the
unit list as `args`. The workflow runs implement → gate → (selective) review.

```js
export const meta = {
  name: 'fable-team-build',
  description: 'Implement scoped units, gate functional ones with the toolchain and visual ones by render, review only risk-surface units',
  phases: [{ title: 'Build' }, { title: 'Gate' }, { title: 'Review' }],
}
const UNIT = { type:'object', required:['unit','status'], properties:{
  unit:{type:'string'}, status:{enum:['done','blocked']},
  files_changed:{type:'array',items:{type:'string'}},
  test_cmd:{type:'string'}, failure_excerpt:{type:'string'}, notes:{type:'string'} } }
const VERDICT = { type:'object', required:['unit','approved'], properties:{
  unit:{type:'string'}, approved:{type:'boolean'}, issues:{type:'array',items:{type:'string'}} } }

const RULES = args.rules   // the composed preamble (repo+branch, no-git, test cmds, style, quirks)
// units: [{ id, spec, tier:'haiku'|'sonnet'|'opus', lane:'functional'|'aesthetic',
//           test_cmd, accept, risk:boolean, files:[...], isolate:boolean }]
const units = args.units

const results = await pipeline(units,
  u => agent(
    `${RULES}\n\nUNIT ${u.id}\n${u.spec}\n\nAcceptance: ${u.accept}\n` +
    `Prove it with: ${u.test_cmd}\nTouch ONLY: ${(u.files||[]).join(', ')}\n` +
    `Return ONLY the schema. Do not commit.`,
    { label:`build:${u.id}`, phase:'Build', model:u.tier, schema:UNIT,
      ...(u.isolate ? { isolation:'worktree' } : {}) }),

  (built, u) => {
    if (!built || built.status !== 'done') return built
    // Trivial functional unit: the gate is enough, no lead-grade review.
    if (u.lane === 'functional' && !u.risk) return built
    // Risk-surface or aesthetic: one review pass with the matching lens.
    const lens = u.lane === 'aesthetic'
      ? `Look at the rendered result of "${u.id}". Judge layout, hierarchy, spacing, ` +
        `motion, copy, and UX against: ${u.accept}. Default approved=false unless it is clean and coherent.`
      : `Adversarially review unit "${u.id}". Spec: ${u.spec}. Files: ${(built.files_changed||[]).join(', ')}. ` +
        `Run ${u.test_cmd}. Hunt the risk surface (races, lifecycle, auth, secrets, data loss). ` +
        `Default approved=false unless it meets the acceptance criterion cleanly.`
    return agent(lens,
      { label:`review:${u.id}`, phase:'Review',
        model: u.lane === 'aesthetic' ? 'sonnet' : 'sonnet',
        agentType: u.lane === 'aesthetic' ? 'ui-ux' : 'code-reviewer', schema:VERDICT })
      .then(v => ({ ...built, verdict:v }))
  }
)
return results.filter(Boolean)
// The lead reads this back: integrate approved units, fix/redo blocked or rejected ones
// itself, run the project gate, eyeball the aesthetic renders, then commit on the feat branch.
```

Scale the fleet to the request: a quick bug → a few units, single review; "build the X
subsystem" or "be thorough" → larger breakdown, multi-lens review, a completeness pass
("what unit is missing?") before declaring done. When unsure, lean thorough for
build/audit work, lean lean for a quick fix — and below the solo-floor, just do it.

## What the lead does NOT do here

- Does not delegate the *orchestration*. Hard reasoning can go to a senior worker, but the
  architecture, evaluation, taste, and integration stay with the lead.
- Does not maximize task count — it decomposes to the **fewest** script-provable units.
- Does not skip the toolchain gate to "save a step" — the gate is the cheap evaluator.
- Does not route aesthetics through the toolchain (it can't judge them) nor pretend they're
  free — it budgets a visual pass for them.
- Does not push to trunk, commit secrets/planning docs, or merge without the project's rules.
- Does not invent architecture it could have looked up — research first, time-boxed.
