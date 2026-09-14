# Version 11 validation record

Checked on macOS on 14 September 2026: **126 passed native/synthetic checks**. These are not passing tests for a real Mule application or a cross-model benchmark.

## Evidence for this revision

The generic router is now a short read-only instruction file; it no longer embeds/runs a disk-writing index before routing. Static checks establish its file shape and the HTTP instruction order: POM/repository and extraction, common-library/applied-header resolution, then recursive application analysis. They do not prove that every agent follows that order. The native recursive index remains available inside both workers at the appropriate stage.

Both workers include a read-only pre-write scope guard. Synthetic tests accept the explicit MUnit artifact list without creating any paths, and reject POM/source/settings/command definitions, target, helper scripts, plans, other YAMLs, traversal, case collisions, symlink ancestors and scratch inside the module. Empty write lists are accepted without directory creation. A separate before/after content/path audit remains an instruction requirement: the guard checks proposed paths, not arbitrary future tool actions, source semantics or filesystem races. Verified OS-temp provenance of scratch remains the caller's responsibility.

The installer rejects custom destinations below a Maven project before creating command directories. Runtime instructions no longer permit target output in the original application: a provisioned, isolated temp mirror is required or runtime remains unverified.

Fixture manifest version 2 adds explicit required/type/enum constraints and provenance. The integrated macOS batch validates them before any staging destination is created. Checks exercise valid and invalid string/integer enums, missing required and optional fields, case-sensitive property names, JSON Pointers, explicit null, arrays and order-independent object enums. An APIKit-invalid role requires the exact expected required/type/enum failure; ordinary business fixtures cannot opt into expected failures. These records are compiled by the agent from source; the native gate cannot discover a schema fact that the plan omitted, evaluate arbitrary DWL or implement the entire RAML specification.

Earlier scenario/mock/header/error-suite/contract-marker/extraction/dependency/installation and recursive-worker-index/batch checks were rerun. Optional delegation and automatic continuation are instruction-level behavior, not a live VS Code benchmark. Subagents may reduce repeated parent context; no total-token or cost reduction is measured.

## Passed checks

