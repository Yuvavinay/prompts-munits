# MUnit Automation — Portable Session Context

**Prepared:** 13 September 2026  
**Current implementation:** Version 7.0.0  
**Handoff status:** Current prompt package delivered; 59 macOS checks passed; Windows execution and real Mule runtime validation remain unverified.  
**Purpose:** Continue this project in another session or assistant, including Claude, without restarting requirements discovery.

## Read this first

We are building reusable Markdown instructions that make an existing coding agent generate and maintain Mule 4 / MUnit 2.x tests. We are **not currently generating tests for a supplied Mule application**. No real target application was provided or tested in this session.

Version 7 is the latest delivered package. Continue from its actual files; do not reconstruct them from this summary or revert to an older scaffold prompt. This context records requirements and decisions. It is not a request to install anything, run application tests, or rewrite the package automatically: follow the user's next task.

The latest reported failures were required headers hidden behind RAML endpoint `is: [...]`, unnamed HTTP request connectors, and missing APIKit/main error suites. Version 7 addresses these in the prompt templates and executable checks. Complete recursive scenario generation is still required. Success on a real Mule application has **not** been demonstrated.

## Files to bring into the next session

The handoff archive contains this layout:

```text
MUNIT-AUTOMATION-CONTEXT.md
munit-automation-v7/
  README.md
  VALIDATION.md
  munit-generate.prompt.md
  munit-http-listener.prompt.md
  munit-non-http-listener.prompt.md
```

The three prompt files are the implementation. README contains installation commands; VALIDATION contains evidence and limitations. This context alone explains the project, but the actual prompt files are necessary for reviewing or changing it reliably.

For another assistant, attach this context plus the package, or extract the archive and attach the Markdown files individually. Do not assume access to files, tools, source repositories or conversation history from the previous session. A normal chat can review the instructions; executing them against an application requires a tool-enabled environment with file access and the existing Mule toolchain.

The supplied installer targets VS Code chat with GitHub Copilot. **It is not a Claude-specific installer.** The instruction content is model-agnostic; any adaptation to another host’s slash-command or skill-discovery format is a separate task.

### Using this handoff in Claude or another session

Take `munit-automation-v7-handoff.zip`, which contains this context and all five current package Markdown files. If the destination cannot read the ZIP, extract it and attach the six Markdown files individually. Use the suggested opening message at the end of this document, replacing the final placeholder with the next task.

Read this context first, then the actual router/worker files before changing implementation. No local paths, previous attachments, authoring scripts or tools from this session are required to understand the delivered package. The commands needed by its users are embedded in the Markdown. The local regression harness is not included in the handoff; VALIDATION.md records what was exercised, so a new session must not claim to have rerun those checks without actually doing so.

The currently authorized handoff task is documentation only. No application tests were generated, no installation into the user's VS Code profile was performed as part of this handoff, and no target application was supplied. Continue with the user's next instruction rather than automatically installing or changing a project.

## Latest changes: version 7

- Resolve every applied endpoint/resource/method is-trait, including libraries, includes, nested traits, parameters and resourceType inheritance, into the effective request contract. Do not scan only inline headers or response headers. Cache definitions but preserve operation overrides and evidence.
- Put every effective required header in a concrete `<http:headers>` DataWeave map on every Mode A/B/C execution request. Required values come from actual contract/test-property evidence; unresolved values block rather than become invented credentials.
- Set `doc:name="Request to <actual request path>"` on each test request. Error requests also retain valid anchor headers and the response validator last.
- Inventory concrete source handlers independently from planned suites. Require APIKit cases in apikit-error-test-suite.xml and main cases in error-test-suite.xml. Do not omit suites because endpoint generation finished or the plan has no handler rows.
- Reconcile stale existing request headers/names and missing handler cases with narrow edits; preserve unrelated tests.
- The native plan schema is now version 2. Added sourceRoots, scope/sourceKind, entry/public flow names, effective contracts and handler inventory. Every test also records mode, handlerCases and request (null for direct tests). Handler identity is source-relative file plus one-based document-order ordinal among concrete on-error elements; no source doc:id is required for identification.
- The gate independently scans XML; checks handler/type-case ownership, actual suite/test existence, error modes, request names and canonical header maps. It does not parse RAML or execute DataWeave. Headers must first be resolved and independently audited by the agent.

