# Version 12.1.0 validation

**53 platform-neutral authoring/synthetic checks passed.** Only Python's standard library was used to read package text, parse canonical XML examples and exercise independent synthetic reference checks. No native macOS command, PowerShell, installer, VS Code, agent model or Mule/Maven runtime was executed for this revision. The user's actual Maven cache was not inspected.

## This revision

- The router remains unchanged except for its version. It only detects the source type and hands off.
- The HTTP worker must attempt local common-library recovery before accepting unresolved headers as empty or using the source-backed fallback. It searches exact loose files and cached ZIP entries, verifies the dependency's own identity/version and follows the actual applied trait chain. The known library filename cannot replace POM selection of the main API.
- Native extraction examples now check their temp parent against physical application/repository/.m2 locations before allocating or extracting. Windows rejects reparse ancestors and preserves full drive roots; the shell example resolves physical directories. Existing archive-entry safeguards remain. This was reviewed and checked as text, not executed.
- Both workers explicitly use the current host's tools and selected model without provider-specific APIs or mandatory subagents. This is portability by design, not a model benchmark.
- The existing fixture, recursive scenario, mock, error-suite and incremental reconciliation requirements remain. Non-HTTP fixture documentation now consistently describes raw backend output without the HTTP-only pass-through role.

## Executed checks

- Package metadata, compact binary router and self-contained workers
- Extraction text checks protected-root guards before allocation/extraction; no native execution
- Mandatory applied-trait library recovery, immutable POM root and model-neutral capability contract
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

## Size and verification limits

Router: 1,187 bytes; HTTP worker: 63,338; non-HTTP worker: 42,594. File size is not a measured token, cost or reliability result.

The synthetic validators are independent authoring checks. They do not execute the native snippets or prove an agent will discover all RAML/DWL semantics. XML examples were checked structurally; actual application schemas and runtime remain authoritative. Most source-to-scenario/header/mock completeness and publication checks are instructions for the executing agent, not a deterministic compiler.

Supported-platform commands, local artifact availability, real model adherence, actual generated application tests, isolated runtime results, no-op refresh and measured coverage still require validation in the target environment. Do not call this production-certified or carry forward prior releases' native/runtime validation claims. ZIP integrity and exact packaged-byte comparisons are performed separately during packaging.
