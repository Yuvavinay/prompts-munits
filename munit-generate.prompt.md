​---
mode: 'agent'
name: 'munit-generate'
version: '1.4.0'
kernel-rev: 'K-2026.08.b'
description: 'Auto-detect and generate (or sync) MUnit test suites for REST, Scheduler, or mixed Mule 4 projects. Usage: /munit-generate <file.xml> | all | errors'
argument-hint: '<file.xml> | all | errors'
author: 'Yuva Jilagam'
---

# Generate MUnit Test Suite — Unified (REST + Scheduler Auto-Detect)

> **Self-contained.** Full kernel K0–K9, RULE ZERO, both deltas, Phase 0–3 router. No embedded scripts.

---

## Changelog

| Version | Date    | Change |
|---------|---------|--------|
| 1.4.0   | 2026-08 | K8 gains small-flow 100% target (≤25 doc:id elements → every element must be covered); K8 Step 1 table adds `<until-successful>` fork; K8 Step 3 clarified: all elements with `doc:id` count toward MUnit coverage, not just backend connectors; REST Step 4.1 upgraded to full per-branch checklist with branch-count GATE (now matches munit-generate-api); scheduler section gains explicit async branch coverage and discriminator guidance |
| 1.3.3   | 2026-08 | B21/B22 ban invalid doc:id UUIDs (non-hex chars and duplicates within file); B23 bans inline JSON in mock then-return without readUrl; K4 UUID format note; K9 gains UUID-validity, unique-doc:id, and flow-source-traceability checks |
| 1.3.2   | 2026-08 | Step 0e routing split: `all` always marks every operation GENERATE_MODE (`SYNC_MODE` never fires for `all` — every operation gets a full fresh generation pass and any existing suite is overwritten in place); named `<file.xml>` still routes to `/munit-sync` when the suite already exists; Phase 1 routing table expanded to three rows with explicit `all` / named distinction |
| 1.3.1   | 2026-08 | RAML made a global pre-flight gate for the REST route — when the route includes REST and the RAML artifact cannot be obtained, the entire command halts before any suite/JSON/property is written (SCHEDULER-only route stays exempt); the "no RAML dependency" branch is now an explicit hard stop; coverage wording softened |
| 1.3.0   | 2026-08 | Kernel alignment across REST/Scheduler/Unified (kernel-rev K-2026.08.b): scheduler-scoped bans renumbered B18–B20 to remove the `B14`/`B15` collision with the REST bans in this same file; Windows RAML extraction now creates its temp target (`tar -C` won't); Mode C (APIKit) renumbered MC1/MC2/MC3; for ROUTE = BOTH, Mode C now runs exactly once (HTTP-anchored) instead of twice; B12 step reference made step-agnostic; coverage wording softened from "guarantees" to "typically reaches" |
| 1.2.0   | 2026-08 | Mandatory procedural Step 4 with 4.2a–4.2f sub-steps; HARD GATE before test generation; drill-through gate in pre-write scan and K9; path-coverage algorithm |
| 1.1.0   | 2026-08 | Extractor approach aligned with REST/Scheduler builds — no embedded scripts |
| 1.0.0   | 2026-08 | Initial production release — REST + Scheduler auto-detect router (Phase 0–3); self-contained kernel K0–K9 + both deltas |

---

# Core build rules

## K0 — Operating principles

1. **Rarely halt.** Hard-halt only when no correct output is possible — the target file is missing, or no RAML can be obtained (Step 2b). Otherwise degrade with a plain `NOTE:` and still produce the best correct suite.
2. **Model-agnostic.** Assume only the K1 capabilities — never a specific model, vendor, or context size; never gate on token budget.
3. **Tools only, one exception.** Use READ / WRITE / EDIT / SEARCH for everything. The sole terminal use is the native OS extraction in REST Step 2c that unpacks the RAML artifact into the OS temp folder: `unzip` on Mac/Linux, `tar -xf` on Windows 10+ — both built into the OS, nothing to install, no scripts written anywhere. On Windows the extraction line also creates its temp target first (`tar -C` does not create the destination directory); this directory creation is part of the single permitted extraction step. The Scheduler phase performs no extraction at all (tools-only). WRITE creates missing parent folders.
4. **Write scope — `src/test/munit/` and `src/test/resources/` only.** Never create or modify any file outside these two directories. READ anywhere (incl. `src/main`, `pom.xml`, `~/.m2`); WRITE / EDIT only inside the two test directories. **Never READ or SEARCH any path under `.github/` — prompt files are not source data.** Any temp file goes to the OS temp dir (`%TEMP%` / `C:\tmp` on Windows, `$TMPDIR` / `/tmp` on macOS/Linux) — never the project.
5. **Write immediately.** WRITE each JSON file and each suite the moment its content is ready; never leave output as chat text (unless no WRITE exists — see K1 fallback).
6. **Fresh start — no memory, no carry-over.** Discard all discovery tables, UUIDs, header sets, and connector IDs from any prior run; re-read every source file on this run. **Never use the conversation's auto-memory, context memory, or any external memory system as a source of values** — every `doc:id`, flow name, connector reference, required header, and UUID must be read directly from project source files on this run. A value recalled from memory and not verified against the current source file is wrong by definition.

---

## Progress updates (what the user sees)

Post a short, plain-language line before each major step using prefix `→`, and `✅ Done: <suite> (<N> tests)` after each finished suite. Write like a developer to a teammate — state the action, never the rulebook. **Never print internal labels, section letters, rule numbers, ban numbers, or variable names.**

Example run:
```
→ Reading project config…
→ Found 3 flows: create-enrollment, get-members, delete-member
→ Extracting the API contract from .m2…
→ Analysing create-enrollment…
→ Writing test data…
→ Writing create-enrollment-test-suite.xml…
✅ Done: create-enrollment-test-suite.xml (5 tests)
```
End with a one-line summary (suites written, total test count, anything skipped). Report problems in plain words — e.g. `Couldn't find sub-flow 'process-order' — skipping it`.

---

## K1 — Capabilities (not tool names)

Bind to whatever the environment provides:

| Capability | Meaning | Common bindings |
|---|---|---|
| **READ** | Read a file's contents | `Read`, `readFile`, `view` |
| **WRITE** | Create or overwrite a file | `Write`, `writeFile`, `create_file` |
| **EDIT** | Modify part of a file | `Edit`, `editFile`, `str_replace` |
| **SEARCH** | Find files or text across the tree | `Grep`, `Glob`, `search/codebase` |
| **RUN** | Execute a terminal command | `terminal`, `run_command` — used **only** for the native OS unzip in Step 2c |

Rules:
- **SEARCH empty-result:** if SEARCH returns nothing for a name seen in a prior READ, stop using SEARCH this run and resolve by direct READ; don't retry the same query.
- **WRITE-confirm:** after each WRITE, READ it back; if missing, print `WRITE PENDING — accept the change in your editor, then re-run.` and halt.
- **No-WRITE fallback:** if no WRITE exists, output each file in a fenced block headed `### File: <path>`, still run the K9 scan, and end `INLINE OUTPUT MODE — copy each block manually.`

---

## K3 — RULE ZERO (hard bans B1–B17)

Scan the assembled XML before **every** WRITE. On any match: apply the fix, note it briefly (e.g. `auto-fixed: missing enable-flow-sources`), re-scan, and WRITE only when zero matches remain. **Never abort or halt for a ban — auto-fix and continue.**

| # | Forbidden pattern | Fix |
|---|---|---|
| B1 | `assert-that` in any element | Replace the whole assert with the **Canonical Validation Block** (K4) |
| B2 | `verify-call` anywhere | Delete the entire `<munit-tools:verify-call>…</munit-tools:verify-call>` block |
| B3 | `"#[(output` (parenthesised DWL) | Rewrite as `"#[output` — drop the wrapping `( )` |
| B4 | `processor="json-logger:logger"` in a `mock-when` | Delete that `mock-when` block (json-logger does no external I/O) |
| B5 | test name contains `happy-path` | Rename `<SUITE_BASE>-<connector-desc>-success` |
| B6 | test name contains `error-path` | Rename `<SUITE_BASE>-<connector-desc>-<error-slug>` |
| B7 | test name contains `default-choice` | Rename `<SUITE_BASE>-<what-otherwise-represents>` |
| B8 | `<?xml` ≠ 1, `<mule>` ≠ 1, or `</mule>` ≠ 1 | Merge into ONE XML document |
| B9 | `mock-when` inside `<munit:validation>` | Move it to `<munit:behavior>` |
| B10 | `mock-when` inside `<munit:execution>` | Move it to `<munit:behavior>` |
| B11 | `MunitTools::equalTo` | Replace with the Canonical Validation Block (K4) |
| B12 | `processor="mule:flow-ref"` mock on a success path or inside `<async>` | Replace with backend connector mocks **only when Step 4 already produced a resolved connector list for that sub-flow**. Detection: any `<mock-when processor="mule:flow-ref">` in a non-error test (no `expectedErrorType`, no `<munit-tools:error>` in behavior). Fix: look up that sub-flow in the resolved connector inventory built during the drill-through earlier this run. If the sub-flow is RESOLVED (connectors listed) → remove the flow-ref mock and add those connector mocks. If the sub-flow is UNRESOLVED (K5 could not find it) → **keep the flow-ref mock** — a passing test with a flow-ref mock covers the flow-ref element itself; a failing test with no mock covers nothing. Add `<!-- NOTE: sub-flow <name> unresolved — flow-ref mock retained; coverage limited to this element -->` inside the `mock-when` to flag it for manual follow-up. Never remove a flow-ref mock unless its replacement is already in hand. |
| B13 *(REST)* | Any script written anywhere, or any terminal command other than the native OS extraction in Step 2c (which, on Windows, also creates its OS-temp target folder) | Never — the OS extraction is the only permitted terminal use, exclusively to unpack the RAML artifact into OS temp |
| B14 *(REST)* | `<flow-ref>` inside `<munit:execution>` | Replace with `<http:request>`; exception: raise-error tests that invoke directly via `<flow-ref>` (test type 5 in Steps 7–8) |
| B15 *(REST)* | `<munit:execution>` missing `<munit:set-event>` | Add it as the **first child** of `<munit:execution>` |
| B16 | `<munit:behavior />` or empty `<munit:behavior>` | Mock at least one reachable connector — behavior is NEVER empty |
| B17 *(REST)* | `<munit:test>` missing `<munit:enable-flow-sources>` as its first child | Add per the enable-flow-sources rules in K4: Mode A HTTP tests → `<MAIN_LISTENER_FLOW>` + `<PUBLIC_FLOW>`; Mode A raise-error flow-ref tests → `<PUBLIC_FLOW>` only; Mode B → `<MAIN_LISTENER_FLOW>` only; Mode C → `<MAIN_LISTENER_FLOW>` + `<PUBLIC_FLOW>` |
| B18 *(Scheduler)* | Any script written anywhere, or any terminal command (scheduler builds are strictly tools-only — no RAML, no extraction step) | Never — no shell commands, no code of any kind in the scheduler phase |
| B19 *(Scheduler)* | `<munit:enable-flow-sources>` anywhere in a scheduler suite | Delete every occurrence — scheduler flows invoke via `<flow-ref>`, no HTTP listener |
| B20 *(Scheduler)* | Inline JSON in an XML attribute (`value="#[{ … }]"`) in a scheduler suite | Extract to `out/*.json`, reference with `readUrl('classpath://out/…')` |
| B21 | Any `doc:id` value containing a character outside `[0-9a-f]` — e.g. the letters g through z appear anywhere in the value | Replace the entire `doc:id` with a fresh valid UUID using only 0-9 and a-f |
| B22 | Two or more elements in the same file sharing the same `doc:id` value | Assign a new unique UUID to every duplicate; after fixing, all `doc:id` values in the file must be globally unique |
| B23 *(REST)* | Inline JSON literal without `readUrl(...)` in `<munit-tools:payload value>` inside `<munit-tools:then-return>` — any pattern matching `"#[{…}]"` or `"#[output … --- {…}]"` where the DWL body embeds JSON directly rather than reading from a file | Extract the JSON to `out/<file>.json`; replace the `value` with `"#[output application/json --- readUrl('classpath://out/<file>.json', 'application/json')]"` |
| B24 *(REST)* | `<munit:variables>` inside `<munit:set-event>` in `<munit:execution>` | Delete the entire `<munit:variables>` block — the `set-event` in `<munit:execution>` must contain ONLY `<munit:payload>`; the flow sets its own variables from the incoming HTTP request, headers, and transforms, so pre-seeding them in the test is wrong and will conflict with what the real flow does |
| B25 *(REST)* | v2-style fields in a v1 `/enrollment` (non-v2) input JSON file — specifically any of: `reltioId`, `persona`, `phoneNumbers` (array), `emails` (array), `conditions` (array), `externalIds`, `product`, `isHipaaValidationRequired`, or `consents` as an array | Replace with v1-RAML-compliant fields: `uid` (string), `phoneNumber` (string), `phoneNumberType` (string), `email` (string), `conditions` (string), `consents` as an object with boolean fields (`textConsent`, `tcpaConsent`, `emailConsent`, `marketingConsent`, `ongoingSupport`, `medicalResearchConsent`, `marketingSMSEnabled`); `consentInfo` is a separate top-level object holding `envelopeId`; the v1 flow reads `vars.envelopeId = payload.consentInfo.envelopeId` — the v1 schema has `additionalProperties: false` so any banned field causes `APIKIT:BAD_REQUEST` |

> B13–B15, B17, B23–B25 are **REST-scoped** (apply only in the REST phase); B18–B20 are **Scheduler-scoped** (apply only in the Scheduler phase); B21 and B22 are **global** (apply in every suite regardless of type). Numbers do not overlap, so a given ban number means the same thing everywhere in this file. Ban numbers are maintainer-facing only — never print them to the user.

---

## K4 — Canonical XML building blocks

Every value below is fixed. Substitute only `<PLACEHOLDER>` tokens.

**`<UUID>` format (mandatory for every `doc:id` and element ID):** a valid UUID contains exactly 32 hex digits — characters 0-9 and a-f **only** (letters g through z are not hex digits). Format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. Every element gets its own **unique** UUID — no two elements in the same file may share a `doc:id`. Sequential placeholder patterns such as `a1b2c3d4-e5f6-g7h8-i9j0-k1l2m3n4o5p6` are **invalid** — g, h, i, j, k, l, m, n, o, p are not hex characters.

### Suite root

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mule xmlns:munit="http://www.mulesoft.org/schema/mule/munit"
      xmlns:munit-tools="http://www.mulesoft.org/schema/mule/munit-tools"
      xmlns="http://www.mulesoft.org/schema/mule/core"
      xmlns:doc="http://www.mulesoft.org/schema/mule/documentation"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="
        http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd
        http://www.mulesoft.org/schema/mule/munit http://www.mulesoft.org/schema/mule/munit/current/mule-munit.xsd
        http://www.mulesoft.org/schema/mule/munit-tools http://www.mulesoft.org/schema/mule/munit-tools/current/mule-munit-tools.xsd">
    <munit:config name="<BASE_NAME>-test-suite.xml" doc:id="<UUID>" />
    <!-- REST suites add xmlns:http + its schemaLocation entry + the import below -->
    <!-- test cases -->
</mule>
```

- **REST suites** add `xmlns:http` and its `xsi:schemaLocation` entry, and immediately after `<munit:config>` add:
  `<import doc:id="<UUID>" doc:name="Import" file="test-config.xml" />`
  (the `file` value is always the literal `test-config.xml`).
- Add `xmlns:anypoint-mq` / `xmlns:apikit` / `xmlns:db` / `xmlns:vm` / `xmlns:jms` **only** when the suite actually uses those processors, each with its matching `schemaLocation`.
- The first byte of the file is `<`. Never a BOM, never leading whitespace before `<?xml`.

---

### Test naming

Let `<SUITE_BASE>` = the suite file's own name **without** the `.xml` extension — it always ends in `-test-suite` (e.g. `create-enrollment-test-suite`, `api-test-suite`, `error-handler-test-suite`).

**Every `<munit:test name>` MUST start with `<SUITE_BASE>-`**, followed by a scenario descriptor. So every test name contains the substring `-test-suite`. The descriptor conveys the connector / branch / error meaning and still obeys B5–B7 (no `happy-path` / `error-path` / `default-choice`).

Examples for `create-enrollment-test-suite.xml`:
- `create-enrollment-test-suite-http-request-success`
- `create-enrollment-test-suite-db-select-connectivity`
- `create-enrollment-test-suite-choice-premium`

Wherever a step writes `<BASE_NAME>-<desc>` as a test name, read it as `<SUITE_BASE>-<desc>`.

---

### Canonical Validation Block — exactly one assert, two trace loggers

```xml
<munit:validation>
    <logger level="INFO" doc:name="Log Validation Start" doc:id="<UUID>"
            message="#[payload]" category="${log.category.base}.<BASE_NAME>.validation.start" />
    <munit-tools:assert doc:name="Assert payload" doc:id="<UUID>">
        <munit-tools:that><![CDATA[#[import * from dw::test::Asserts
---
payload must notBeNull()]]]></munit-tools:that>
    </munit-tools:assert>
    <logger level="INFO" doc:name="Log Validation End" doc:id="<UUID>"
            message="#[payload]" category="${log.category.base}.<BASE_NAME>.validation.end" />