Use the complete current package, replacing all three installed definitions together with the README update command. A fresh generation run must create a version 2 temporary plan. Windows execution and real Mule runtime behavior remain unverified. Native macOS regressions now include three suites/eight tests and omitted-error-suite/trait-header failures.

Details that must survive the handoff:

- `is: [...]` contains trait applications. Resolve their definitions and effective request headers; neither an empty-looking endpoint header block nor the lack of inline method headers proves that no headers are required.
- The emitted header map must match the source-derived contract and planned values. The check rejects a required header omitted from both the request plan and XML when it is still required by the effective contract. If the agent omits the header from the effective contract itself, the independent RAML audit must catch it; the native gate is not a RAML parser.
- APIKit and main error suites are separate required work queues after endpoint suites. The source scan prevents omission of both handler inventory and error-suite rows from passing merely because existing endpoint suites are valid.
- Handler discovery follows exact source references, including defaultErrorHandler-ref and reusable `<on-error ref="..."/>`. Application names such as `apikit-error-hander` and `main-errror-handlers` must be read verbatim; they are examples of source spellings, not hardcoded discovery names.
- APIKit Mode B injects a supported error at the actual router and executes the real handler body. Main Mode C keeps the router real and uses an evidenced business trigger. Both include required anchor-operation headers; every external call reached inside handler sub-flows still needs its own appropriate mock.
- An existing selected test missing the current headers, request name or handler scenario is stale. Patch affected content and add missing cases; unchanged valid tests and unrelated tests remain preserved.

## User objective and fixed environment choices

Developers perform a one-time installation of the instruction files, then invoke a slash command from the VS Code sidebar while selecting **Agent** and model **Auto**.

Supported main invocations:

```text
/munit-generate
/munit-generate all
/munit-generate orders.xml
/munit-generate orders.xml,customers.xml
/munit-generate errors
```

The no-argument command should interactively discover the application and ask for scope. An explicit argument already supplies scope and should not trigger another confirmation. File selection must work recursively, including nested source folders and dependencies in other files. Missing or ambiguous list items must not be silently dropped.

Constraints:

- No additional extensions, downloaded dependencies or package installations for the installation/file workflow.
- Use built-in Windows PowerShell 5.1/.NET and native macOS commands. PowerShell is not assumed to be built into macOS.
- An already working, authorized chat environment and compatible Mule/Java/Maven/MUnit setup remain prerequisites for execution.
- Keep the production project structure and implementation intact. Changes are limited to MUnit-related artifacts.
- All generation/extraction/validation commands belong inside the worker Markdown. One-time installers are enclosed in README; no extra script download is required.
- The router is small. Each worker is self-contained, without a shared helper-prompt dependency.
- Avoid duplicated explanations, repeated source reads and a separate refresh implementation.
- Do not pin a model or model-specific tool names. “High-end” was an aspiration for quality, not authorization to choose a particular model.

## Required architecture

| File | Responsibility |
| --- | --- |
| `munit-generate.prompt.md` | Discover actual HTTP/non-HTTP entry sources, interact with the user when scope is missing, and execute the appropriate worker |
| `munit-http-listener.prompt.md` | RAML-first HTTP/APIKit analysis, endpoint-specific suites, recursive scenario planning, mocks, error suites and synchronization |
| `munit-non-http-listener.prompt.md` | Any non-HTTP source, XML/DWL input provenance, recursive scenario planning, mocks, error handling and synchronization |

An outbound `http:request` is not an HTTP listener. A source-less sub-flow is not automatically a non-HTTP entry: trace its callers. Handle scheduler, MQ/JMS/VM/file/custom sources rather than assuming every non-HTTP application is a scheduler.

