---
name: lead-generation-and-cold-outbound
description: Guides agents through building a cost-efficient cold-outbound lead generation pipeline using cascading (waterfall) data sourcing, AI-driven contact discovery beyond LinkedIn, self-correcting recurring jobs, goal-mode task contracts for list-building, a recurring campaign-optimization loop, and open-source fallbacks instead of paid SaaS. Use when asked to build or automate a lead list, enrich contacts, stand up a cold email pipeline, or cut GTM tooling spend. Not for one-off manual prospecting under ~50 leads, where the automation overhead outweighs the benefit.
---

# Lead Generation and Cold Outbound

## Overview

Treat lead generation as a waterfall pipeline, not a single data pull: cheap, high-precision sources first, broad AI web search only to fill the gaps they leave, and open-source tooling wherever it matches paid-tool quality. This keeps cost per lead low while reply rates stay high, because coverage and contact accuracy are checked layer by layer instead of trusted from one source.

## When to Use

- Building a new outbound/cold-email lead list from scratch
- A single-source list (LinkedIn scrape, Google Maps export) has coverage gaps for niche or local companies
- A target decision-maker can't be found because they have no LinkedIn or social profile
- You want a list that refreshes itself on a schedule instead of a one-time export
- You're paying for several SaaS tools (Clay, BuiltWith, a hosted LLM, a scraping subscription) and want to cut cost per lead

**Not for:** campaigns under ~50 leads (manual sourcing beats building a pipeline), or any list where scraping/cold email would violate applicable law (CAN-SPAM, GDPR, CASL, etc.) — check that before automating, not after.

## Core Process

### Step 1: Company waterfall sourcing

1. Pick the single highest-precision source for the niche as tier 1 — a LinkedIn-based database (e.g. Clay, Prospeo) for role-based B2B targeting, or Google Maps/Places for local-business targeting.
2. Export tier 1, then mark coverage per ICP segment (industry × size × region). Segments returning zero or few results are the gaps, not the whole list.
3. For gapped segments only, run tier 2 — broad AI web-search sourcing (Exa.ai, Parallel.ai, Ocean.io or equivalent) scoped to that segment, filtered to companies with a live website.
4. Deduplicate tier 1 + tier 2 by domain before contact enrichment. Never enrich the same company twice — it's the single biggest avoidable cost in the pipeline.

### Step 2: Contact discovery beyond LinkedIn

1. Try LinkedIn/company-page lookup first — cheapest and most structured.
2. If no profile match, fall back to agentic web search: query name + role + company through a search/AI-overview API and a custom parser that pulls name + title out of unstructured mentions (press releases, conference speaker pages, podcast guest lists).
3. Verify every fallback-tier contact against a second signal (email-pattern match, staff page on the company site) before adding it to the send list. Fallback sourcing has a materially higher false-positive rate than a LinkedIn hit and must not go out on one signal alone.

### Step 3: Self-correcting recurring jobs, not brittle cron

1. Anything that can hit an edge case (rate limit, schema change, empty page, new markup) should run as an agent loop, not a fixed script: the agent runs the job, detects the anomaly, adjusts its query or approach, and retries — instead of failing silently or throwing a stack trace at 3am.
2. Minimum recurring jobs to run this way: (a) a daily top-N ICP company/role refresh from the tier-2 web-search sourcing, (b) a daily reply-rate / bounce-rate rollup posted somewhere a human reviews it.
3. Give every agentic job an explicit failure budget (e.g., stop and alert after 3 consecutive failed self-corrections). Autonomous retry is not the same as unattended forever — a job that silently keeps "adjusting" a broken query burns money without producing leads.

### Step 4: Goal-mode contract before any non-trivial list-building task

Don't dispatch a vague instruction like "get me a list of X." Establish a contract first:

1. Load context — existing list, ICP definition, prior campaign performance.
2. Ask 10–15 clarifying questions covering: source priority order, required output columns, dedup rules, verification threshold, output format.
3. Write explicit pass/fail acceptance criteria (e.g., "every row has a verified email, columns are normalized to the agreed schema, output is a CSV, zero duplicate domains").
4. Keep working until the criteria pass. Partial coverage is not done — it's a gap that needs to go back through Step 1's waterfall.