</munit:validation>
```

The validation block is **never** omitted — not for `on-error-propagate`, not for raise-error tests, not for any test type. Never use `verify-call`, `assert-that`, `times`, or `MunitTools::equalTo`.

---

### Four-logger rule

Every test has **exactly four** `INFO` loggers with `message="#[payload]"`:
- Execution-start and execution-end — both in `<munit:execution>`
- Validation-start and validation-end — both in `<munit:validation>` (shown above)

No test ever has fewer than four.

---

### `then-return` child order (XSD-enforced)

Within any `<munit-tools:then-return>`, children MUST appear in this order:
`variables` → `payload` → `attributes` → `error`

Never place `attributes` before `payload`. Include only the children the mock actually needs.

---

### Success mock (payload from a file)

```xml
<munit-tools:mock-when processor="<PROCESSOR>" doc:name="Mock <CONNECTOR_DOC_NAME>"
                       doc:id="<MOCK_UUID>">
    <munit-tools:with-attributes>
        <munit-tools:with-attribute attributeName="doc:id" whereValue="<CONNECTOR_DOC_ID>" />
    </munit-tools:with-attributes>
    <munit-tools:then-return>
        <munit-tools:payload value="#[output application/json --- readUrl('classpath://out/<OUT_FILE>.json', 'application/json')]"
                             mediaType="application/json" encoding="UTF-8" />
    </munit-tools:then-return>
</munit-tools:mock-when>
```

**`<CONNECTOR_DOC_ID>`** is the verbatim value of the `doc:id="…"` attribute on the actual connector element in the source XML — for example `<http:request doc:id="95b70e00-7d70-46f9-85dc-a84263ddf24a" …>` → `whereValue="95b70e00-7d70-46f9-85dc-a84263ddf24a"`. Always read it directly from the source file; never guess or copy from another connector. Match **by `doc:id` alone** — never add a `doc:name` with-attribute alongside it (extra attribute filters over-constrain the match and break when names differ between environments).

- **Attributes on HTTP mocks** — add `<munit-tools:attributes value="#[{'statusCode': 200}]" />` **after** payload whenever any downstream expression (a `<choice>` `<when>` condition, DWL transform, or `<foreach>` collection expression) reads `attributes.statusCode` or any other `attributes.*` field on the mocked connector's response. Use the inline form `#[{'statusCode': 200}]` for simple status-code-only cases — no separate file needed. For complex attributes shapes (multiple fields, nested objects), write to `out/<OUT_FILE>-attributes.json` and reference with `readUrl(...)`, `mediaType="application/java"`.
- **Error mock** — swap the `then-return` body for `<munit-tools:error typeId="<ERROR_TYPE>" />`.
- Match `os:*` mocks by `doc:id` **only** (never `doc:name`). `os:retrieve` with a `target` returns via `<munit-tools:variables>` (key = the target var); without a target, via `<munit-tools:payload>`. `os:store` / `os:remove` / `os:contains` → empty `<munit-tools:then-return />`.

---

### APIKit Router mock (Mode B only)

`apikit:router` is matched by `config-ref` **only** — never by `doc:id` or `doc:name`. `<APIKIT_CONFIG_REF>` is the `name=` attribute of `<apikit:config>` in `src/main/mule/api.xml`.

```xml
<munit-tools:mock-when processor="apikit:router" doc:name="Mock APIKit Router" doc:id="<UUID>">
    <munit-tools:with-attributes>
        <munit-tools:with-attribute attributeName="config-ref" whereValue="<APIKIT_CONFIG_REF>" />
    </munit-tools:with-attributes>
    <munit-tools:then-return>
        <munit-tools:error typeId="<ERROR_TYPE>" />
    </munit-tools:then-return>
</munit-tools:mock-when>
```

---

### HTTP request (REST execution)

```xml
<http:request method="<METHOD>" doc:name="Request to <RESOURCE_PATH>" doc:id="<UUID>"
              path="<RESOURCE_PATH>" responseTimeout="120000" config-ref="<CONFIG_REF>">
    <http:headers><![CDATA[#[output application/java
---
{
    <REQUIRED_HEADERS>
}]]]></http:headers>
    <!-- error tests only, as the LAST child: -->
    <!-- <http:response-validator><http:success-status-code-validator values="200..599" /></http:response-validator> -->
</http:request>
```