For a mixed application, plan HTTP first, including RAML, then non-HTTP. Missing required HTTP RAML blocks the entire selected invocation before project writes. Merge shared configuration/property plans and main-handler scenario ownership before applying either route. Never overwrite one route’s handler tests with the other’s.

Refresh is integrated into both workers. The current package intentionally has three prompt files, even though four older reference prompts were attached by the user.

## HTTP contract and suite ownership

The first HTTP generation step is resolving and extracting the RAML artifact from the local Maven repository using current POM/APIKit evidence. Resolve coordinates, inherited properties/profiles, classifier/type, repository overrides and local snapshot metadata. Do not choose the first ZIP or newest archive without evidence.

Missing RAML must clearly stop the run with no project writes. Current prescribed message:

```text
RAML not found. Please add the RAML artifact declared by pom.xml to the local Maven repository and rerun /munit-generate. No project files were written.
```

Extract into a fresh OS temporary directory, preserving includes and nested paths. Resolve API roots, traits, libraries, types and examples. Missing required fragments also block generation. No remote download or unrelated contract substitution.

The user requested extraction that “never fails.” Do not promise the impossible: missing, corrupt, inaccessible or unsafe archives must fail clearly and safely. Native commands are provided for both platforms; Windows execution is still unverified.

`all` means **every APIKit-routed method/path operation**, not every source filename and not only the first public flow. One XML can contain many endpoints. Cross-check contract operations, explicit APIKit mappings and actual operation flow definitions in both directions.

Each endpoint owns one suite. Media-type variants become scenarios within that endpoint suite; distinct listener/router contexts require explicit grouping. The suite’s test count comes from the flow’s scenarios. One suite per endpoint does **not** mean one test per endpoint.

`errors` discovers real entry anchors and fixtures but requires only the applicable handler suites, not new endpoint suites.

## Recursive traversal and scenario requirements

Before candidate test XML, build:

1. A definition table: exact flow/sub-flow name, source file/location, ordered processors, calls and DWL dependencies.
2. A caller-specific branch checklist: every construct and alternative, including empty/log-only branches.
3. A scenario table: full branch vector, input/attributes, error trigger, ordered processing, required external mocks, owning suite and test name.

For every `flow-ref`: search by actual name, read its complete definition, list processors, record backend selectors and recurse. This applies inside choices, scatter-gather, loops, batch, async, retries and handlers. Cache the definition’s text once; retain separate execution contexts for different callers and branches. Duplicate definitions and unresolved dynamic calls need resolution, not guessed filenames or hidden coverage gaps.

Required expansion:

- **Choice:** every `when` and `otherwise`, including implicit fall-through. Respect preceding false conditions and ancestor predicates. Expand nested/sequential branch combinations that form distinct feasible execution scenarios.
- **Sequence:** retain upstream and downstream processing around branches and sub-flow calls.
- **Scatter-gather:** one all-routes-primary scenario plus each meaningful route alternative with other routes still executing. Mock the union of calls from all concurrently executed routes. Add cross-route combinations only where observed interaction requires them.
- **Foreach/parallel-foreach:** actual representative nonempty iterations, nested alternatives/errors, and empty/multiple inputs where behavior changes.
- **Batch:** pre-batch, all steps and acceptance conditions, success/failure records, aggregators, on-complete and nested calls. Respect failure thresholds and actual propagation.
- **Try/handlers/raise-error:** success and each feasible type/condition/body alternative, with a real trigger and correct continue/propagate behavior.
- **Until-successful:** success and actual retry exhaustion induced by repeated inner failure; do not mock the scope itself.
- **Cache/object store:** distinct hit/miss and relevant Boolean conditions.
- **Async:** internal branches and errors remain in scope. Require supported bounded completion evidence; a fixed sleep or HTTP return alone does not prove completion.

No fixed minimum test count, duplicate padding, silent recursion cap or skipped branch justified by application size. A truly linear graph can have one scenario, but that must be established by traversal.

