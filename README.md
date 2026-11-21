# Simplicity Skills for Claude Code

Use these two skills to review a solution before you hand it over. Each skill identifies parts that have no current use, makes a decision about each part, and shows a simpler solution.

| Skill | When to use it |
|---|---|
| [simplicity-spec](skills/simplicity-spec/SKILL.md) | Write or review requirements, an architecture decision, an API, a database schema, a workflow, or an access model. Use it to assess patterns, service boundaries, infrastructure, deployment, monitoring, and CI/CD. |
| [simplicity-code](skills/simplicity-code/SKILL.md) | Write, review, or refactor code. |

The skill instructions are in Russian. The tables below explain the checks in English. You can request the result in another language. Contributions with an English version are welcome.

## How to review a solution

Use the checks while you write and again before you deliver a nontrivial result. Start with the user action, its consumers, the result, and the project constraints. For each entity, state, asynchronous operation, access rule, dependency, or setting, find a current reason to keep it. A reason can be a user story, an acceptance criterion, a business rule, a regulatory requirement, or an existing architecture boundary.

1. Check whether the current behavior already meets the requirement. Then check the existing product or code. For code, check the standard library. For a design, check the component that owns the rule.
2. Check built-in platform features and mechanisms the project already uses. Try a direct change in the existing component before you introduce a new mechanism.
3. Use the tables below to examine each addition. A plan for future growth needs a specific current requirement before it can justify extra complexity.
4. Choose a decision for each finding. Revise the solution and provide the applicable check. Show the result even when the review finds nothing to change.

| Decision | Action |
|---|---|
| `KEEP` | Keep the element. Identify the current need. |
| `SIMPLIFY` | Replace the element with a simpler option that meets the need. |
| `REMOVE` | Remove an element that has no required use. |
| `DEFER` | Record a valid future need outside the current change. For a design, record the current limit, a measurable condition for review, and the next mechanism. For code, include those details when the limit is known. |
| `ASK` | Ask the author when the change would affect required behavior, or when the requirements conflict or allow different interpretations. |

Preserve required behavior, validation at trust boundaries, protection against data loss, security, accessibility, and compliance. Keep the ability to test or monitor required behavior. Preserve a place to extend the solution when there are real variants or a later change would be expensive.

Review the complexity of the solution. Use separate checks for document structure and style, including repeated facts and unnecessary boilerplate.

### Example review

```markdown
## Simplicity pass

| Element | Signal | Risk or cost | Requirement | Decision | Replacement |
|---|---|---|---|---|---|
| Table `workflow_states` | No configurable transitions | Users have no task that requires a state catalog | None found for the table | REMOVE | A status field on the main entity |
| Event `OrderCalculated` | Local operation | No external consumer uses the event. The caller needs the result immediately. | Calculate on save | SIMPLIFY | Call the calculation function directly |
| Column `metadata` | Unspecified keys | No consumer uses the keys today | One channel is in use | DEFER | Define the fields when a second channel is confirmed |

### Change summary

Removed two tables, three statuses, and one asynchronous event.
Kept the change log for audit and the department access check required by the role model.
A single field, `approval_status`, and three explicit transitions cover the approval workflow.
The current design covers one channel. When a second channel is confirmed, identify its required fields.
```

If every design element has a reason to stay, the English result is `Simplicity pass: clean — every element maps to a requirement.` Each skill gives the exact Russian wording for its result.

## Requirements and architecture

| Element | Check |
|---|---|
| Entity with no action | Which user story or business operation uses the database or API entity? |
| Status with no transition | Who changes the status, when do they change it, and what does it block? |
| Extra statuses | Which scenario requires `draft`, `archived`, `cancelled`, or `pending_review`? |
| General-purpose model | Would an explicit field cover the one case served by `type`, `category`, `metadata`, `attributes`, `rules`, or `config`? |
| Settings with no owner | Who manages the rules, settings, or matrices? Which screen or process do they use? |
| Access controls | Would one or two roles or an owner check suffice? Which rule requires more attributes, policies, scopes, or inheritance? |
| CQRS | Do reads and writes require different models, loads, or independent scaling? |
| Event Sourcing | Does the business need to reconstruct state, or would a change log or the current status suffice? |
| Event for a local operation | Does a synchronous operation in one domain need a queue if the caller needs the result immediately? |
| Saga with two or three steps | Does a distributed transaction have critical failure cases after some steps complete? |
| Extra channels | Which requirements call for webhooks, import, export, or notifications beyond the required channel? |
| Abstraction for one case | Is there a second independent use for the interface, strategy, rule engine, or template? |
| Unused database field | Which code, API, screen, or acceptance criterion uses the field? |
| Lookup table | Would an enumeration cover a small, stable set of values that business users do not manage? |
| Version history | Does a calculation, comparison, or legal record depend on the version at a given date? |
| Retry and duplicate protection | Can the operation cross a network, queue, or external system? Can a retry or partial failure affect the result? |
| Duplicate ownership fields | Does a tenant or organization field repeat a boundary that the parent entity sets? |
| Extensive exception handling | If the exceptions take more space than the main case, which exceptions occur in the requested scenarios? |
| Internal details in the API | Can a resource or action replace internal names such as `commands`, `events`, `handlers`, `workflowId`, or `policyId`? |
| Wider process change | Does a local change to data, a calculation, or access require new roles, a new process, or another lifecycle? |

