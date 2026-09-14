# Version 10 validation record

Checked on macOS on 14 September 2026: **101 passed checks**. These are native-helper and synthetic output-gate checks, not passing Mule application tests or a model benchmark.

## What changed and what was exercised

The generic router now makes a binary decision from a namespace-aware recursive scan of the entire module: actual HTTP listener → REST; otherwise → non-HTTP. A nested fixture exercises XML files, DWL directories, shared/cyclic flow references, a listener using an alternate XML prefix, misleading outbound/config/comment text, and a mixed HTTP/scheduler module. Invalid XML, duplicate definitions, overlapping roots, missing roots and a symlink loop exercise failure/completeness handling. The index records call edges; it does not execute or semantically expand the cyclic flow.

The fixture-batch helper receives all prepared JSON sources in one manifest, preflights all rows before staging, copies original bytes, and re-reads every copy. Checks cover multiple owners, nested prepared-source paths, arrays, a null seed, optional JSON BOM preservation, invalid UTF-8, missing JSON, a malformed final row, duplicate/case-invalid/unsafe destinations and invalid roles/evidence/owner records. The malformed final row leaves no new staged batch. Role/evidence checks enforce structure, not truth of a RAML/DWL claim.

The existing scenario, exact source mock, required-header, request-name, APIKit/main error-suite, source-backed contract, extraction, local dependency and installer checks were rerun against version 10. Handler discovery now selects reachable handlers for every scope so an independent scheduler handler does not become a REST obligation; a separate full source audit must still account for out-of-scope/unowned declarations. Existing checks continue to reject omitted handlers reachable from the selected API graph.

The protocol for recursive RAML example/reference resolution, DWL import/function/producer tracing, caller-specific scenario expansion and refresh invalidation is embedded as instructions. No complete RAML or DWL parser was added. The harness supplies known scenario plans and prepared JSON; it does not prove an agent discovers every endpoint, creates correct raw backend data, or obeys continuation rules.

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

## Unverified and limits

- Windows PowerShell execution of installer, extraction, discovery, probes, fixture staging and gates. Windows blocks are supplied; macOS was the execution platform.
- Actual VS Code prompt discovery, agent adherence and results across model choices.
- Complete RAML examples/traits/types and DWL semantics on a real application; unavailable library constraints remain unverified under the retained source-backed policy.
- Mule/MUnit schema/runtime, actual error injection, batch/async completion and measured processor coverage.
- Real-application refresh/no-op behavior, token consumption, cost and latency.

No target Mule application or the user's Maven cache/other system was accessed. No prompt can guarantee correct output for every model and API complexity. An unavailable main API still stops REST project writes; unknown essential facts cannot be fabricated. All code needed by end users is embedded in the Markdown files. The authoring/regression harness is separate from the distributable package.