For each scenario, derive mocks from its **complete executed path**: prefix, selected branches, nested calls, concurrent routes, handler body and suffix. Do not mock mutually exclusive calls that do not run. Coalesce repeated occurrences only when their mock behavior is identical; differing concurrent outcomes require supported discrimination, not shared mutable counters.

Every reachable executable processor should have path evidence. The established measured-coverage targets are 100% for small graphs with at most 25 executable doc:id elements and at least 80% for larger graphs, without weakening branch obligations. Source-ID counts, planned paths and measured runtime coverage are distinct.

## Fixtures, mocks and configuration

Allowed output locations:

```text
src/test/munit/*.xml
src/test/resources/in/*.json
src/test/resources/out/*.json
src/test/resources/test-config.xml
src/test/resources/properties/app-properties-test.yaml
src/test/resources/properties/app-secrets-test.yaml
```

Normal output from an existing Maven test run may appear under target. Extraction, plans, staging and temporary validators stay outside the project in OS temp. Do not edit POM or production XML/DWL to make generation succeed.

HTTP base input comes from the operation’s RAML JSON example verbatim after validation. Branch copies change only actual discriminators and must satisfy complete predicates, required fields, JSON types, enums and other contract constraints. Bodyless requests use a null event seed and omit HTTP body. Do not invent business fields or invalidate a schema to force a business branch.

For non-HTTP, DWL **plus the XML producer of its input** establishes data origin. Separate entry payload/attributes, flow-derived values, backend results and properties. A scheduler querying DB before a transform usually has a null entry seed; the DB rows belong in out/. A message consumer may require a real input fixture. Do not pre-seed variables that production processors should calculate.

out/ contains raw backend connector results consumed by subsequent processing, not the final transform output or formatted application error response. A verified end-to-end pass-through is the narrow exception for using a RAML response example. Preserve target/targetValue and attribute semantics; unknown types require evidence.

Every mock return-payload element must load a literal out/ JSON resource:

```xml
<munit-tools:payload value="#[MunitTools::getResourceAsString('out/actual-file.json')]"
                     mediaType="application/json" encoding="UTF-8"/>
```

Use `read(MunitTools::getResourceAsString(...), 'application/json')` when a parsed result is required. Bare `payload`, `#[payload]`, inline JSON and dynamic/placeholder mock filenames are forbidden. The payload expression remains legitimate for execution loggers and HTTP bodies. Event-preserving mocks omit the returned payload instead of returning `#[payload]`.

Mock source-derived outbound operations: HTTP, DB, Salesforce, WSC, SFTP/FTP/file, object store, MQ/JMS/VM and other actual external connectors. Match current source doc:id/doc:name and necessary discriminators. Never mock resolved flow-refs, loggers/json-loggers, transforms, branches, loops, batch jobs or handlers to avoid real traversal. OS contains returns Boolean; store/remove can have an empty return when their semantics preserve the event.

Mock placement: behavior only. Return child order: variables → payload → attributes → error, omitting unused children according to the installed schema. A fully connector-free scenario may use the documented B16-exception comment; a connector-free local branch alone is insufficient.

Every suite must contain exactly one top-level core import:

```xml
<import file="test-config.xml" doc:name="Import"/>
```

Retain the suite’s own unique munit:config. Shared HTTP client configuration belongs in test-config.xml, not inline in each suite. Reuse or derive real loopback/port/protocol/TLS/base-path settings; no guessed configuration. Validate import/global behavior for a suite alone and the combined run.

Copy only the two specified test YAMLs from source, byte-for-byte. Do not copy environment variants, app-constants.yaml, app-errors.yaml or apikit-errors.yaml. Do not append fabricated credentials or delete unrelated pre-existing files to satisfy the allowlist. Copying YAML alone does not prove the test property mechanism is active.

## Error suites and XML conventions

Latest required names are `apikit-error-test-suite.xml` and `error-test-suite.xml`. Older reference names must not silently replace them.

Read actual error-handler.xml or the referenced equivalent, including default/global/inline handlers and nested calls. Cover every handler type, when condition and body branch. ANY is a matcher, never an injectable error type. A missing handler group does not require an invented empty suite; unreachable declared obligations must be reported.

