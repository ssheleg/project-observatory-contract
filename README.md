# project-observatory — Fabric contract surface

The public half of [Project Observatory](https://github.com/ssheleg/project-observatory):
the JSON Schemas and admission fixtures a Fabric host must compile and run
**before** any credential or project data is exchanged.

The implementation stays private, and so does its provider manifest — the
manifest is an installation artefact, required by the contract to carry a
`connection.executableRef` pointing at one machine's executable. It reaches a
host by direct configuration, which the contract's discovery step allows.

| Path | What a host does with it |
|---|---|
| `schemas/capability-*.schema.json` | validates `estate.survey` input and output |
| `schemas/record-*.schema.json` | validates `project.record` input and output |
| `fixtures/*.json` | bounded, non-publishing admission probes |
| `probes/assertions.md` | what each probe asserts, in prose |

## Tags are the provider revision, and they never move

Every URI a manifest references is pinned to `rev-N`, where `N` is the
`provider.revision` that manifest declares. A tag is never repointed: a changed
schema is a new revision and a new tag. The publisher verifies this by fetching
every URI anonymously and comparing the bytes with its source, so a stale
publication fails the source repository's own gate rather than a host's
admission.

Contract: [fabric-agent-contract](https://github.com/passioncode-ai/fabric-agent-contract) `0.1.0`.