Child order inside `<http:request>`: headers → uri-params → query-params → response-validator.
`path` is the resource path only — the base path lives on the HTTP config; prefixing it here double-prefixes → 404.

---

### Canonical REST test structure

`<munit:enable-flow-sources>` is the **first child** of every `<munit:test>` in a REST suite (B17). Its content varies by mode per the table below.

```xml
<!-- Mode A — HTTP path (success, connector error, choice, try-catch, batch, foreach, cache) -->
<munit:test name="<SUITE_BASE>-<scenario-desc>" description="<scenario-desc>">
    <munit:enable-flow-sources>
        <munit:enable-flow-source value="<MAIN_LISTENER_FLOW>" />
        <munit:enable-flow-source value="<PUBLIC_FLOW>" />
    </munit:enable-flow-sources>
    <munit:behavior>
        <!-- connector mocks (K4) + os:* mocks -->
    </munit:behavior>
    <munit:execution>
        <munit:set-event doc:name="Set Event" doc:id="<UUID>">
            <munit:payload value="#[output application/json --- readUrl('classpath://in/<IN_FILE>.json', 'application/json')]"
                           mediaType="application/json" encoding="UTF-8" />
        </munit:set-event>
        <logger level="INFO" doc:name="Log Execution Start" doc:id="<UUID>"
                message="#[payload]" category="${log.category.base}.<BASE_NAME>.execution.start" />
        <http:request method="<METHOD>" doc:name="Request to <RESOURCE_PATH>" doc:id="<UUID>"
                      path="<RESOURCE_PATH>" responseTimeout="120000" config-ref="<CONFIG_REF>">
            <http:headers><![CDATA[#[output application/java
---
{
    <REQUIRED_HEADERS>
}]]]></http:headers>
        </http:request>
        <logger level="INFO" doc:name="Log Execution End" doc:id="<UUID>"
                message="#[payload]" category="${log.category.base}.<BASE_NAME>.execution.end" />
    </munit:execution>
    <munit:validation>
        <logger level="INFO" doc:name="Log Validation Start" doc:id="<UUID>"
                message="#[payload]" category="${log.category.base}.<BASE_NAME>.validation.start" />
        <munit-tools:assert doc:name="Assert payload" doc:id="<UUID>">
            <munit-tools:that><![CDATA[#[import * from dw::test::Asserts
---
payload must notBeNull()]]]></munit-tools:that>
        </munit-tools:assert>
        <logger level="INFO" doc:name="Log Validation End" doc:id="<UUID>"
                message="#[payload]" category="${log.category.base}.<BASE_NAME>.validation.end" />
    </munit:validation>
</munit:test>
```

**Enable-flow-sources rules:**

| Mode | Flows to enable |
|---|---|
| Mode A — all test types via HTTP (success, connector error, choice branches, try-catch, batch, foreach, cache miss/hit) | `<MAIN_LISTENER_FLOW>` + `<PUBLIC_FLOW>` |
| Mode A — raise-error test invoking directly via `<flow-ref>` (B14 exception; `<on-error-continue>` wraps the raise) | `<PUBLIC_FLOW>` only |
| Mode B — APIKit error suite | `<MAIN_LISTENER_FLOW>` only |
| Mode C — Error-handler suite | `<MAIN_LISTENER_FLOW>` + `<PUBLIC_FLOW>` |

Error tests that go via HTTP keep both flows enabled — only the mock's `then-return` differs. Async tests keep both flows and add the sleep element after `<http:request>`.

---

## K5 — Sub-flow discovery

Sub-flow names and their filenames often differ — resolve by **searching file contents**, never by guessing filenames.

Build a `name → file` map once: SEARCH `src/main/mule/**/*.xml` for `<flow name="` and `<sub-flow name="`. A name in two files is ambiguous — mock it by `doc:id`. Resolve each `<flow-ref name="X">` via the map. If still unresolved: re-scan files already READ this run; SEARCH `<sub-flow name="X"`; READ `common/common-flows.xml` and `.../error-handlers.xml`; if still not found, note `Couldn't find sub-flow 'X' — skipping it` and continue.

In each resolved sub-flow, record every backend connector (including inside `<try>`, `<choice>`, `<scatter-gather>`, `<foreach>`, `<parallel-foreach>`, `<batch:step>`, `<async>`) with its processor type, `doc:name`, and `doc:id`. **Backend connectors** (mock each; `json-logger:logger` is not one — B4): `http:request` · `db:*` · `salesforce:*` · `wsc:consume` · `sftp:*` · `ftp:*` · `file:*` · `os:store|retrieve|contains|remove` · `anypoint-mq:*` · `jms:*` · `vm:*` · custom.

**Per-path inventory (mandatory for any flow containing `<choice>` or `<scatter-gather>`):** a flat connector list is not enough — build a **path map** that records which connectors are reachable on each distinct execution path. Rules:

- For every `<choice>` at any depth: record connectors separately per branch — one entry for each `<when>` path and one for `<otherwise>`. Never aggregate across branches.
- For every `<scatter-gather>`: each branch is an independent parallel path. Record its connectors independently. If a branch contains a `<choice>`, apply the choice rule recursively inside that branch.
- Nesting is unlimited: a choice inside a scatter-gather branch that itself contains another choice produces leaf paths at every level. Record each leaf path separately.
- One test will cover exactly one path through the tree. The path map is the direct input to the test plan — one row per path = one test.

---

## K6 — JSON discipline

- **Reuse existing.** Before writing an `in/` / `out/` file, READ it; if valid, keep it and note `reusing existing: <path>`.
- **REST phase `in/` content comes from the RAML example directly.** `$requestExample` (Step 2f) is WRITTEN verbatim as `in/<BASE_NAME>-request.json` — no key derivation, correct by definition. For choice branches, copy the base example and change only the discriminator field **value(s)**.
- **`out/` = raw backend output** = the DWL's `payload` INPUT — not the DWL output or the RAML response example, unless the DWL is a confirmed pass-through (`<ee:set-payload>` body is exactly `payload` or `output … --- payload`), in which case the RAML success example is correct for `out/`.
- **One valid root per file.** READ each `in/` / `out/` back: single root, no `}{` / `][`, no trailing commas, correct root type (object vs array). Fix before assembling.
- **RAML compliance for every `in/` file (branch copies included).** The base `$requestExample` from the RAML is already compliant. Branch-specific copies must stay compliant — three mandatory checks before writing any `in/` file:
  1. **Enum values.** For every field the RAML defines with an `enum:` list, the value used in the JSON must be one of the listed literals. Never invent a value (e.g. `"GENERAL"`, `"Cell"`) not present in the RAML enum. Find the enum by reading the RAML type definition or the data-type file it includes. Tip: open the RAML type file, search for `enum:`, and use one of its entries.
  2. **JSON types.** Do not change a field's JSON type when making a branch copy. If the RAML declares a field as `array`, keep it as an array (e.g. `conditions: [{"condition": "..."}]`, not `conditions: "pso"`). If declared as `object`, keep it as an object.
  3. **Required fields.** All required fields (those without `?` or `required: false` in the RAML) must be present in every `in/` file — including inside nested objects and array items. When adding or replacing a consent/item in an array, include ALL required fields for that item type (e.g. a consent object needs `type`, `submittedDate`, `providedBy`, `status`, `providedChannel` — not just `type`).

  **Compound condition guard (run before the copy step).** Read each `<when>` expression in full — many conditions are compound (`A AND B`, `NOT isEmpty(X) AND Y == "v"`, `A OR B`). List EVERY field and variable the expression tests. For a branch copy to route correctly, ALL tested fields must be set to values that satisfy the condition together. A copy that changes only one field while leaving others at their base-file values will silently route to a different branch at runtime. Example: if the `<when>` tests `notificationType == "email" AND isSavingCardRequested == true`, BOTH fields must be changed in the branch copy — not just `notificationType`. Nested `<choice>` inside a `<when>` branch introduces further compound conditions: recurse and list those fields too.

  **Workflow:** (1) read the full `<when>` expression → list every field it tests → (2) copy the RAML example verbatim → (3) apply ALL listed field changes so the condition evaluates to true → (4) run the three RAML compliance checks above as a HARD GATE → (5) WRITE only when all three checks pass.

---

## K7 — Test properties (copy once, before the first suite)

Exactly **two files** must be copied from `src/main/resources/properties` to `src/test/resources/properties`. Both are mandatory — missing either is a hard stop.

| File | If missing from `src/main` |
|---|---|
| `app-properties-test.yaml` | `STOP: app-properties-test.yaml not found in src/main/resources/properties — create it before running.` |
| `app-secrets-test.yaml` | `STOP: app-secrets-test.yaml not found in src/main/resources/properties — create it before running.` |

1. READ `src/main/resources/properties/app-properties-test.yaml`; WRITE identical content to `src/test/resources/properties/app-properties-test.yaml`. Halt with the message above if the source does not exist.
2. READ `src/main/resources/properties/app-secrets-test.yaml`; WRITE identical content to `src/test/resources/properties/app-secrets-test.yaml`. Halt with the message above if the source does not exist.
3. If `$requiresClientCredentials`, EDIT the copied `app-secrets-test.yaml` to append `munit.client.id` / `munit.client.secret` placeholders.
4. **Never** copy or create any other file in the properties folder (no `*-dev/qa/prod.yaml`, `app-constants.yaml`, `app-errors.yaml`, `apikit-errors.yaml`). Do not invent files.
5. **Verify:** only these two files were added; flag any pre-existing non-test file with `NOTE: non-test property present — <filename>`; never delete files you didn't create.

---

## K8 — Coverage contract