## Infrastructure and deployment

| Element | Check |
|---|---|
| Orchestrator | Would Docker Compose, a few containers, or systemd suffice in place of Kubernetes, Helm, or a service mesh? |
| New service | Can a module in the current service avoid a new deployment, repository, or container? |
| Cache | Which measurement shows that reads need a cache such as Redis? |
| Message broker | Can a direct call or the existing bus handle the task without Kafka, RabbitMQ, or a separate stream? |
| Replicas, failover, or sharding | Which availability target or measured load requires read replicas, sharding, several zones, or failover? |
| Multiple regions or clouds | Which data residency or latency requirement calls for this distribution? |
| Monitoring tools | Would logs and `/metrics` suffice? Which need justifies Prometheus, Grafana, Loki, Tempo, and alerts? |
| CI/CD | Which check or delivery step needs each pipeline stage, build matrix, or environment? |
| Infrastructure as code and scaling | Does a single resource need a Terraform or Ansible wrapper? Which load profile requires autoscaling or a change to limits? |
| Self-hosted component | Is a managed service or an existing project component sufficient? |
| Extra environments or databases | Who uses the new environment or database instance? Could the data use the existing schema? |

## Code

| Element | Check |
|---|---|
| Long implementation | Can 50 lines replace 200 while preserving required behavior? |
| Abstraction for one case | How many implementations use the interface, strategy, factory, protocol, or generic type? |
| Unused options | Which calls use each parameter, hook, and option? |
| Checks for impossible input | Can any current caller supply the input handled by this branch or exception handler? |
| Language model for text replacement | Can `sed`, `grep`, `awk`, `jq`, or a direct edit do the work? |
| New command or flag | Can instructions in the skill or a single existing command do the work? |
| Framework for one call | Can the code call the API or function directly without LangChain, an orchestrator, or a dependency injection container? |
| Standard library wrapper | Does the library already perform the operation in one call? |
| Extra layers | Does single-table CRUD need a service, repository, mapper, and data transfer object? |
| Optimization | Which benchmark or profile justifies the cache, pool, batch, or index? |
| Setting for a fixed value | Could a constant replace the environment variable, flag, or setting? |
| Unrelated refactoring | Does the task require changes to nearby code, comments, or formatting? |
| Unstated assumption | Which interpretation of the request did the implementation use? |
| Duplicate implementation | Does the project or standard library already provide the feature? |
| Class hierarchy | Would a function or composition suffice? |
| Complex types | How many concrete types use the generic implementation? |
| Asynchronous code | Does a queue, thread, or asynchronous call serve concurrent I/O or a process boundary? |
| Extra input formats | Which input formats does the task require? |

## Reasons to keep a mechanism

The Russian skill files list the reasons for each mechanism. The following cases illustrate how to use them.

- Introduce an abstraction when at least two real implementations need it. Where possible, wait for a third.
- Use a cache or optimize code when a measurement shows a performance problem. A repeated, expensive calculation can also justify caching.
- Use an event, queue, or broker for an asynchronous consumer, peak buffering, or exchange across service boundaries. Check whether the scenario can tolerate delayed updates.
- Use an orchestrator when many services need scaling or automatic recovery. For one to three containers, first check a simpler option.
- Consider CQRS for different read and write models, high read load, or complex projections. Consider Event Sourcing when the business needs legally significant history, state reconstruction, or recalculation of past results.
- Use replicas, high availability, or sharding when a numerical availability, volume, or load requirement justifies them.

## Measure the effect

Run the same real task with and without the skill in separate fresh contexts. Use the same model and repository revision. Compare the resulting artifacts or diffs. Record tool calls, model calls, elapsed time, and cost where available.

Include a task that invites extra complexity, a task that already has a minimal solution, and a task with safety requirements. Both results must pass the same correctness, safety, and acceptance checks. If there is nothing to remove, report zero improvement. Report savings for a repository only when you have measured a baseline.

## Install

### Marketplace

Install both skills as the `simplicity` plugin.

```text
/plugin marketplace add SVS696/svs-skills
/plugin install simplicity@svs
```

Claude Code selects the skills from their descriptions. To select a skill yourself, use `/simplicity:simplicity-spec` or `/simplicity:simplicity-code`. To update both skills, use `/plugin update simplicity@svs`.

### Manual installation

```bash
git clone https://github.com/SVS696/simplicity-skills.git
cp -r simplicity-skills/skills/simplicity-spec ~/.claude/skills/
cp -r simplicity-skills/skills/simplicity-code ~/.claude/skills/
```

Ask Claude Code to “run a simplicity pass” or «проверь на простоту». To make the check explicit in a project, add a reference to its specification rules, agent definition, or `CLAUDE.md`.

## Design and credits

The skills apply the same checks during drafting and before handoff. Each finding needs a decision and a link to the current need. The reviewer provides the revised solution.

The approach draws on Andrej Karpathy’s guidelines for focused code changes, the requirements literature (NALABS and ISO/IEC/IEEE 29148), and the YAGNI and KISS principles. Another model provided a second opinion during design and testing. The skills run in [Claude Code](https://docs.claude.com/en/docs/claude-code).

## License

[MIT License](LICENSE).
