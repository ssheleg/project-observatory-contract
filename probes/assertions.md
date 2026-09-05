# Admission probe assertions

`estate.survey` is a read capability with `effect: none`. Every probe below is
bounded, mutates nothing, and can be run against a cold machine.

## P1 — `survey-single-project` (the admission fixture)

Input: `fixtures/admission-input.json` — one project by id.

- The result validates against `capability-output.schema.json`.
- `counts.projects == 1`, and `projects[0].id` equals the requested id.
- `projects[0].membershipRules` has exactly one entry per entry in
  `projects[0].repositories`. A repository with no stated rule is a failure, not
  a warning: the rule is the evidence that the link is not a guess.
- `evidence` is non-empty and every entry resolves to a source id present in the
  registry.
- `degraded` is present. An omitted `degraded` fails — silence about coverage is
  the failure mode this field exists to prevent.
- No write occurs: the registry's git status and the ledger's max revision are
  unchanged after the call.

## P2 — `survey-rejects-name-inference`

Input: a project scope whose vault note names two repositories of an owner that
holds twelve.

- Exactly the two named repositories appear.
- The other ten do not. This is fixture **T1** from `docs/knowledge-pack.md`,
  and it is a probe rather than only a unit test because it is the defect most
  likely to reappear as a well-meaning improvement.

## P3 — `survey-declares-degradation`

Input: `fixtures/degradation-input.json` — the whole estate.

- With the event store unreachable the call still returns a typed result.
- `degraded` names the store and its reason.
- `degraded` names bitbucket: 16 repositories are known only from a local remote
  because no API listing is configured (OQ-0002).
- Those repositories appear with `discoveredBy: local-remote-only`.
- The call does not fail. A survey that returns nothing because one source is
  down is less useful than one that returns what it has and says what it lacks.

> **This probe was rewritten on 2026-09-03.** It first asserted that removing the
> GitHub token would make `degraded` name GitHub. That token gates the
> *collector*, not this capability — `estate.survey` reads the registry — so the
> assertion could never have run. A probe that cannot be executed as written is a
> false declaration, and declaring it is worse than having one probe fewer.

## Cross-cutting

- Timeout and cancellation return a typed partial result marked in `degraded`,
  never a transport error dressed as an empty survey.
- No probe publishes, messages, charges, deploys, mutates production, or reads a
  credential. The registry carries no `credential` data class by construction.
- Probe failure is `probe-failed`. A missing negotiated MCP capability or a
  missing declared tool is `protocol-incompatible`, which is a different verdict
  and must not be reported as the first.