### Step 5: Recurring campaign-optimization loop

1. On a fixed schedule (weekly minimum, daily for high-volume campaigns), pull reply-rate data segmented by role, industry, and company size.
2. Identify which offer/subject-line variants correlate with the highest reply rate per segment.
3. Output a short insight report with a recommended next action per segment — not a raw table dump — and route it to the same channel as the Step 3 reply/bounce rollup so both land in one place a human actually checks.

### Step 6: Prefer open-source over paid where quality is equivalent

Before renewing or adding a paid SaaS integration, check whether a self-hosted alternative covers the same need at comparable quality:

| Paid tool class | Open-source / self-hosted alternative | Tradeoff to check before switching |
|---|---|---|
| Managed browser QA / competitor monitoring | Agent-driven browser automation (e.g. Browser Use) | Needs hosting compute; no vendor SLA |
| Hosted LLM for custom per-lead variable generation | A locally-run open-weight model, fine-tuned on your own examples | Needs GPU/hosting capacity; must validate output quality against the paid model on a sample before cutover |
| Website-copy scraper subscription | Plain HTML-to-text extraction library | No JS-rendering support unless paired with a headless browser |
| BuiltWith-style tech-detection subscription | Static tech-fingerprint script against page HTML/response headers | Lower coverage than a maintained fingerprint database; fingerprints need periodic re-checking as sites update |

Validate any substitution against a sample of the paid tool's output before fully switching. "Cheaper" only counts if precision and recall hold up — don't take a vendor's marketing claim (or a video's) about a specific model or release date at face value; verify it against the model's own release notes before building a dependency on it.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "One good source (LinkedIn/Clay) is enough" | Coverage gaps on niche or local companies go silently to zero — waterfall to tier 2 before assuming a segment is genuinely empty |
| "No LinkedIn profile means no lead" | Web-mention and AI-overview search surface decision-makers with no social footprint; skipping them shrinks the addressable list for no reason |
| "The script failed on an edge case, I'll just rerun it manually" | Wrap it in an agent loop with a failure budget instead — manual reruns don't scale and hide the underlying pattern that will recur |
| "We'll figure out the acceptance criteria as we go" | Define pass/fail criteria before starting list work — "looks about right" is not a stopping condition |
| "Weekly reply-rate analysis can wait until it's a problem" | The optimization loop should run before conversion drops, not after — it's a standing job, not a fire drill |
| "Open-source is free so it's strictly better" | Validate output quality against the paid baseline first; hosting and maintenance cost, plus lower coverage, can offset the savings |
| "It's just cold email, compliance doesn't apply to us" | CAN-SPAM/GDPR/CASL apply based on recipient location and data source, not sender intent — check before scaling volume |

## Red Flags

- A single-source export is treated as the final list with no coverage check per ICP segment
- A contact is added to the send list from one unverified fallback-tier hit
- A "recurring job" silently drops rows or exits on the first unexpected response instead of adapting
- List-building work starts with no written pass/fail criteria
- Campaign performance data sits in a spreadsheet nobody reviews between sends
- A paid tool is kept "because it's already set up" without ever testing whether an open-source alternative is good enough
- A specific model name, version, or release date is repeated from a secondhand source (video, blog post) without checking the vendor's own release notes

## Verification

- [ ] Company list has recorded coverage per ICP segment (tier-1 vs tier-2 sourced)
- [ ] No duplicate domains in the final company list
- [ ] Every fallback-sourced (non-LinkedIn) contact has at least two corroborating signals
- [ ] Recurring jobs (sourcing refresh, reply/bounce rollup, optimization loop) run on a schedule and post output somewhere a human reviews it
- [ ] Any open-source substitution has been validated against a paid-tool sample before full cutover
- [ ] Outbound volume and sourcing methods have been checked against applicable email/data-privacy law for the target region