- All three YAML metadata blocks and model-unpinned headers
- macOS Prompt/Skill installers, exact body preservation, idempotent installation, backups, missing-source and symlink safeguards
- All embedded Bash blocks pass syntax checks
- Four nested-choice scenarios accepted with eleven full-path backend mock bindings and source identity checks
- Rejects one-test suite when four scenarios are planned
- Rejects correct count but wrong scenario identity
- Rejects missing otherwise test
- Rejects missing nested sub-flow backend mock
- Rejects missing prefix call mock
- Rejects missing suffix call mock
- Rejects resolved flow-ref mocked instead of backend
- Rejects wrong source doc:id
- Rejects fabricated selector even when both plan and XML use it
- Rejects wrong existing return fixture
- Rejects wrong scenario input fixture
- Rejects wrong discriminator value inside the correctly named input fixture
- Rejects bare payload instead of file-backed return
- Rejects missing import
- Rejects duplicate import
- Rejects one test claiming mutually exclusive branches
- Rejects uncovered required branch despite four test files/names
- Rejects planned error replaced with success
- Rejects malformed nested backend fixture
- Rejects missing owning endpoint suite
- Accepts error-only mock when the scenario plan expects that error
- Accepts typed read(getResourceAsString(...)) return
- Accepts explicitly preserved legacy tests without counting them toward required scenarios
- HTTP A/B/C and non-HTTP templates parse with checked source/set-event/logger/assertion structure
- Native macOS RAML extraction accepts nested valid content; rejects missing, traversal and case collisions
- Rejects missing http:headers element
- Rejects request display name absent
- Rejects request display name without endpoint
- Rejects required trait header missing from XML
- Rejects wrong required header value
- Rejects required trait header omitted from both request plan and XML
- Rejects duplicate case-insensitive header name
- Rejects empty listener entry inventory
- Rejects HTTP listener omitted from all plan
- Accepts required-header name comparison without case sensitivity
- Accepts explicit empty map for a contract with no required headers
- Accepts three suites/eight tests including APIKit comma-separated types, main connectivity and ANY cases with explicit headers
- Rejects missing apikit-error-test-suite.xml
- Rejects apikit-error-test-suite.xml omitted from plan
- Rejects missing error-test-suite.xml
- Rejects error-test-suite.xml omitted from plan
- Rejects both handler inventory and error suites omitted despite current source declarations
- Rejects one comma-separated handler type omitted
- Rejects stale source handler condition
- Rejects APIKit global handler relabeled as local to evade owning suite
- Rejects missing headers in APIKit error suite
- Rejects missing request name in main error suite
- Rejects missing error response validator
- Rejects router mock used to claim main handler coverage
- Rejects missing external mock inside handler body
- File scope follows named error-handler refs and handler sub-flow calls
- Rejects file-scope reachable handlers omitted from both plan and suite list
- File scope follows defaultErrorHandler-ref
- File scope follows a reusable global on-error reference
- Non-HTTP worker gate accepts direct tests without HTTP request contracts
- Local probe returns explicit empty-directory evidence without claiming the whole dependency is unavailable
- Finds a separately cached exact-version library ZIP although the main API ZIP does not bundle it
- Exact coordinate lookup does not substitute an available newer library version
- A second probe locates an exact transitive library with a wrapper directory; safe extraction preserves sibling files
- Probe reports all ambiguous archive/entry candidates instead of choosing the first
- Probe reports an existing loose requested entry alongside archive candidates
- Probe rejects coordinate and entry path traversal
- Missing repository is a separate access/location failure, not a missing-library conclusion
- Corrupt dependency archive fails distinctly instead of silently becoming an empty result
- Probe rejects unsafe archive paths before extraction
- Existing library archive lacking the requested entry returns evidence for the remaining fallback search
- Creates and accepts a four-test source-backed suite with a missing library, actual file provenance and pending-contract marker
- Rejects source-backed contract without suite marker
- Rejects source-backed contract lacking actual source provenance
- Rejects fabricated source-backed evidence even if the plan claims it
- Rejects source-backed status without unresolved-reference records
- Rejects missing library relabeled resolved without removing the unresolved state
- Rejects contract with no explicit completeness status
- Rejects source-backed suite still omitting known required header
- Rejects pending-contract marker hidden inside a test instead of at suite level
- Accepts reconciliation back to resolved contract status and removal of the pending suite marker
- All workers embed identical standalone native discovery; recursive protocol and version-3 schema agree
- Recursive discovery finds nested XML/DWL, scoped call edges and a cyclic/shared flow graph without mistaking outbound/config/comment text for an HTTP source
- Namespace-aliased inbound HTTP listener selects REST after a full recursive module scan
- HTTP plus independent scheduler module still selects the single REST route
- Malformed source cannot fall through to a false non-HTTP route
- Discovery rejects duplicate flow definitions across nested files
- Discovery rejects a filesystem symlink cycle instead of recursing indefinitely
- Overlapping configured source roots do not duplicate definitions
- Missing/unreadable configured root leaves discovery incomplete rather than guessing a route
- One fixture batch stages all prepared owners, nested-source examples, backend array and null seed while preserving every byte
- A malformed final JSON row prevents the entire fixture batch from being staged
- Fixture batch rejects duplicate destination
- Fixture batch rejects case-varying destination
- Fixture batch rejects path traversal
- Fixture batch rejects response example used as backend role
- Fixture batch rejects missing fixture evidence
- Fixture batch rejects missing fixture owners
- Fixture batch rejects missing prepared source
- Fixture batch preserves optional JSON BOM bytes; XML no-BOM rule is not incorrectly applied to source JSON
- Fixture batch rejects invalid UTF-8
- REST source-handler gate excludes an independently owned scheduler handler while retaining selected-graph checks
- Router is instruction-only/read-only; HTTP contract/header stages precede recursive graph analysis; no target-output exception or routine continuation menu remains
- Installer rejects a project-local custom destination without creating .vscode or command files
- Integrated batch accepts effective string and integer enums on a valid base request
- Fixture constraints reject known enum violation
- Fixture constraints reject wrong JSON type
- Fixture constraints reject missing required field
- Fixture constraints reject missing constraints inventory
- Fixture constraints reject constraint without provenance
- Fixture constraints reject malformed JSON Pointer
- Fixture constraints reject case-changed property name
- Fixture constraints reject empty declared enum
- Fixture constraints reject expected failure on a valid-business role
- Integrated batch accepts a deliberately invalid APIKit input only with its exact expected enum violation
- Deliberate APIKit case rejects a different failure from the one expected
- APIKit-invalid role cannot silently omit its expected violation
- Missing optional fields pass while required fields retain their checks
- Integrated batch supports structured array enums and explicit null root constraints
- Object enum equality ignores key order without ignoring value types
- Write guard accepts only the MUnit artifact plan and creates no project directories or files
- Write guard rejects POM/source/settings/commands/target/plans/helper scripts/extra YAML/traversal destinations
- Write guard rejects scratch at the application root
- Write guard rejects scratch nested inside the application
- Write guard rejects case-colliding planned suite destinations
- Write guard rejects a symlink ancestor instead of writing through it
- Write guard accepts a no-op write list without creating artifact directories

## Unverified

- Windows PowerShell execution; no Windows runtime was available for installer, extraction, constraints, path guards or other native blocks.
- Actual VS Code slash-command behavior, subagent availability, instruction adherence, continuation and model-specific results.
- Complete RAML trait/type/example semantics and DWL/branch correctness on a real application.
- Actual source-tree no-op/targeted refresh behavior and isolated Mule/MUnit schema/runtime, error injection, async/batch completion and measured coverage.
- Token consumption, latency and cost across serial/delegated execution.

No target Mule application, the user's Maven cache or the excluded other system was accessed. Only synthetic sources and local package files were used. A missing root RAML still stops REST project writes; a locally cached common library must be searched before any fragment fallback. Unknown values cannot be manufactured and a non-null assertion does not prove business correctness.
