# Token Budget and Model Selection

Reference guide for estimating token cost and choosing the right model before starting any skill or workflow. Read this before spawning subagents, enabling dynamic workflows, or invoking `/ship`.

---

## Model Selection

### The universal rule

Choose the model based on the **nature of the judgment required**, not the size of the task. Every provider has three tiers; the mapping below applies across all of them.

| Tier | Task profile | Examples across providers |
|------|-------------|--------------------------|
| **Fast / cheap** | Pattern matching, format validation, boilerplate, read-only search, coverage scans | Claude Haiku · GPT-4o mini · Gemini Flash · Codex (OpenAI) |
| **Balanced** | Most implementation: code review, writing tests, ETL, refactoring, report generation | Claude Sonnet · GPT-4o · Gemini 2.5 Pro · Grok 3 |
| **Deep reasoning** | Security audits, architecture decisions, competing-hypothesis debugging, tasks where wrong = expensive | Claude Opus · o3 / o4-mini (reasoning) · Gemini 2.5 Pro (thinking mode) |

### Provider-specific model map (current as of mid-2026)

| Provider | Fast | Balanced | Deep reasoning |
|----------|------|----------|---------------|
| **Anthropic** | Haiku 4.5 | Sonnet 4.6 | Opus 4.8 |
| **OpenAI** | GPT-4o mini · Codex | GPT-4o · GPT-4.1 | o3 · o4-mini |
| **Google** | Gemini 2.0 Flash | Gemini 2.5 Pro | Gemini 2.5 Pro (thinking) |
| **xAI** | Grok 3 Mini | Grok 3 | Grok 3 (thinking) |
| **Meta (local)** | Llama 3.1 8B | Llama 3.3 70B | Llama 3.1 405B |

> Codex note: strong at code review within Claude Code via `--dangerously-skip-permissions`; weaker at open-ended reasoning. Use for focused code-diff review, not architecture decisions.

### Per-task guidance (provider-agnostic)

| Task | Tier | Why |
|------|------|-----|
| Read-only codebase search | Fast | Pattern matching, no judgment |
| Data validation / format checks | Fast | Binary pass/fail |
| Boilerplate / template generation | Fast | Low-stakes, reversible |
| Code review (routine PR) | Balanced | Good judgment, not extreme depth |
| Writing tests | Balanced | Logic needed, not deep reasoning |
| ETL / report generation | Balanced | Implementation work |
| Security audit | Deep | Wrong call is expensive |
| Architecture decisions | Deep | High leverage, hard to reverse |
| Production incident debugging | Deep | Wrong root cause is costly |
| Competing-hypothesis analysis | Deep | Requires holding multiple models simultaneously |

### Per-persona defaults (Claude Code)

| Persona | Default model | Reasoning |
|---------|--------------|-----------|
| `code-reviewer` | Sonnet | Needs good judgment, not extreme depth |
| `security-auditor` | Opus | Wrong security calls are expensive |
| `test-engineer` (coverage scan) | Haiku | Pattern matching, not reasoning |
| `test-engineer` (writing tests) | Sonnet | Test logic needs real understanding |
| Research / `Explore` subagent | Haiku | Read-only, returns a digest |
| Parallel fan-out cells (analytics) | Haiku or Sonnet | Haiku for validation cells; Sonnet for modeling cells |
| Agent Teams investigators | Sonnet | Needs reasoning but not Opus depth for most issues |
| Agent Teams (production incident) | Opus | Stakes are high, wrong root cause is costly |

To override the model for a Claude Code persona, set `model: claude-haiku-4-5-20251001` (or `-sonnet-4-6`, `-opus-4-8`) in the persona's YAML frontmatter.

---

## Token Budget by Pattern

### Pattern 1 — Direct invocation
**Baseline cost.** One context window, one persona, one artifact.

| Task size | Rough token range | Guidance |
|-----------|------------------|----------|
| Small file review | 5k–20k | Always fine |
| Medium PR (10–20 files) | 20k–80k | Normal |
| Large codebase search + report | 80k–200k | Consider `Explore` subagent to isolate the read phase |

### Pattern 3 — Parallel fan-out with merge (`/ship`)
**Cost: N × (single-persona cost) + merge turn.**

Three personas on a mid-size PR ≈ 3 × 40k + 10k merge = ~130k tokens total. Usually worth it for pre-ship verification because quality of finding per token is higher (each persona stays focused).

If you're running `/ship` on a tiny 2-file change, direct invocation is cheaper and fast enough.

### Pattern 5 — Research isolation (`Explore` subagent)
**Cost: isolated sub-agent (Haiku) + digest in main context.**

The sub-agent reads 50 files; the main agent receives a 2k digest. Main context stays clean. Almost always cheaper than reading 50 files inline.

### Dynamic Workflows (parallel agent cells)

| Decomposition size | Typical cost | Guidance |
|-------------------|-------------|----------|
| 5–10 cells (pilot) | 50k–200k | Start here to validate the approach |
| 20–50 cells | 200k–800k | Use for recurring, high-value work |
| 100+ cells | 800k–5M+ | Justify against manual equivalent (hours of analyst time) |

**Always run a 5-cell pilot on the hardest cells first.** If those fail or cycle excessively, the acceptance criterion or decomposition is wrong — fix it before scaling.

**Dynamic workflow token multipliers:**
- Each reviewer agent adds ~0.5× per cell
- Each revision cycle adds ~1× per failing cell
- Aggregation agent adds a flat ~20k–50k at the end

### Agent Teams (experimental)

Three Sonnet teammates running 10–15 minutes of adversarial debugging ≈ 300k–600k tokens. Justified only when the wrong root cause is more expensive than that cost. For routine PR review, use `/ship` instead.

---

## Cost vs. Quality Decision Tree

```
Is the acceptance criterion binary (pass/fail, reconciles, tests pass)?
├── Yes → Can it be parallelized across independent cells?
│         ├── Yes, 10+ cells → Dynamic workflow. Run 5-cell pilot first.
│         └── No, or < 10 cells → Direct invocation or slash command.
└── No (requires human judgment) → Do NOT use a workflow.
                                    Use a persona with Opus if stakes are high.
```

---

## Practical Guardrails

**Set a ceiling before you start.** Claude Code's token usage is visible in the session. Decide in advance: "If this hits 500k tokens and isn't done, I stop and diagnose."

**Pilot before scaling.** For any decomposition, run the 3–5 hardest cells first. If they cycle more than twice, the acceptance criterion or the decomposition axis is wrong.

**Prefer Haiku for read-only cells.** Validation, format-checking, and coverage-scan agents don't need Sonnet reasoning. Switching those cells to Haiku cuts cost by ~5×.

**Inline vs. isolated context.** If a subagent's input is > 30k tokens of files it needs to read, use `Explore` isolation so the main context doesn't fill up. If input is < 10k, inline is fine.

**Token cost vs. human time.** For analytics workflows: 1M tokens ≈ $3–15 depending on model. Compare to 4 hours of analyst time at your org's rate. The break-even is usually at 20–30 cells.

---

## Related

- `references/orchestration-patterns.md` — which pattern to use (cost is one input)
- `skills/dynamic-workflows-for-analytics/SKILL.md` — analytics-specific decomposition patterns
- `agents/` — persona definitions with model frontmatter you can override
