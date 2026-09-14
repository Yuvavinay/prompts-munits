# Version 9 validation record

Checked on macOS on 14 September 2026: **80 passed checks**. These validate installation, extraction, probes and native source/plan/XML behavior. They are not passing Mule application tests or a cross-model benchmark.

## Changed behavior and evidence

Version 8 blocked generation when a referenced library was genuinely unavailable. Version 9 allows source-backed generation when the main API is available: contract status and missing references must be explicit, provenance excerpts must exist in current files, and each affected suite must contain the top-level pending-contract marker. The native output states RAML unverified. This intentionally relaxes complete RAML verification as a prerequisite to generating files; it does not infer the missing headers or constraints.

The new regression constructs a four-scenario suite with a missing-library record and actual local test-property evidence. It accepts that marked output and rejects missing markers/provenance/status, fabricated evidence, omitted known headers, and a missing reference relabeled as resolved. It also checks reconciliation back to resolved status with the pending marker removed. The existing scenario, handler, header, mock, installation and local-library-probe checks were rerun against the version 9 implementation.

The harness constructs plans and candidate XML from synthetic fixtures; it does not prove that an agent independently chooses valid fallback evidence. Literal provenance matches are not semantic proof that a header value or fixture is correct. The endpoint-at-a-time continuation policy and startup banner are instruction-level behavior, not verified live model behavior.

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

## Unverified and limits

- Windows PowerShell execution of installer, extraction, probes and gates.
- VS Code discovery, agent adherence, automatic continuation and per-model results.
- Full RAML resolution and source-backed evidence selection on a real application.
- Correctness/completeness of unknown headers and body constraints in missing libraries.
- Actual application initialization and Mule/MUnit runtime, error injection, DWL semantics, batch/async completion and measured coverage.
- Real-application targeted refresh/no-op behavior, token consumption, latency and cost.

The main API missing-root stop remains. Known essential values cannot be manufactured; unavailable essential case evidence must remain a reported gap. Empty or disabled suites are forbidden as completion substitutes. A missing library can prevent runtime initialization even after files are generated. Generated with RAML validation pending is not fully validated or runtime passed.

No access to the user's failing application or Maven cache occurred. The quoted model difference remains user-reported. The final package/context ZIP is checked for integrity and byte equality separately from these regressions.