REST modes:

- A: endpoint tests enable the actual main listener and public operation flow.
- B: APIKit handler tests enable main listener only and mock the exact router to a valid error.
- C: main-handler tests enable their actual listener/public pair and trigger the real business chain; no router-mock shortcut.

REST enable-flow-sources is the first test child. Execution starts with set-event containing payload only. Use real local HTTP requests, with a narrow explicit direct-sub-flow raise-error exception. Non-HTTP tests invoke flow-ref with sources disabled and only evidenced source-boundary event data.

HTTP error requests place the 200..599 success-status-code validator last. Do not use expectedErrorType for errors delivered as HTTP responses. Direct propagated-error tests need a supported local catch/guard so execution-end and validation still run.

All suites start with the XML declaration, no BOM/leading whitespace, and have one Mule root. Generated optional doc:id values may be omitted; when present, they must be unique lowercase hexadecimal UUIDs. Source mock identifiers remain verbatim. Add namespaces only for actual XML usage.

SUITE_BASE is the filename without .xml and ends in -test-suite. Owned test names begin SUITE_BASE-. Never use happy-path, error-path or default-choice in test names.

Each test has exactly four test-owned INFO loggers with message `#[payload]` and exactly one munit-tools:assert using:

```dataweave
import * from dw::test::Asserts
---
payload must notBeNull()
```

No assert-that, verify-call or MunitTools::equalTo. **This user-required assertion checks only non-nullness; it does not prove business output or HTTP status correctness.** Do not silently replace it with stronger assertions, erase useful existing assertions to enforce it, or fabricate non-null output. Report the policy conflict when necessary.

## Existing tests and the executable gate

all, named-file and multi-file runs all reconcile existing tests. Compare current source behavior and scenario evidence, not filename/doc:id alone. If already correct, preserve bytes/mtime and make no project writes or property recopies. Otherwise update affected selectors/fixtures/mocks, add missing scenarios or rewrite only changed test bodies. Preserve unrelated tests and useful custom content.

Check all shared fixture consumers. Existing combined suites can migrate source-matched selected tests to endpoint owners, validating destination first and removing only migrated originals. Do not blindly delete unmatched tests, disable failures, or overwrite whole suites. Obsolete cases require a separate cleanup decision.

Version 7 uses a frozen OS-temp plan.json with the version 2 fields described above, plus:

- `required`: scenario/branch/processor/handler obligation IDs.
- `exclusiveGroups`: mutually exclusive branch-arm IDs.
- `suites`: owning files, explicitly preserved unrelated tests, and all planned selected tests.
- Each planned test has `name`, `covers`, `inputs`, `mocks`, `mode`, `handlerCases` and `request`.
- Inputs record resource paths and scalar discriminator checks using JSON Pointer.
- Mocks record processor, exact selector map, source file, returned resource paths and error types.

The complete schema and native checks are embedded in each worker. The sample plan deliberately lacks an otherwise test: it is an illustration that must fail until completed, not a copy-ready plan.

The gate checks actual names and mock sets against the plan, resolves mock selectors against current source, parses referenced JSON and checks input discriminator values. It also catches uncovered obligations and mutually exclusive branches claimed by one test. **A plan generated from already-written tests defeats this design and is forbidden.** Independently audit the plan against source first.

## Validation status and limitations

The version 7 validation report records **59 passed macOS checks**. These are structural, installation, extraction and source/plan/XML regression checks, not 59 passing Mule application tests. The handoff ZIP was also checked for archive integrity and equality with the delivered files.

Validated locally on macOS:

- Prompt and Skill installation, body preservation, unchanged-install behavior, backups and selected installation safeguards.
- Native RAML extraction with valid nested data and controlled missing/unsafe cases.
- XML template parsing and selected REST/non-HTTP structural conventions.
- A synthetic nested-choice endpoint requiring **four tests and eleven mock bindings**.
- A combined **three-suite/eight-test** fixture including two APIKit and two main-handler cases, required header maps, exact request names and source-discovered handler obligations.
- Rejection of missing headers/names/error suites, including error suites and handler rows omitted together from the plan; traversal through defaults and reusable on-error refs.
- Rejection of one-test output, missing otherwise/nested/prefix/suffix mocks, wrong scenario names, stale/fabricated selectors, wrong/malformed fixtures, wrong input discriminators, missing imports and uncovered obligations.