**Hard floor ≥ 80% processor coverage** (MUnit's report). The strategy below typically reaches ≥ 86% on well-structured flows, and 100% on simple single-operation APIs; actual coverage depends on flow shape. Exempt: async error paths and retry-exhaustion inside `<async>`.

**Small-flow 100% target:** when the total count of elements with a `doc:id` across the main flow and all reachable sub-flows is ≤ 25, the coverage target is 100%. In Step 3, enumerate every element with a `doc:id` individually and confirm each is reached by at least one planned test. Add a dedicated test for any unreached element — even if it is only a logger or transform on an otherwise-empty branch. 100% is achievable and expected for simple, single-operation APIs with no unresolvable sub-flows.

---

### Path-coverage algorithm (strict — run before writing any test)

The agent enumerates every execution path first, then maps one test per path. Pattern-matching is not enough — this algorithm applies to every project.

**Step 1 — Enumerate every execution path.**
Traverse the processor tree depth-first from the flow entry. At each branching element, fork:

| Element | How to fork |
|---|---|
| `<choice>` | One path per `<when>` **and** one path for `<otherwise>` — always both, no exceptions |
| `<scatter-gather>` | Each branch is an **independent** parallel path — never cross-multiply branches |
| `<foreach>` / `<parallel-foreach>` / `<batch:step>` | One iteration-representative path; apply choice rule inside the body; mock the iteration collection as a non-empty array so the body executes at least once |
| `<try>` with `<error-handler>` | One success path + one error path per `<on-error-*>` handler |
| `<until-successful>` | One success path (connector mocked to succeed) + one retry-exhausted path (connector mocked with `MULE:RETRY_EXHAUSTED`); add `expectedErrorType="MULE:RETRY_EXHAUSTED"` on the retry-exhausted test unless the flow catches it with `<on-error-continue>` |

Recursion is unlimited: apply these rules inside every branch, at every depth, including a `<choice>` inside a `<scatter-gather>` branch that itself contains another `<choice>`. Every leaf node (a fully-resolved path from entry to exit) becomes one test.

**Step 2 — Rules per element.**

`<choice>` — every `<when>` branch gets its own test; `<otherwise>` always gets its own test even when it invokes no connectors. When `<otherwise>` has no connectors, add `<!-- B16-exception: provably empty otherwise branch -->` inside `<munit:behavior>` — the test still covers the `<choice>` processor itself.

`<scatter-gather>` — generate:
- One **all-branches-happy** test: every branch takes its primary path, every connector in every branch fires.
- One **per-branch-alternative** test for each branch that has a non-trivial internal path (contains a `<choice>` or a connector that can fail): set that branch's condition/variable to its alternative value while all other branches remain on their happy path.
Do not test cross-branch combinations — branches are independent.

`<foreach>` / `<parallel-foreach>` / `<batch:step>` — mock iteration connectors as if executing one iteration. Apply the choice rule to any `<choice>` inside the body.

`<try>` with connector — success path + one error path per `<on-error-*>` handler. Cache (`os:retrieve` gating a branch) — miss test + hit test, never collapsed.

`<raise-error>` (bare in a choice branch or wrapped in `<try>`) — one test per reachable raise path.

**Step 3 — Processor cross-check (mandatory, run before the first WRITE).**
List every **element with a `doc:id`** in the flow and all reachable sub-flows — this includes loggers, transforms, set-variable, set-payload, choice, foreach, scatter-gather, try, async, batch:job, raise-error, and all backend connectors. MUnit measures coverage on every element with a `doc:id`, not only backend connectors. For each element, confirm it appears in the execution path of at least one planned test. If any element has zero coverage, add a test that exercises the path containing it. This drives coverage toward the floor — and toward 100% for small flows.

**Step 4 — Announce and proceed.**
Post `→ [N] execution path(s) identified — [N] test(s) planned` before generating XML. If the cross-check adds tests, post the revised count.

---

**No regression on regenerate — REST:** before writing, SEARCH `src/test/munit/**/*.xml` for `<munit:enable-flow-source value="<PUBLIC_FLOW>">`. If found, that file — regardless of its name — is the existing suite for this endpoint. READ its `<munit:test>` names; the new suite must cover every prior path plus any newly discovered ones. WRITE will overwrite that file in place.

**No regression on regenerate — Scheduler:** before writing, SEARCH `src/test/munit/**/*.xml` for `flow-ref name="<SCHEDULER_FLOW_NAME>"`. If a match is found inside a `<munit:execution>` block, that file — regardless of its name — is the existing suite for this flow. READ its `<munit:test>` names; the new suite must cover every prior path plus any newly discovered ones. WRITE will overwrite that file in place.

Never reduce test count without an equal-or-broader replacement.

---

## K9 — Self-validator (run before declaring done)

After writing all suites, READ each back and confirm — fix any failure with EDIT / re-WRITE and re-check before declaring done:

- **Drill-through check (run first):** for every `<munit:test>` in the suite, check its `<munit:behavior>` for any `<mock-when processor="mule:flow-ref">`. Cross-reference with the Step 4.1 checklist:
  - If the flow-ref name is **RESOLVED** → it should not be a flow-ref mock; it should be the connector mocks from Step 4.2f. If it is still a flow-ref mock, the drill-through was skipped. EDIT the test: replace the flow-ref mock with the backend connector mocks. Re-check.
  - If the flow-ref name is **UNRESOLVED** → flow-ref mock is correct; confirm NOTE comment is present.
  - If the flow-ref name is NOT on the Step 4.1 checklist → unexpected; treat as UNRESOLVED, add NOTE.
- No RULE ZERO pattern from K3 (no `verify-call`; no `happy-path` / `error-path` / `negative-path` / `default-choice` test names; no `assert-that`; no empty `<munit:behavior>`; …).
- Every `<munit:test name>` starts with `<SUITE_BASE>-`.
- *(REST)* Every `<munit:test>` has `<munit:enable-flow-sources>` as its first child with correct flows (B17).
- Exactly one `<munit-tools:assert>` per `<munit:validation>`; exactly four loggers per test; `payload` before `attributes` in every `then-return`.
- REST suites have all six required namespace/schema markers; no BOM before `<?xml`.
- `src/test/resources/properties` holds only `app-properties-test.yaml` and `app-secrets-test.yaml` (K7).
- *(REST)* Every `in/` file matches `$requestExample` from the RAML (K6). Re-check each `in/` / `out/` for the single-root rule.
- **Model-agnostic check:** confirm no generated XML, JSON, or property file contains a model name, vendor name, token-budget limit, or context-size reference. These have no place in test output — remove any found.
- **No memory bleed check:** confirm every `doc:id` `whereValue`, flow name, and connector reference used in the suite can be traced to a line read from a project source file on this run. If any value cannot be traced to a source-file READ, treat it as wrong and re-read the source file to obtain the correct value.
- **UUID validity check (B21):** scan every `doc:id` attribute in the suite. Each must match `[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}` exactly. Any `doc:id` containing a letter g-z is invalid — replace it with a fresh valid UUID before declaring done.
- **Unique doc:id check (B22):** count all `doc:id` values in the file; the number of distinct values must equal the total count. Any duplicate must be given a fresh unique UUID. Run this check after fixing B21 violations so the replacements are also unique.
- *(REST)* **Flow-source traceability check:** every `<munit:enable-flow-source value="…">` value must match a flow name read from `api.xml` during Step 0b, or from `error-handlers.xml` read this run. Any value that cannot be traced to a source-file READ this run is wrong — re-read `api.xml` and correct it.

Per suite, post `✅ Done: <filename> (<N> tests)`. Never halt for a fixable issue.

---

---

# Unified Router — Phase 0 through Phase 3

## Acknowledgment
```
UNIFIED: /munit-generate invoked. Detecting flow type and executing full generation...
```

## Phase 0 — Detect flow type
SEARCH for REST triggers and Scheduler triggers. Set ROUTE. Build K5 flow map once.

## Phase 1 — REST phase (if ROUTE = REST or BOTH)

> ⛔ **RAML is mandatory for the REST route — global gate.** Before generating or syncing any REST operation, obtain the RAML (REST Step 2a–2c). If it cannot be obtained, **halt the entire command** with the Step 2 message — do **not** run Phase 2 (Scheduler), Mode B, or Mode C, and write no suites, JSON, or properties. A `SCHEDULER`-only project skips this gate (scheduler needs no RAML); a `BOTH` project with missing RAML stops completely rather than emitting scheduler-only tests.

After Step 0e classifies every operation, apply this routing **per operation before running any generation**:

| Invocation | Operation state | Action |
|---|---|---|
| Named `<file.xml>` | **`SYNC_MODE`** (`$existingSuite` is set) | Run `/munit-sync <$existingSuite>` for this operation — do **not** run Steps 1–8. The sync phases (Phase 0–5 of `munit-sync`) handle RAML refresh, flow re-trace, diff, and surgical patching. |
| Named `<file.xml>` | **`GENERATE_MODE`** (`$existingSuite` is null) | Run full REST delta Steps 1–8 for this operation as normal. |
| `all` | **`GENERATE_MODE`** (always — `SYNC_MODE` never fires) | Run full REST delta Steps 1–8 for **every** operation in the list; overwrite any existing suite in place. |

> **`SYNC_MODE` never fires when the argument is `all`** — every operation always gets a full fresh generation pass. If a suite already exists for an operation, WRITE overwrites it in place (same behavior as K8 "no regression on regenerate"). Mode B and Mode C run once after all Mode A work completes in both cases.

If a named invocation has **all** operations in `SYNC_MODE` → skip Steps 1–8 entirely and jump straight to Mode B and Mode C.
If a named invocation is **mixed** → run sync for each `SYNC_MODE` operation first, then run Steps 1–8 for each `GENERATE_MODE` operation.

Zero user interaction throughout.

## Phase 2 — Scheduler phase (if ROUTE = SCHEDULER or BOTH)
Run full Scheduler delta: Steps 0–6. Run Mode C here — `<flow-ref>`-anchored — **only when ROUTE = SCHEDULER**. When ROUTE = BOTH, Mode C already ran in Phase 1 against the HTTP anchor; do **not** run it again — both variants write the same `error-handler-test-suite.xml`, and a second (weaker, non-HTTP) run would overwrite the HTTP-anchored suite. Zero user interaction.

> **Mode C runs exactly once per invocation.** REST/BOTH → HTTP-anchored in Phase 1. SCHEDULER only → `<flow-ref>`-anchored in Phase 2.

## Phase 3 — Completion summary
```
Summary: Route <REST|SCHEDULER|BOTH> | <N> suite(s) | <T> tests | Status: PASS
```

---

# REST / APIKit — specific steps

> ⛔ **Pre-condition guard.** This command handles flows with `<http:listener>` + `<apikit:router>`. If the target file has `<scheduler>`, `<anypoint-mq:subscriber>`, `<jms:listener>`, or `<vm:listener>` but NO `<http:listener>`, post `WRONG PROMPT: use /munit-generate-scheduler for this file` and halt.

## Acknowledgment
When invoked directly, post exactly `Generating MUnit tests...` as the first output. When invoked via `/munit-generate`, the acknowledgment already showed — skip it. Then post `→` progress lines per the Progress updates section as you go.

## Usage & modes
```
/munit-generate <operation-file.xml> [<file2.xml> …]   # Mode A only
/munit-generate all                                    # Mode A (all ops) + Mode B + Mode C
/munit-generate errors                                 # Mode C only
```
`$r33BatchMode` = true when Mode A has more than one entry (enables cross-operation sub-flow sharing, R33).

## Discovery table (REST tokens — fill every cell by READ this run)
`<BASE_NAME>` operation filename without `.xml` · `<SUITE_BASE>` = `<BASE_NAME>-test-suite` · `<MAIN_LISTENER_FLOW>` · `<PUBLIC_FLOW>` · `<METHOD>` / `<RESOURCE_PATH>` from Step 2d · `<CONFIG_REF>` from test-config · `<CONNECTOR_DOC_NAME>` / `<CONNECTOR_DOC_ID>` (verbatim) · `<ERROR_TYPE>` from `<error-mapping targetType>` · `<AUX_MOCK_IDS>` every `os:*` + `raise-error` by `doc:id` · `<REQUIRED_HEADERS>` from Step 2e (RAML traits, read directly) · `$requestExample` per endpoint from Step 2f · `<UUID>` fresh per element.

---

## Step 0 — Pre-checks
Post `→ Reading project config…`.

- **0a.** READ `src/test/resources/test-config.xml` → `<CONFIG_REF>` = name of `<http:request-config>`. If unreadable: `STOP: test-config.xml missing — create it before running.` and halt.
- **0b.** READ `src/main/mule/api.xml` → `<MAIN_LISTENER_FLOW>`; `$apiKitFlows` = flows named `method:\path:…:apiKitConfig`; `$apiOperations` = the `name` of the `<flow-ref>` inside each.
- **0c.** Build the flow map (K5): SEARCH `src/main/mule/**/*.xml` for `<flow name="` and `<sub-flow name="`, recording every `name → file`. Keep for Steps 1 and 4; note any name in two files (ambiguous — mock by `doc:id`).
- **0d.** Build the Mode A list from `$apiOperations`: `all` → every operation, `$modeErrors = $modeB = true`; `errors` → `[]`, `$modeErrors = true`; named → only names matching `$apiOperations` (with/without `.xml`); unmatched names are skipped with a warning.
- **0e.** *(Named invocations only — skip this check entirely when the argument is `all`.)* When the argument is `all`, mark every operation **`GENERATE_MODE`** immediately and proceed to Step 0.5; `SYNC_MODE` never fires for `all`. For a named `<file.xml>` argument: SEARCH `src/test/munit/**/*.xml` for `<munit:enable-flow-source value="<PUBLIC_FLOW>">`. If a file matches, it is the existing suite for that endpoint — record its path as `$existingSuite` and mark this operation **`SYNC_MODE`**. If no match, `$existingSuite` is null, the suite will be created at `src/test/munit/<BASE_NAME>-test-suite.xml`, and the operation is marked **`GENERATE_MODE`**.
- Post `→ Found [N] flow(s) to test: [list]`. Proceed once test-config is confirmed.

## Step 0.5 — Output directories
WRITE creates any missing parent folders automatically — no explicit creation needed. Suites and JSON land under `src/test/munit`, `src/test/resources/in`, `.../out`, and `.../properties`.

## Batch discipline (`all` mode)
Run Step 2 once — unzip the RAML artifact and READ the root RAML before the loop; reuse the resolved endpoint map for all operations. Run Steps 1 and 3–8 per operation; log per-file failures and continue. Carry forward only: `$coveredSubflows`, resolved RAML endpoint map, `$requiresClientCredentials`, `$requiresJWT`, `<CONFIG_REF>`, `<MAIN_LISTENER_FLOW>`.

## Step 1 — Analyse the operation XML
Resolve the owning file of the operation sub-flow via the flow map (K5). `<BASE_NAME>` = that file's name without `.xml`. READ the full XML. Record:
- Every `<flow-ref>` target (name, doc:id, which element it sits inside)
- Every `<ee:transform>` resource/inline expression
- Every `<choice>` with its `<when>` expressions and discriminator fields
- Every `<ee:set-variable>` (name and expression)
- Direct backend connectors with doc:name and doc:id
- Every `os:*` and `<raise-error>` with `doc:id`
- `<error-mapping>` / `<on-error-*>`

**Explicitly flag every `<flow-ref>` that sits inside a `<choice>` branch, `<scatter-gather>` branch, or any nested element:**

```
Choice-branch flow-refs requiring drill-through in Step 4:
  [choice-when-label]: flow-ref name="[SUB_FLOW_NAME]"
  [choice-when-label]: flow-ref name="[SUB_FLOW_NAME]"
  ...
```

Post this flagged list after Step 1. Each entry in this list **must** be drilled through in Step 4 — this is the checklist Step 4 works from. Post the planned test list (apply B5/B6/B7 naming).

## Step 2 — Resolve the API contract (RAML)
Post `→ Extracting the API contract from .m2…`.

- **2a. Parse `pom.xml`.** READ `pom.xml`. Find the `<dependency>` block containing `<classifier>raml</classifier>`. Extract `groupId`, `artifactId`, and `version`. Resolve any `${property}` placeholder in `version`: scan the `<properties>` block for the matching key; `${project.version}` / `${pom.version}` resolves from the root `<version>` element. If no raml classifier dependency exists:
  ```
  ⛔ No RAML dependency in pom.xml — add a <classifier>raml</classifier> dependency and re-run.
  ```
  Halt the entire run — generate no suites, JSON files, or properties.

- **2b. Locate the zip in `.m2`.** Construct the zip path — `<GROUP_PATH>` = `groupId` with every `.` replaced by `/`:
  - Mac/Linux: `~/.m2/repository/<GROUP_PATH>/<artifactId>/<version>/<artifactId>-<version>-raml.zip`
  - Windows: `%USERPROFILE%\.m2\repository\<GROUP_PATH>\<artifactId>\<version>\<artifactId>-<version>-raml.zip`

  If `-raml.zip` is not found, try `-raml-fragment.zip` at the same path. If neither exists:
  ```
  ⛔ RAML artifact not found at <constructed-path> — run `mvn dependency:resolve` to pull it into .m2, then re-run.
  ```
  Do **not** generate any suites, JSON files, or properties.

- **2c. Extract to OS temp.** RUN the native extraction — no scripts, nothing written into the project:
  - Mac/Linux: `unzip -o "<zip_path>" -d "/tmp/munit-raml/<artifactId>"` (`unzip -d` creates the target directory automatically).
  - Windows 10+ (run in **cmd.exe**): `md "%TEMP%\munit-raml\<artifactId>" 2>nul & tar -xf "<zip_path>" -C "%TEMP%\munit-raml\<artifactId>"` — the leading `md` is required because `tar -C` does **not** create its destination; `2>nul` swallows the harmless "already exists" message on re-runs.

  `<RAML_TEMP_DIR>` = `/tmp/munit-raml/<artifactId>` or `%TEMP%\munit-raml\<artifactId>`. If the RUN shell is PowerShell rather than cmd.exe, create the folder with `New-Item -ItemType Directory -Force -Path "$env:TEMP\munit-raml\<artifactId>"` before the same `tar -xf … -C …`.

- **2d. READ the root RAML.** SEARCH `<RAML_TEMP_DIR>` for `*.raml`; the root is the one containing a `title:` line. READ it in full. Per endpoint, record:
  - Resource path + HTTP method → confirms `<METHOD>` and `<RESOURCE_PATH>` (cross-check with `<PUBLIC_FLOW>` name: split on `:` → `parts[0]` upper-cased = method, `parts[1]` with `\`→`/` and `(name)`→`{name}` = path)
  - `is: [<trait-name>, …]` → note each trait name (resolved in 2e)
  - `body: application/json: example: !include <path>` → note the path (resolved in 2f)

- **2e. READ traits → required headers.** In the root RAML, find the `traits:` block. For each trait name referenced by an endpoint's `is: []`, find its entry. If defined as `!include <traits-file>`, READ that file from `<RAML_TEMP_DIR>`. Extract every `headers:` entry and its `example:` value verbatim — use those values directly in `$requiredHeaders`. Add `Content-Type: application/json` for POST / PUT / PATCH. If any header name contains `client` (case-insensitive), set `$requiresClientCredentials` (note `→ Auth detected: client credentials`). If any header is `Authorization`, set `$requiresJWT` (note `→ Auth detected: JWT`). Do not invent headers not present in the RAML.

- **2f. READ request body example → `$requestExample`.** For each endpoint's `body: application/json: example: !include <path>`, READ that JSON file from `<RAML_TEMP_DIR>`. This is `$requestExample` — written verbatim as `in/<BASE_NAME>-request.json` in Step 5a. If `example:` is inline (no `!include`), read it directly from the RAML text. GET / DELETE with no body → `$requestExample = {}`.

## Step 3 — DWL analysis
Post `→ Analysing [BASE_NAME]…`.

READ every `<ee:transform>` in the operation and all reachable sub-flows. Scan all three forms: `resource="mappings/…"` (READ the `.dwl` in full), inline `<ee:set-payload>`, and inline `<ee:set-variable variableName="X">` (extract expression AND note variable name `X`). Inline set-variable is the most-skipped — scan every one in every sub-flow.

Pass-through check per connector: body exactly `payload` or `output … --- payload` → use the RAML success example for `out/`; mark PASS-THROUGH.

For each non-pass-through DWL, map `payload.*` access → required JSON:

| DWL access | Required `out/` JSON |
|---|---|
| `payload.X` | `{ "X": <val> }` |
| `payload.X.Y` | `{ "X": { "Y": <val> } }` |
| `payload[N].X` / `payload filter …` / `payload map …` | root is an array |
| `payload.X[N].Y` | `{ "X": [ { "Y": <val> } ] }` |
| `sizeOf(payload.X …)` | X = non-null array/string |
| `payload.X default ""` | include X with a non-null value |

For every `vars.<n>.*`, trace to the producing `<flow-ref target="<n>">` (R28).

Choice alignment:

| `<when>` pattern | success (condition FALSE) | branch (condition TRUE) |
|---|---|---|
| `isEmpty(payload.X)` | X present, non-empty | omit X / null / `[]` |
| `payload.X == null` | X non-null | omit X |
| `payload.X == "v"` | X = a different valid string | X = `"v"` |
| `payload.X == false` | X = true | X = false |
| `A or B` | negate both | satisfy either |

Work out the field map and choice-routing map internally (not on screen), then post one plain line: `→ Mapped [N] backend call(s) and [M] routing branch(es); [K] test-data file(s) planned.`

## Step 4 — Connector inventory

**MANDATORY: complete every sub-step below before generating any test XML. Each sub-step requires explicit tool calls. Do not approximate, skip, or defer any sub-step.**

---

### 4.1 — Enumerate the full per-branch checklist

Build a checklist with **one row per branch** for every `<choice>` at any depth in any reachable flow. The unit is a branch, not a flow-ref — a 4-branch `<choice>` always produces exactly 4 rows in this checklist.

Rules for constructing the checklist:
- List rows grouped by `<choice>` with a header that states the choice ID, which flow it lives in, and the total branch count.
- Each `<when>` branch gets its own row with its condition expression.
- `<otherwise>` always gets its own row — even when it invokes no connectors.
- When two different `<when>` branches of the same `<choice>` call the same sub-flow, they must still appear as two separate rows, because they represent distinct execution paths and will become two distinct tests.
- For branches that have NO `<flow-ref>`, write `→  no-flow-ref (direct connectors, or B16-exception if empty)`.

**Required format:**

```
Choice-branch drill-through checklist:

[C1 — create-notification-subflow — 3 branches — all must appear]
[ ] C1-when1 (notificationType == "phone")  →  flow-ref: "send-sms-subflow"
[ ] C1-when2 (notificationType == "email")  →  flow-ref: "process-email-subflow"
[ ] C1-otherwise                             →  no-flow-ref (direct loggers only — B16-exception)

[C2 — SG branch 1 — 2 branches]
[ ] C2-when (not isEmpty vars.phone)        →  flow-ref: "mms-process-subflow"
[ ] C2-otherwise                             →  no-flow-ref (empty — B16-exception)
```

**GATE — mandatory before proceeding to 4.2:**
For each `<choice>` group in the checklist above, verify:

```
(number of checklist rows for this choice) == (count of <when> branches) + 1
```

If any group fails the count, the checklist is incomplete. Add the missing rows immediately. Do NOT proceed to 4.2 until every group passes. Post: `→ GATE PASSED: [N] choices, [M] total branches listed.`

If the checklist is empty (no `<choice>` or `<scatter-gather>` at any depth across all reachable flows), post `→ No choice branches found — skipping to 4.3.`

---

### 4.2 — Drill through each item in the checklist (execute in order, one by one)

For **each unchecked item** in the 4.1 list, execute ALL of the following sub-steps before moving to the next item:

**4.2a — SEARCH for the sub-flow.**
Run SEARCH on `src/main/mule/**/*.xml` for both:
- `<sub-flow name="[EXACT_NAME]"`
- `<flow name="[EXACT_NAME]"`

Post the result. If found → record the file path and continue to 4.2b. If not found → mark item UNRESOLVED, add `NOTE: [NAME] not found after SEARCH — flow-ref mock retained`, check the box, skip to the next item.

**4.2b — READ the file.**
READ the file found in 4.2a in full. Post `→ Reading [NAME] from [file-path]`.

**4.2c — List every processor inside the sub-flow.**
Go through the sub-flow body line by line. For each processor, record:

```
Processors in [SUB_FLOW_NAME]:
  1. logger          doc:name="..."       doc:id="..."
  2. ee:transform    doc:name="..."       doc:id="..."
  3. http:request    doc:name="..."       doc:id="..."   ← BACKEND CONNECTOR
  4. ...
```

**4.2d — Identify backend connectors.**
From the list in 4.2c, mark every backend connector (http:request, db:select/insert/update/delete/stored-procedure, salesforce:*, wsc:consume, sftp:read/write, ftp:read/write, file:read/write, os:store/retrieve/contains/remove, anypoint-mq:publish/consume, jms:publish/consume, vm:publish/consume, custom connectors). These are what get mocked. Loggers, transforms, set-variables, set-payload are NOT mocked.

**4.2e — Recurse if needed.**
If any processor in 4.2c is itself a `<flow-ref>`, add it to the checklist and process it by repeating 4.2a–4.2e before continuing.

**4.2f — Mark RESOLVED and record.**
Add the backend connectors found in 4.2d to the path-tagged inventory:

```
✓ Route 1 (when: SMS)  →  RESOLVED
    mock: http:request  doc:name="Send SMS via Twilio"  doc:id="abc-123"
    out-file: out/[BASE_NAME]-route1-response.json
```

Check the item off the 4.1 list.

---

### 4.3 — Complete the full inventory

Combine the choice-branch connectors from 4.2 with all direct connectors in the main flow. Tag each connector with its execution path. Post:

```
→ Full connector inventory:
  Path: primary-happy-path      | connectors: [list]
  Path: choice-route-1          | RESOLVED | connectors: [list]
  Path: choice-route-2          | RESOLVED | connectors: [list]
  Path: sg-branch1-when         | RESOLVED | connectors: [list]
  Path: sg-branch1-otherwise    | (no connectors)
  Path: choice-default          | (no backend connectors — B16-exception)
  Path: choice-route-X          | UNRESOLVED | flow-ref mock: [name]
→ Total: [N] paths, [M] connectors to mock, [K] unresolved

HARD GATE: every item on the 4.1 checklist is now checked (RESOLVED or UNRESOLVED).
Do not proceed to Step 5 until this gate passes.
```

Auto-correct any `whereValue` not matching a listed `doc:id`.
## Step 5 — Write JSON + properties
Post `→ Writing test data…`.

- **5a — `in/` files.** Reuse if present (K6). Else WRITE `$requestExample` (Step 2f) verbatim to `src/test/resources/in/<BASE_NAME>-request.json`. For each choice branch: (a) read the `<when>` expression in full and list every field it tests — compound conditions require ALL fields changed, not just the outermost one; (b) copy the RAML example verbatim; (c) apply ALL identified field changes so the condition routes to this branch; (d) run all three K6 RAML compliance checks (enum, type, required-fields) as a HARD GATE; (e) WRITE `…/in/<BASE_NAME>-<branch-slug>-request.json` only when checks pass.
- **5b — `out/` files.** Raw backend output (DWL input), never DWL output/RAML example unless PASS-THROUGH. Confirm: consuming DWL identified; full `payload.*` map; root type + intermediates + non-null arrays correct; no DWL-output fields present. Validate (K6).
- **Properties.** K7: READ/WRITE each `*-test.yaml`; never READ secret contents.

## Step 6 — Test-config discovery
Confirm `<CONFIG_REF>` from test-config (READ in Step 0). Never create or modify test-config. For every `<flow-ref>` recursively capture processor/doc:name/doc:id/`<error-mapping targetType>`; capture `os:*` / `raise-error` by `doc:id`. Classify each `<try>` + `<error-handler>`: wrapping a `<raise-error>` → R21; wrapping a backend connector → R35 (skip if `<try>` is inside `<async>`). Classify `<choice>` branches with a bare `<raise-error>` → R38.

## Steps 7–8 — Assemble, write, validate
Post `→ Writing [filename]…` before each WRITE.

**Pre-write scan** (auto-fix each violation and re-scan until zero remain):

- **Drill-through gate (must pass before WRITE):** for every `<munit:test>` that exercises a choice branch containing a flow-ref from the Step 4.1 checklist — confirm that sub-flow's backend connectors appear as `<mock-when>` blocks in its `<munit:behavior>`. If any test's behavior has `<mock-when processor="mule:flow-ref">` for a RESOLVED sub-flow, the Step 4.2 drill-through was skipped. **Stop.** Do not WRITE. Go back to Step 4.2 for that sub-flow, complete 4.2a–4.2f, then rebuild that test's behavior. Only proceed to WRITE once zero RESOLVED flow-refs remain as mock-when targets.
- No `verify-call`; no `if`/ternary inside a mock `payload` value (R27 — split into separate mocks); no `"#[(output"`.
- UNRESOLVED flow-ref mocks are allowed — keep them, confirm NOTE comment is present.
- No `json-logger` mock; no `mock-when` in validation or execution.
- Every connector in the Step 4.3 inventory has a matching mock in tests that take that path; `payload` before `attributes` in every `then-return`; `<munit:behavior>` never empty (B16).
- Every `<munit:test>` has `<munit:enable-flow-sources>` as its **first child** with the correct flows per K4 (B17).
- Every `<munit:test name>` starts with `<SUITE_BASE>-` (K4 test naming).

Assemble ALL tests for the source XML into ONE `<mule>` document (B8) and WRITE once. Then run K9 validate; post `✅ Done: <file> (<N> tests)`.

### Test types (Mode A)
Build every applicable type using K4 blocks. **In every name below, `…` stands for `<SUITE_BASE>`** — e.g. `create-enrollment-test-suite-<desc>`. All tests via HTTP must have both `<MAIN_LISTENER_FLOW>` and `<PUBLIC_FLOW>` in `<munit:enable-flow-sources>` (K4 canonical REST test structure).

These types implement the K8 path-coverage algorithm. Types 3–4 and 10 are the most structurally critical — they cover every branch at every depth.

1. **Success** `…-<connector-desc>-success` — all connectors on the primary happy path mocked (R11) + AUX; 4 loggers; one assert.
2. **Connector error** `…-<connector-desc>-<error-slug>` — target connector `then-return` → `<munit-tools:error typeId="<ERROR_TYPE>"/>`; all connectors before it still succeed; add `<http:response-validator … values="200..599"/>` as last child of `<http:request>` (R16).
3. **Choice `<when>` path** `…-<branch-desc>` — one test per `<when>` branch at every nesting level. Set the discriminator variable/expression so exactly this `<when>` condition is true and all outer `<when>` conditions needed to reach it are also true. Mock only connectors on this path. Applies recursively: a `<when>` branch that contains a nested `<choice>` must itself be fully expanded by this rule — one test per leaf path, not one test per outer `<when>`.
4. **Choice `<otherwise>` path** `…-<otherwise-desc>` — one test per `<otherwise>` branch at every nesting level. Set all `<when>` conditions false so this default branch runs. Behavior mocks all upstream connectors that ran before the choice point. When the `<otherwise>` branch genuinely invokes no external connectors, add `<!-- B16-exception: provably empty otherwise branch -->` inside `<munit:behavior>` — the test still covers the `<choice>` processor. Applies recursively: an `<otherwise>` branch that contains a nested `<choice>` must be further expanded.
5. **Raise-error** (R21 `<try>`-wrapped, or R38 bare in a choice branch) — if `<on-error-continue>` wraps it: no `expectedErrorType`, invoke via `<flow-ref>` in execution, enable `<PUBLIC_FLOW>` only. If no outer handler: add `expectedErrorType="<RAISE_ERROR_TYPE>"` on `<munit:test>`.
6. **Try-catch handler** (R35) — connector inside a `<try>` (not inside `<async>`) mocked to throw `HTTP:CONNECTIVITY`; `<until-successful>` → `MULE:RETRY_EXHAUSTED`. `<on-error-continue>` → no `expectedErrorType`; no outer handler → `expectedErrorType` on test.
7. **Batch step error** (R37) — pre-batch connectors success; primary batch connector throws; aggregator-step connectors success.
8. **Foreach / parallel-foreach branch** (R36) — populate iteration collection; drill through `<flow-ref>` in body (R11); apply choice rule (types 3–4) to any `<choice>` inside the body.
9. **Cache miss / cache hit** (R39) — when `os:retrieve` gates a branch, generate BOTH: miss and hit. Never collapse to one.
10. **Scatter-Gather paths** — applies whenever a `<scatter-gather>` is present:
    - **All-branches-happy** `…-<sg-slug>-success`: every branch takes its primary path; every connector across all branches fires. One mock per connector.
    - **Per-branch-alternative** `…-<sg-slug>-<branch-slug>-<alternative-desc>` — one test per branch that has a non-trivial internal path. In this test: set that branch's condition/variable to its alternative value (so its alternative path runs); all other branches still take their happy path and their connectors still fire. One test per branch with alternatives — not one test for all branches combined. Apply choice rule (types 3–4) inside the branch recursively if its alternative path itself contains a `<choice>`.
    - **Mock strategy**: in each per-branch-alternative test, the mock list = connectors on the alternative path of the target branch (if any) + connectors on the happy path of every other branch. If the alternative path has no connectors, behavior mocks only the other branches' connectors.

**Async (R34):** add `<munit-tools:sleep time="10" timeUnit="SECONDS" doc:name="Sleep" doc:id="<UUID>"/>` in `<munit:execution>` right after `<http:request>`. Do not assert on async output.

**Async branch coverage (mandatory):** an `<async>` block that calls a sub-flow via `<flow-ref>` is NOT exempt from coverage — apply the full K8 path-coverage algorithm to the sub-flow it invokes exactly as if it were synchronous. If the async sub-flow contains a `<choice>` with N branches, generate N tests, one per branch, each with `<munit-tools:sleep time="10" timeUnit="SECONDS" …/>` after `<http:request>`.

**Discriminator identification (run before writing async tests):** read every `<when>` expression in the async sub-flow's `<choice>` elements. Record every field that gates a branch — these are the discriminators. Discriminators may be:
- A **payload field** (e.g. a flag field in the request body)
- A **variable** set earlier in the flow from a payload field or header
- A **runtime property** (read via `p('…')`)
- An **HTTP request header** mapped to a variable in an upstream transform (e.g. `vars.commonHeaders.sourceSystem` ← `attributes.headers.'X-...'`)
- Any **combination** of the above (e.g. field-A == true AND header-B == "value")

**Per-branch test:** for each branch, create one `in/` request JSON (or reuse an existing one if it already routes to that branch) with the discriminator field(s) set to the value(s) that make exactly that `<when>` condition true. When the discriminator is an HTTP request header, set that header's value in the `<http:headers>` block of the `<http:request>` in `<munit:execution>`. Mock every backend connector reachable on that branch's path (including all connectors in nested sub-flows it calls) with `statusCode: 200` in `<munit-tools:attributes>` where downstream code checks `attributes.*`.

**Common coverage gap:** when all planned tests use request data whose discriminator values route every execution to the same log-only `<otherwise>` branch, the entire connector chains behind the other `<when>` branches are never invoked and remain at 0 % coverage. Detect this during Step 4 path enumeration: if the same `in/` file is reused for an async test that covers a different `<when>` branch, verify that its discriminator fields actually satisfy that branch's condition — if not, create a separate `in/` file that does.

---

## Mode B — APIKit error suite → `api-test-suite.xml`
`<SUITE_BASE>` = `api-test-suite`. One test per `<on-error-propagate type="…">` in the APIKit error handler (including `ANY`).

Each test: `<munit:enable-flow-sources>` with `<MAIN_LISTENER_FLOW>` **only** (not `<PUBLIC_FLOW>`); mock `apikit:router` by `config-ref` to `then-return` the error type; one `set-event` from any operation's `in/` request; a real `<http:request>` + `200..599` validator; 4 loggers; one assert. `ANY` → mock with `HTTP:CONNECTIVITY`. Types are read from the file, never hardcoded. `expectedErrorType` does not appear on any `<munit:test>`.

## Mode C — Error-handler suite → `error-handler-test-suite.xml`
`<SUITE_BASE>` = `error-handler-test-suite`.

- **MC1.** Find `error-handlers.xml` (`src/main/mule/common/` then `.../commons/`). Collect all `<error-handler>` elements; discard any whose `name` contains `apikit` (case-insensitive). For every `<on-error-continue>` / `<on-error-propagate>` in kept handlers, record `type` and `slug = type.replace(':','-').lower()`. Find `<MAIN_LISTENER_FLOW>` (has `<http:listener>` and references a kept handler). READ test-config for `<CONFIG_REF>`.
- **MC2.** Reuse existing Mode A `out/` files for pre-trigger success mocks — do NOT write new `out/` files for error responses. This (APIKit-anchored) Mode C never synthesises new `out/` data; the error type lives only in the mock's `<munit-tools:error typeId="…"/>`.
- **MC3.** All tests in ONE document. Each test: `<munit:enable-flow-sources>` with **both** `<MAIN_LISTENER_FLOW>` and `<PUBLIC_FLOW>`; behavior has pre-trigger success mocks + the error mock on the trigger; execution loads any `in/` request + `200..599` validator; 4 loggers; one assert. **`expectedErrorType` MUST NOT appear on any `<munit:test>`** — the error type lives only in `<munit-tools:error typeId="…"/>`. Scan and remove any `expectedErrorType`. After writing, READ back and confirm all six required namespace/schema markers; fix any missing. Post `✅ All done — test suites written.`

---

## Reference — where files live

Operations `src/main/mule/operations/` · router + public flows `src/main/mule/` · common sub-flows / error handlers `src/main/mule/common/` (or `.../commons/`) · DWL `src/main/resources/mappings/` · properties `src/main/resources/properties/*.yaml` · test HTTP config `src/test/resources/test-config.xml` · test data `src/test/resources/in|out/` · suites `src/test/munit/`.

---

---

# Scheduler / MQ / JMS / VM — specific steps

> ⛔ **Pre-condition guard.** This command handles flows triggered by `<scheduler>`, `<anypoint-mq:subscriber>`, `<jms:listener>`, or `<vm:listener>`. If the target file has `<http:listener>` + `<apikit:router>` but none of these triggers, post `WRONG PROMPT: use /munit-generate-api for this file` and halt.

Scheduler flows have no HTTP listener and no inbound payload — execution invokes the flow directly via `<flow-ref>`. There is no `in/` folder. Output: `src/test/munit/<BASE_NAME>-test-suite.xml`.

## Acknowledgment
When invoked directly, post exactly `Generating MUnit tests...` as the first output. When invoked via `/munit-generate`, the acknowledgment already showed — skip it. Then post `→` progress lines per the Progress updates section as you go.

## Usage
```
/munit-generate-scheduler <scheduler-file.xml> [<file2.xml> …]
/munit-generate-scheduler all      # every scheduler XML → its own suite, then Mode C
/munit-generate-scheduler errors   # Mode C only
```

## Discovery table (scheduler tokens — fill every cell by READ this run)
`<SCHEDULER_FLOW_NAME>` name of the flow containing the trigger · `<BASE_NAME>` scheduler filename without `.xml` · `<SUITE_BASE>` = `<BASE_NAME>-test-suite` · `<BACKEND_CONNECTOR>` every external-I/O element in the full path (processor, doc:name, doc:id) · `<HANDLER_TYPE>` the `type=` of the `<on-error-continue>` / `<on-error-propagate>` wrapping a connector — read it directly, never derive from `<error-mapping>` · `<ERROR_TYPE>` `<error-mapping targetType>` or `HTTP:CONNECTIVITY` (batch-step errors only) · `<OS_MOCK_IDS>` every `os:*` by `doc:id` (+ `target` for `os:retrieve`) · `<ATTRIBUTES_REQUIRED>` connectors whose downstream DWL reads `attributes.*` · `<BATCH_STEPS>` each `<batch:step>` name + `acceptPolicy` · `<CHOICE_DISCRIMINATORS>` · `<UUID>` fresh per element.

Fixed conventions: logger `message="#[payload]"`; categories `${log.category.base}.<BASE_NAME>.{execution|validation}.{start|end}`; the invocation flow-ref `doc:name` MUST be `"Ref <SCHEDULER_FLOW_NAME>"`.

---

## Step 0 — Validate inputs
Post `→ Reading project config…`. If the argument is `errors` → go straight to Mode C.

Identify the scheduler files: SEARCH `src/main/mule/**/*.xml` for `<scheduler`, `<anypoint-mq:subscriber`, `<jms:listener`, and `<vm:listener`; the files containing one are the candidate set. Build the flow map (K5) with a SEARCH for `<flow name="` / `<sub-flow name="` to drive Step 2 lookup. Build targets: `all` → every scheduler file; named → only names in the candidate set (append `.xml` if missing); invalid names are skipped.

Post `→ Found [N] scheduler flow(s) to test: [list]`. No valid files → STOP.

## Step 0.5 — Directories + properties
No explicit directory creation — WRITE creates missing parents. Scheduler suites use `src/test/munit`, `src/test/resources/out`, and `.../properties` (no `in/`). Run the K7 property copy once before any suite.

## Batch loop
Run Steps 1–6 + K9 once per valid file; project-wide files (common-flows, error-handlers) are READ once and reused; per-file failure is logged and the loop continues. When the argument was `all`, run Mode C after all scheduler suites complete.

## Step 1 — Analyse the scheduler XML
Post `→ Analysing [BASE_NAME]…`.

READ the file in full. Record `<SCHEDULER_FLOW_NAME>`; direct `<flow-ref>` targets; direct backend connectors; direct `os:*` (doc:id + `target`); direct `<choice>` `<when>` expressions + discriminators; the flow-level error-handler kind.

After recording `<SCHEDULER_FLOW_NAME>`, SEARCH `src/test/munit/**/*.xml` for `flow-ref name="<SCHEDULER_FLOW_NAME>"`. If a match appears inside a `<munit:execution>` block, that file — regardless of its name — is the existing suite for this flow; record its path as `$existingSuite` and WRITE will overwrite it in place. If no match, `$existingSuite` is null and the suite will be created at `src/test/munit/<BASE_NAME>-test-suite.xml`.

**Explicitly flag every `<flow-ref>` that sits inside a `<choice>` branch or `<scatter-gather>` branch:**

```
Choice-branch flow-refs requiring drill-through in Step 2:
  [choice-when-label]: flow-ref name="[SUB_FLOW_NAME]"
  [scatter-gather-branch > choice-when]: flow-ref name="[SUB_FLOW_NAME]"
```

Post this flagged list. Each entry must be drilled through in Step 2 using sub-steps 4.2a–4.2f before any test XML is generated.

## Step 2 — Recursive sub-flow trace
For each `<flow-ref>` (and recursively), resolve the owning file via K5. In each, record: every backend connector (processor/doc:name/doc:id verbatim); every `os:*`; every `<batch:job>` — each `<batch:step>` name + `acceptPolicy`, the primary connector per process step, and all aggregator-step connectors; every `<foreach>` / `<parallel-foreach>` and its inner connectors; every `<try>` + `<error-handler>` — the connector inside, the handler kind, the exact `type=` as `<HANDLER_TYPE>`, and any `<raise-error>`; every nested `<choice>`. Connector order: pre-batch → batch process-step → aggregator-step → on-complete.

**MANDATORY drill-through for every choice-branch flow-ref. Use sub-steps 4.2a–4.2f from the choice-branch flagged list (Step 1):**

**4.2a** SEARCH `src/main/mule/**/*.xml` for `<sub-flow name="[NAME]"` and `<flow name="[NAME]"`.
**4.2b** If found: READ the file. Post `→ Reading [NAME] from [file]`.
**4.2c** List every processor. Identify backend connectors (http:request, db:*, etc.) — these get mocked.
**4.2d** Recurse into any further `<flow-ref>` inside the sub-flow.
**4.2e** Mark RESOLVED — add connector mocks to that branch's path. The test mocks these connectors, NOT the flow-ref.
If 4.2a finds nothing: mark UNRESOLVED — flow-ref mock retained as fallback; add NOTE.

**HARD GATE:** every item on the Step 1 flagged list must be checked (RESOLVED or UNRESOLVED) before proceeding to Step 3. Post the full inventory with RESOLVED/UNRESOLVED status for each path.

## Step 3 — DWL mapping
For each backend connector and each `os:retrieve`, READ the consuming DWL (inline CDATA or `resource="mappings/…"`) and synthesise the minimum valid `out/` JSON from the actual `payload.*` / `vars.*.*` accesses (string → `"test-value"`, int → `1`, bool → `true`, UUID → `"test-uuid-1234"`, array → `[{…}]`). If a downstream `<choice>` tests `isEmpty(payload)` / `sizeOf(payload)==0`, the success mock returns non-empty and you add a separate empty-result branch test.

**Attributes scan (mandatory):** in the same and all other consuming DWL / `<choice>` / `<foreach>` expressions, search for `attributes\.`; if found, add the connector to `<ATTRIBUTES_REQUIRED>` and WRITE `out/<BASE_NAME>-<doc-name-kebab>-attributes.json` (e.g. `{ "statusCode": 200 }`). If not, omit `<munit-tools:attributes>` from that mock. WRITE each `out/` file and validate (K6). `<doc-name-kebab>` = `doc:name` lower-cased, spaces→hyphens, specials stripped.

Post `→ Mapped [N] backend call(s) and [M] routing branch(es); [K] test-data file(s) planned.`

## Step 4 — Error / raise-error / batch analysis
`<try>` + `on-error-continue` inside a `<batch:step>` → feeds an `ONLY_FAILURES` / `ALL` aggregator (RS16 trigger). `<try>` + `on-error-propagate` at flow level → propagate variant. `<try>` + `on-error-continue` at flow level → standard variant. Scan ALL sub-flows for `<raise-error type="…">` → one test per distinct reachable path. `<batch:step>` with `acceptPolicy="ONLY_FAILURES"` or `"ALL"` → RS16 test.

## Step 5 — Test set
Post `→ Writing test data…`. Print the full planned list before generating any XML:

| Finding | Test name | `expectedErrorType` on `<munit:test>` |
|---|---|---|
| always | `<SUITE_BASE>-<primary-connector-slug>-success` | — |
| `<choice>` `<when>` with distinct connectors | `<SUITE_BASE>-<branch-desc>-choice` | — |
| `<choice>` `<otherwise>` (always) | `<SUITE_BASE>-<otherwise-desc>` | — |
| `<try>` + `on-error-continue` connector | `<SUITE_BASE>-<connector-slug>-error` | — |
| `<try>` + `on-error-propagate` connector | `<SUITE_BASE>-<connector-slug>-error` | `= <HANDLER_TYPE>` |
| reachable `<raise-error>` | `<SUITE_BASE>-<slug>-raise-error` | `= <NS>:<CODE>` |
| `<batch:step>` `ONLY_FAILURES` / `ALL` | `<SUITE_BASE>-<batch-step-name>-error` | — |
| `<scatter-gather>` — all-branches-happy | `<SUITE_BASE>-<sg-slug>-success` | — |
| `<scatter-gather>` — per-branch-alternative (one per branch with non-trivial internal path) | `<SUITE_BASE>-<sg-slug>-<branch-slug>-<alt-desc>` | — |

Every test uses the Canonical Scheduler Test Structure from K4; the only per-test variation is whether `expectedErrorType` is present.

## Step 6 — Generate the suite
Post `→ Writing [filename]…` before each WRITE.

**Pre-write scan** (auto-fix each violation and re-scan until zero remain):
- **Drill-through gate:** for every test covering a choice branch with a flow-ref: confirm the sub-flow's backend connectors appear as `<mock-when>` blocks in its behavior. If any test has `<mock-when processor="mule:flow-ref">` for a RESOLVED sub-flow, stop — the Step 2 drill-through was skipped. Complete 4.2a–4.2f for that sub-flow, rebuild the test's behavior, then re-scan. Only proceed to WRITE when zero RESOLVED flow-refs appear as mock-when targets. UNRESOLVED flow-ref mocks are permitted — confirm NOTE comment present.
- No `verify-call`; no `<munit:enable-flow-sources>` (B19); no terminal command or script (B18); first byte is `<?xml` (no BOM/leading whitespace).
- `payload` before `attributes` in every `then-return`; every `<ATTRIBUTES_REQUIRED>` connector has its `-attributes.json` on disk; no inline JSON in an XML attribute without `readUrl` (B20).
- Error-type strictness: every `<munit-tools:error typeId="…"/>` uses the verbatim source handler `type=`; `on-error-continue` → `expectedErrorType` absent; `on-error-propagate` → `expectedErrorType` present with the exact handler `type=` value. A propagate/raise-error test missing `expectedErrorType` is a defect — fix before WRITE.
- Every `<munit:test name>` starts with `<SUITE_BASE>-`; all K3 bans clear.

Assemble ALL tests into ONE `<mule>` document (one `<?xml>`, one `<mule>`, no `<import>` — no HTTP config needed) and WRITE once. Then run K9 validate; post `✅ Done: <file> (<N> tests)`.

### Per-test deviations
- **Success:** all connectors return success `out/` payloads. Name after the primary connector (B5).
- **Connector error — continue:** the failing connector's `then-return` → `<munit-tools:error typeId="<HANDLER_TYPE>"/>`; earlier connectors keep success; no `expectedErrorType`.
- **Connector error — propagate:** same, but the handler is `<on-error-propagate>`; add `expectedErrorType="<HANDLER_TYPE>"` on `<munit:test>` (exact same value as the mock `typeId`).
- **Batch step error (RS16):** execution always includes the sleep. Behavior order: pre-batch → batch-step-pre-primary → primary (error) → aggregator (success) → `os:store` on-complete (empty).
- **Choice branch (RS14):** if the discriminator is a variable, put a `<munit:set-event>` with `<munit:variables>` as the **first element of `<munit:behavior>`** to force the branch value; property discriminators (`p('…')`) need no set-event. Mock only connectors reachable on that branch.
- **Choice otherwise:** behavior mocks all upstream connectors so all `<when>` conditions fail. When the default branch genuinely invokes no connectors, add `<!-- B16-exception: provably empty default branch, no connectors reachable -->` inside `<munit:behavior>`.
- **Raise-error (RS21):** add `expectedErrorType="<NS>:<CODE>"`; behavior mocks pre-raise connectors and sets the variable(s) that force the raise condition.
- **Scatter-Gather all-branches-happy:** set every branch's discriminator variable to the value that makes its primary `<when>` condition true. Mock every connector across every branch. This is the primary success test for the scatter-gather.
- **Scatter-Gather per-branch-alternative:** for the target branch, set its discriminator to the value that makes its primary `<when>` false (its alternative path runs). Set all other branches' discriminators to keep their primary `<when>` true. Mock only the connectors in the branches that still fire on their primary path; the target branch's alternative path may have no connector to mock. If it has none, behavior mocks only the other-branch connectors and adds `<!-- B16-exception: branch alternative invokes no connectors -->`.

**Async branch coverage (mandatory for scheduler async blocks):** an `<async>` block inside a scheduler flow that calls a sub-flow via `<flow-ref>` is NOT exempt from coverage. Apply the full K8 path-coverage algorithm to the sub-flow it invokes exactly as if it were synchronous. If the async sub-flow contains a `<choice>` with N branches, generate N tests — one per branch — each with `<munit-tools:sleep time="10" timeUnit="SECONDS" doc:name="Sleep" doc:id="<UUID>"/>` in `<munit:execution>` immediately after the `<flow-ref>`.

**Discriminator identification (run before writing async tests):** read every `<when>` expression in the async sub-flow's `<choice>` elements. Record every field, variable, or property that gates a branch. Discriminators in scheduler flows are typically variables or runtime properties. For each branch, set the discriminator via `<munit:set-event><munit:variables>` placed as the **first element of `<munit:behavior>`** so the `<when>` condition routes to the intended branch. Mock every backend connector reachable on that branch's path.

**Common coverage gap:** if all planned tests use the same variable values that route every execution to the same `<otherwise>` branch, all connector chains behind the other `<when>` branches remain at 0% coverage. Detect this during Step 3 path enumeration and create branch-specific variable set-events for each distinct `<when>` path.

---

## Mode C — Error-handler suite (non-HTTP anchor) → `error-handler-test-suite.xml`
`<SUITE_BASE>` = `error-handler-test-suite`. Triggered by `errors` when the anchor flow is scheduler / MQ / JMS / VM. Tests invoke the anchor flow via `<flow-ref>` — no HTTP listener.

- **MC1.** READ `error-handlers.xml` (`common/` or `commons/`). Per handler record: `type=` (`<HANDLER_TYPE>`); continue vs propagate; whether the body contains a direct `<raise-error type="…">` (`<RAISE_ERROR_TYPE>`, distinct from `<HANDLER_TYPE>`); `<ee:transform>` variable assignments. READ `app-errors.yaml` to resolve property keys; READ `mappings/errors/exception-response-payload.dwl` for the standard output shape. Detect the anchor flow (a flow whose `<error-handler ref>` points at the handler; if several, pick the fewest-connector one); note whether it has `<batch:job>` (→ add the sleep). Identify the trigger connector (first backend `http:request` in the path) and pre-trigger connectors (success mocks). For `APP:*` types, find the `<flow-ref>` (and its `doc:id`) whose target sub-flow raises that error — mock that flow-ref instead of a connector.
- **MC2.** Synthesise each `out/` JSON: standard path → `{ "error": { "errorCode": "<status>", "title": "<title>", "code": "<code>", "details": "<desc>" } }` (`ANY` → the `default` property group); custom `APP:*` → the inline DWL shape. Validate (K6).
- **MC3.** `<SCHEDULER_FLOW_NAME>` = the anchor flow. All tests in ONE document. Decision table:

| Handler | mock `typeId` | `expectedErrorType` on test |
|---|---|---|
| `on-error-continue` (any) | `<HANDLER_TYPE>` | — |
| `on-error-propagate`, no inner raise | `<HANDLER_TYPE>` | `= <HANDLER_TYPE>` |
| `on-error-propagate`, inner `<raise-error>` | `<HANDLER_TYPE>` | `= <RAISE_ERROR_TYPE>` |

The mock `typeId` always triggers the handler; for a propagate-with-inner-raise test, the test's `expectedErrorType` is the raised type (what ultimately propagates). Post `✅ All done — test suites written.`

---

## Reference — where files live

Scheduler flows `src/main/mule/*.xml` (root) · operation/sub-flows `src/main/mule/operations/` then root + common files · common sub-flows / error handlers `src/main/mule/common/` (or `.../commons/`) · DWL `src/main/resources/mappings/` · properties `src/main/resources/properties/*.yaml` · mock responses `src/test/resources/out/` · suites `src/test/munit/`.