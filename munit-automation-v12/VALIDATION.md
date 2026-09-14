# Version 12 validation

**52 platform-neutral authoring/synthetic checks passed.** Only Python's standard library was used to read package text, parse canonical XML examples, and exercise independent synthetic reference checks. No macOS-specific command/runtime, PowerShell, installer, VS Code or Mule/Maven command was executed for this revision, as requested.

## Changes and their verification

The router now performs only read-only source detection and handoff. HTTP selects the MAIN API artifact from pom.xml/APIKit and the effective local repository. The known common-library filename is used only when following applied trait dependencies after extraction; it cannot replace main-artifact selection.

The workers were rewritten around one workflow, one source-derived ledger, one invariant table, canonical templates and one acceptance checklist. Output requirements for recursive paths, raw connector mocks, fixtures/constraints, request headers, APIKit/main error suites, test-config import, exactly two properties and targeted refresh remain visible. Source ownership/scenario/mock/header completeness is now checked by the instructed acceptance procedure rather than v11's large native plan framework. The large native discovery/path guard were also removed; path protection is an explicit pre/post publication check. This is not a claim of equivalent automated enforcement.

The main extraction commands were retained byte-identically and compared as TEXT only. Executable fixture constraints remain embedded; this revision did not execute them. XML example checks do not substitute for the installed Mule/MUnit XSD or actual runtime.

## File sizes

| Prompt | v11 bytes | v12 bytes | Reduction |
| --- | ---: | ---: | ---: |
| munit-generate | 4,013 | 1,187 | 70.4% |
| munit-http-listener | 166,221 | 58,886 | 64.6% |
| munit-non-http-listener | 129,335 | 42,380 | 67.2% |

Size reduction measures file bytes, not model token consumption, correctness or latency. The native appendices account for much of the remaining worker size; they keep the files independently usable without new helper dependencies.

## Executed authoring checks

- Package metadata, compact binary router and self-contained workers
- Extraction blocks retained byte-identically as text; not executed
- Required source/contract/recursion/refresh/continuation rules remain visible
- Canonical A example XML structure and instantiated source/mock/header contract
- Canonical B example XML structure and instantiated source/mock/header contract
- Canonical C example XML structure and instantiated source/mock/header contract
- Canonical non-http example XML structure and instantiated source/mock/header contract
- Synthetic rejection: missing shared import
- Synthetic rejection: bare backend payload
- Synthetic rejection: stale mock source selector
- Synthetic rejection: wrong backend resource
- Synthetic rejection: wrong request display name
- Synthetic rejection: required-header map removed
- Synthetic rejection: duplicate test name
- Synthetic rejection: extra logger
- Synthetic rejection: variables inserted into set-event
- Synthetic rejection: error response validator removed
- Synthetic rejection: Mode C public-flow source missing
- Synthetic rejection: mock moved outside behavior
- Synthetic fixture constraints: required/type/enum/predicate, arrays and escaped pointers
- Synthetic rejection: required value omitted
- Synthetic rejection: invalid enum
- Synthetic rejection: wrong branch discriminator
- Synthetic rejection: boolean used as number
- Synthetic rejection: number used as boolean
- Synthetic rejection: wrong nested enum
- Synthetic rejection: boolean enum is not numeric one
- Synthetic publication allowlist accepts only approved MUnit artifact roles
- Synthetic rejection: write outside approved scope: pom.xml
- Synthetic rejection: write outside approved scope: .github/prompts/munit-generate.prompt.md
- Synthetic rejection: write outside approved scope: target/plan.json
- Synthetic rejection: write outside approved scope: src/main/mule/api.xml
- Synthetic rejection: write outside approved scope: src/test/resources/properties/app-constants.yaml
- Synthetic rejection: write outside approved scope: src/test/resources/properties/app-secrets-prod.yaml
- Synthetic rejection: write outside approved scope: src/test/resources/in/../../main.json
- Synthetic rejection: write outside approved scope: /src/test/munit/a-test-suite.xml
- Synthetic rejection: write outside approved scope: src\test\munit\a-test-suite.xml
- Synthetic rejection: duplicate/case-colliding output
- Synthetic source-ledger comparison: four endpoint scenarios plus APIKit/main-handler owners
- Synthetic rejection: one test substituted for all endpoint branches
- Synthetic rejection: otherwise branch omitted
- Synthetic rejection: prefix external-call mock omitted
- Synthetic rejection: nested sub-flow external-call mock omitted
- Synthetic rejection: suffix external-call mock omitted
- Synthetic rejection: APIKit error suite omitted
- Synthetic rejection: main handler suite omitted
- Synthetic rejection: handler matcher substituted for concrete error trigger
- Synthetic rejection: known required applied-trait header omitted from plan
- Synthetic rejection: distinct branch scenario reused wrong input
- Synthetic reconciliation preserves unrelated legacy test without using it as selected coverage
- Synthetic rejection: unrelated legacy test deleted
- Synthetic XML discovery: namespace identity, cross-file calls, reuse and active cycles

## Production validation still outstanding

No target Mule application was supplied or run. The synthetic graph/fixture/plan validators belong to the authoring harness; they are not execution of a deployed generator, the native commands or an agent's real analysis. Preserve this distinction when using these files in another session.

Production readiness remains unverified until the actual application's schemas/runtime, source/handler/trait inventories, generated scenarios/mocks, valid fixture values, isolated runtime execution and no-op refresh have been assessed in its supported environments. Agent compliance, missing-library fallback semantics, subagent behavior, measured processor coverage and model/token claims are also unverified. Do not carry forward v11's 126-check count as a v12 execution result.