Not validated:

- Windows PowerShell execution.
- Actual VS Code slash-command discovery or agent adherence.
- Complete traversal/generation/synchronization on a real Mule application.
- MUnit XSD/runtime, full DWL/predicate correctness, async/batch completion or measured coverage.
- Cross-model quality, token consumption, latency or cost.

Version 7 adds executable handler/request checks and is larger than version 6. No token, latency or cost reduction is claimed.

The native gate enforces consistency with the plan. It cannot prove that the agent discovered every real branch or that the tests exercise it at runtime. Current status is “instruction package plus tested structural/plan checks,” not “proven production-ready test generator.” Model-agnostic means no model-specific dependency/pinning, not identical success on all models or automatic compatibility with every host.

## History that must not be lost

The user supplied older unified scaffold, REST scaffold, scheduler scaffold and REST refresh prompts. Useful ideas retained: name-based flow resolution, per-branch checklists, recursive drilling, raw backend fixture analysis and current-source refresh.

Later requirements supersede contradictory older rules: no blind overwrite on all, no missing-flow skip presented as success, no assumption all non-HTTP sources lack inputs, no empty OS-contains return, no weaker “suite exists” completion gate, and no older error-suite naming overriding current names.

The user reported earlier outputs that covered only one endpoint, used bare payload in mock returns, omitted the required import, and later produced only one test per suite. The user explicitly said not to inspect the other system where those failures occurred; it was not accessed. Do not imply those failures have been fixed in that application. Version 7 is the current prompt-level remedy, with local synthetic regression evidence for headers and handler suites as well as the earlier scenario/mock cases.

A prior Vibes comparison concluded that MUnit generation, mock data, test updates and reusable skills are not exclusive to this build. Its proposed value is the explicit team-specific workflow. No superiority, cost or productivity claim has been measured. A longer justification was written for version 4; it is not the current implementation specification.

## Recommended next work, when requested

1. Read all three version 7 prompts and VALIDATION.md before editing. Keep requirements fixed unless the user changes them.
2. Use an authorized representative Mule application or a deliberately constructed runnable fixture. Do not search for the previously excluded remote system.
3. Compare the independently discovered call/branch graph to the agent’s plan, then compare plan to generated tests and runtime coverage.
4. Exercise nested choices, multiple endpoints in one XML, shared sub-flows, scatter-gather, iteration, handlers, caches and asynchronous/batch work.
5. Run Windows installation/extraction/gates in a real Windows environment; retain macOS coverage.
6. Make an implementation change and verify targeted reconciliation, then repeat unchanged and verify a true no-op.
7. Change failing logic or checks rather than merely adding more repeated instructions. Add relevant regressions and report static versus runtime evidence separately.

Use concise progress updates. Do not restart settled architecture/model/tool questions, assert unverified coverage, or promise that prompts cannot fail. Keep user-facing deliverables in portable Markdown with commands enclosed.

## Suggested opening message for another session

```text
Read MUNIT-AUTOMATION-CONTEXT.md and the attached munit-automation-v7
Markdown files. Continue from version 7, preserving the recorded requirements.
The central issue is complete recursive flow/sub-flow scenario generation and
mocking every external call on each actual execution path—not one generic test
per suite. Also retain effective RAML trait headers, Request to <path> names,
and source-required APIKit/main error suites. Distinguish the 59 tested macOS
checks from unverified RAML-resolution behavior and Mule runtime results.
Do not pin a model or assume access to the previous session's environment.
Use the attached files as the implementation, not this summary alone. If your
environment cannot read an attached file, identify the missing file before editing.

My next task is: [describe the review, change, or application validation needed].
```
