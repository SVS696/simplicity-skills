# Simplicity Skills for Claude Code

> Two [Claude Code](https://docs.claude.com/en/docs/claude-code) skills that catch **over-engineering before you ship** — an automatic "simplicity pass" over whatever you're about to hand off, with a forced per-element decision.

LLMs (and people) over-build: extra entities, statuses nobody asked for, premature patterns (CQRS, Event Sourcing, microservices), defensive code for inputs that can't happen, abstractions before the second use case, "future-proof" flexibility. In **specs** this slips past a quick read and gets torn apart in backlog grooming. In **code** it becomes complexity you have to maintain forever.

These skills add the missing layer: a reviewer that flags complexity in the *substance of the solution* — not the formatting, not the prose — and rewrites it simpler **before a human sees the draft**.

---

## What's inside

| Skill | Scope |
|---|---|
| [`simplicity-spec`](skills/simplicity-spec/SKILL.md) | Specs / requirements, architecture decisions & ADRs, API design, DB schema changes, workflow / state machines, RBAC/ABAC, microservice boundaries, pattern choice (CQRS / Event Sourcing / event-driven / saga / rule engine), **and infrastructure / DevOps decisions** (Kubernetes vs docker-compose, cache / queue / replica / sharding, observability stack, CI/CD, extra services / environments). |
| [`simplicity-code`](skills/simplicity-code/SKILL.md) | Writing, reviewing and refactoring code. |

> **Language note.** The `SKILL.md` files are written in **Russian** (the author's working language). The method is language-agnostic — the smell catalogs below are translated, and changing the output language is a one-line edit. PRs with an English variant are welcome.

---

## The core rule

> Start with the **smallest** solution that satisfies the **explicit** requirements. Every extra entity, status, async flow, abstraction, permission dimension, dependency or config knob must map to a **current** requirement (a user story, acceptance criterion, business rule, regulatory constraint, or an existing architectural boundary). If it doesn't map — `REMOVE` or `DEFER`.

**"For the future / for flexibility / for scalability / just in case" is not a justification — it is itself a smell.**

---

## How it works

- **Dual mechanism** — a *generation guardrail* (stops the drift while writing) **and** a *draft linter* (catches what slipped through anyway).
- **Auto + always-show** — runs automatically before any non-trivial spec/code, and **always** prints the result: a `Simplicity pass` table when it finds something, or a single line when it's clean. The point is that over-engineering can't slip past a quick read.
- **5-step pass** — (1) fix the minimal core → (2) start from the simplest variant → (3) run the smell list → (4) check every complication against a justification trigger → (5) **rewrite the solution already simplified** (diagnosis without a rewrite just leaves you the bloated draft).
- **Forced decision per element** — `KEEP` · `SIMPLIFY` · `REMOVE` · `DEFER` · `ASK`.

### Output format

When something is found, you get a table you can scan in seconds:

```markdown
## Simplicity pass

| Element | Smell | Why redundant / risk | Backing requirement | Decision | Use instead |
|---|---|---|---|---|---|
| `workflow_states` table | Statuses without transitions | No story where the user drives states | none found | REMOVE | One status field on the main entity |
| `OrderCalculated` event | Event for a local op | No external consumer; result needed now | calc on save | SIMPLIFY | Synchronous domain call |
| `metadata` column | Catch-all model | Nobody specified who writes/reads keys | none found | DEFER | Explicit fields when a case appears |

### Complexity delta
Removed: −2 tables, −3 statuses, −1 async event.
Kept (justified): audit log (traceability); department access dimension (current RBAC model).
Before → after: a separate approval workflow engine → `approval_status` field + 3 explicit transitions.
```

When clean: `Simplicity pass: clean — every element maps to a requirement.`

---

## Smell catalog — specs & architecture

| Smell | Checkable signal |
|---|---|
| Entity without an action | A DB/API entity with no user story / business operation behind it. |
| Status without transitions | Statuses introduced, but no who/when/what-it-blocks. |
| "Just-in-case" statuses | `draft / archived / cancelled / pending_review` with no described scenario. |
| Catch-all model | `type / category / metadata / attributes / rules / config` where one concrete behavior is needed. |
| Config without an admin | Configs/rules/matrices described, but no role/screen/process that manages them. |
| RBAC/ABAC beyond the case | Dimensions/policies/inheritance/scope where 1–2 roles or an owner check suffice. |
| CQRS without asymmetry | Commands split from reads with no separate read model / load / independent scaling. |
| Event Sourcing instead of an audit log | Storing the full event history where a change log or current status is enough. |
| Event/queue for a local op | A queue/event where the action is synchronous, in one bounded context, result needed now. |
| Saga for 2–3 steps | Orchestration/choreography with no real distributed transaction or critical partial failures. |
| Unrequested channels | webhook/import/export/notifications beyond the single required channel. |
| Abstraction before the 2nd case | Interface/strategy/rule engine/templating before a second independent variant exists. |
| "Future" DB columns | A column that no logic/API/UI/acceptance criterion uses. |
| Lookup table instead of an enum | A lookup table + CRUD + permissions where the set is small, stable, business-unmanaged. |
| Versioning without a need | versioning/snapshots/effective-dating with no as-of recompute / comparison / legal-record scenario. |
| Resilience without a network | Idempotency/retries for operations that never cross a network/queue/external system. |
| Redundant ownership dimensions | tenant/org fields duplicating a boundary the parent entity already sets. |
| Exceptions richer than the basics | An edge-cases section longer than the main logic, full of unrequested scenarios. |
| API mirrors the internals | The external contract leaks `commands/events/handlers/workflowId/policyId` instead of a plain resource/action. |
| Local task → process change | Business process/roles/lifecycle change for what was a local data/calc/access change. |

## Smell catalog — infrastructure / DevOps

| Smell | Checkable signal |
|---|---|
| Orchestrator without scale | Kubernetes / Helm / service-mesh where docker-compose / a couple of containers / systemd suffice. |
| Service instead of a module | A new deploy unit / repo / container where a module in an existing service is enough. |
| Cache without measured load | Redis / a cache layer with no proven read problem (profile, metric). |
| Broker for a single task | Kafka / RabbitMQ / a dedicated stream where a sync call or an existing bus suffices. |
| HA/replicas/sharding without requirements | Read replica, sharding, multi-AZ, failover with no load/availability (SLA) requirement. |
| Multi-region / multi-cloud too early | Geo-distribution with no data-residency or latency requirement. |
| Heavy observability on day one | Full Prometheus + Grafana + Loki + Tempo + alerting where logs + `/metrics` are enough. |
| Over-built CI/CD | Multi-stage pipelines / build matrices / extra environments beyond real needs. |
| IaC / autoscaling before the need | Terraform/Ansible wrapper modules around a single resource; HPA/autoscaling/limit tuning with no load profile. |
| Self-hosting over what exists | A self-hosted component where a managed alternative or an existing project component already covers it. |
| Extra environments / DBs | A new stand, environment, or DB instance with no use scenario; separate storage for a set that fits an existing schema. |

## Smell catalog — code

| Smell | Checkable signal |
|---|---|
| 200 lines instead of 50 | The implementation is noticeably longer than the task needs; an obviously shorter form exists. |
| Abstraction without a 2nd case | `interface`/Strategy/Factory/Protocol/generic with a single implementation. |
| Premature generality | "Future" params/hooks/options never called from current code. |
| Guarding the impossible | `try/except`/checks/branches for inputs that can't occur at current call sites. |
| LLM for mechanics | A model/subprocess where `sed`/`grep`/`awk`/`jq` or a direct read+edit suffice. |
| Command/flag instead of a recipe | A new CLI flag/command where one command or a short instruction does it. |
| Framework for a call | LangChain/orchestrator/DI container where a direct API/function call suffices. |
| Wrapper over stdlib | A custom helper over what the library already does in one line. |
| Layers without a need | service→repository→mapper→DTO for single-table CRUD. |
| Premature optimization | Cache/pool/batching/index with no measured problem (profile/bench). |
| Config for one value | env/flag/setting where a constant is enough. |
| Refactoring the neighbors | Touching working code/comments/formatting outside the task. |
| Silent assumption | One of several ambiguous readings picked without surfacing it. |
| Reinventing the wheel | Writing what already exists in the project or the standard library. |
| Class hierarchy / inheritance | A class tree where a function or composition suffices. |
| Type acrobatics | Heavy generics/types for a single concrete type. |
| async without a reason | Queue/thread/async with no real concurrent IO or process boundary. |
| Parsing non-existent cases | Handling input formats/variants the task never has. |

## When complexity *is* justified

Each skill ships a "triggers" table — complexity is allowed only when its trigger is real and current, e.g.:

- **New abstraction** → ≥ 2 real implementations *now* (the rule of three is even safer).
- **Cache / optimization** → a *measured* performance problem, not a hypothesis.
- **Event / queue / broker** → a real async consumer, peak buffering, or a service boundary.
- **Orchestrator (k8s)** → genuinely many services to scale/self-heal, not 1–3 containers.
- **CQRS / Event Sourcing** → distinct read/write models, or a legally meaningful history / as-of recompute.
- **Replica / HA / sharding** → an availability (SLA) or volume/load requirement backed by numbers.

---

## Where it sits

These skills are deliberately narrow. They are **not**:

- a document-hygiene checker (no JSON/validation/logging boilerplate, Occam's razor on duplicated facts);
- a prose-style pass (de-bureaucratese, removing AI tells).

They cover the gap those leave: **the complexity of the solution itself.** Run them alongside whatever you already use for formatting and prose.

---

## Install

Clone and drop the skills into your Claude Code skills directory:

```bash
git clone https://github.com/SVS696/simplicity-skills.git
cp -r simplicity-skills/skills/simplicity-spec ~/.claude/skills/
cp -r simplicity-skills/skills/simplicity-code ~/.claude/skills/
```

They auto-trigger from their `description` frontmatter (drafting/reviewing specs, architecture, infra, or code). You can also invoke them explicitly — e.g. "проверь на простоту" / "run a simplicity pass".

To make them fire reliably inside a specific project, add a one-line pointer from that project's spec-writing rules or `CLAUDE.md` / agent definition.

---

## Design notes & credits

- **Dual guardrail + linter, forced-decision table** — the key idea is that diagnosis alone is useless; the skill must rewrite the artifact and force a `KEEP/SIMPLIFY/REMOVE/DEFER/ASK` call per element, with a mandatory mapping to a current requirement.
- Inspired by Andrej-Karpathy-style coding guidelines (simplicity first, surgical changes, think-before-coding), the *requirements smells* literature (NALABS; ISO/IEC/IEEE 29148), and the perennial YAGNI / KISS / "the simplest thing that could possibly work."
- Designed and stress-tested with a second-opinion pass from another model.

## License

[MIT](LICENSE).
