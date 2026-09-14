# Version 8 validation record

Checked on macOS on 14 September 2026: **70 passed checks**. These are installation, extraction, template and native source/plan/XML consistency checks, not passing Mule application tests.

## Scope of this update

The previous 59 checks were rerun against version 8. Added checks exercise the embedded local dependency probe with a synthetic Maven repository and actual ZIP files. The root API ZIP omits its library; a separate exact-version library ZIP is found in the local repository. A second probe finds a nested dependency under a wrapper directory, and the extraction command preserves sibling files. Different-version artifacts are not substituted; ambiguous candidates remain multiple candidates; corrupt/unsafe artifacts and missing repository locations produce distinct outcomes.

The harness manually supplies dependency coordinates and drives each probe/extraction step. It does not demonstrate a full RAML parser or prove that an agent discovers and drains every transitive reference. The bounded fallback search and source classification remain explicit agent responsibilities. No target application, real user Maven cache or excluded other system was inspected.

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

## Not verified

- Windows PowerShell installer, extraction and probe/gate execution. These blocks were reviewed, not executed on Windows.
- Actual VS Code discovery, Agent/Auto execution, model adherence or automatic continuation.
- Full RAML dependency/trait/resourceType/parameter interpretation and complete source classification on a real application.
- Actual missing-library state in the user's reported project.
- Mule XSD/runtime, supported error injection, complete predicates and backend fixture semantics, async/batch completion and measured coverage.
- Real-application narrow refresh/no-op behavior and cross-model quality, latency, tokens or cost.

## Packaging and continuity

The current three prompt files were based on retained version 7 QA copies because the earlier outputs folder was empty. The native scenario/handler/request gates were retained and their previous regressions rerun. The macOS installer retains Prompt/Skill formats, backups, idempotence and the tested preflight safeguards; the Windows installer has the corresponding documented interface but remains execution-unverified.

The context records the user report and current requirements. The handoff ZIP contains the context and all five package Markdown files. Archive integrity and byte equality with the delivered files are checked separately during packaging.

The native dependency probe is a candidate locator, not an identity decision or semantic resolver. Source/contract analysis and runtime verification remain required. A truly missing library remains a valid blocker, and a payload-not-null assertion cannot prove business correctness.
