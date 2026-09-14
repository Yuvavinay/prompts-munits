---
mode: 'agent'
agent: 'agent'
name: 'munit-http-listener'
version: '9.0.0'
description: 'Generate and reconcile every HTTP endpoint scenario after recursive flow drilling; validate each planned test and external-call mock.'
argument-hint: 'all | <file.xml> | <file1.xml,file2.xml,...> | errors'
---

# HTTP MUnit: all scenarios for every routed operation

Use `/munit-http-listener all`, a source XML path/list, or `errors`. No argument → ask for scope. A source file can contain many operations. `all` requires one owning suite for every APIKit-routed method/path operation; media-type variants are scenarios within that suite. Resolve different router/listener contexts explicitly. `errors` discovers operation anchors but writes only applicable handler suites.

## Missing dependency policy: generate from available evidence

This version changes the previous all-or-nothing rule for a missing RAML **library/fragment**, when the main API RAML is present. Finish the local dependency search once; then choose the next state without asking the user to choose A/B/C:

| State | Required next action |
| --- | --- |
| Dependency resolved | Use the effective RAML contract and continue normal generation. |
| Main API found; a library remains unavailable | Set affected operation contracts to source-backed, retain the missing references, and continue creating endpoint and error suites from available implementation/test evidence. |
| Main API itself absent or unreadable | Retain the existing missing-root stop; do not invent API routes. |
| Essential fixture/property/trigger facts cannot be established anywhere available | Identify the exact affected cases. Do not make up values, create empty/dummy suites or claim those cases are complete. A missing library alone is not such a case-specific finding. |

Print `→ Main RAML found; <reference> remains unavailable after local search. Continuing source-backed suite generation; RAML completeness will remain unverified.` Do not end with a discovery summary, generic missing-RAML message, continuation question or unrelated non-HTTP options.

For a source-backed contract, use this evidence order: resolved parts of the current RAML; current working tests/fixtures tied to the selected operation; production XML/DWL request consumers and variable producers; existing test-property values/configuration. Trace every header/value/body shape to that evidence. Existing tests are evidence to check against current source, not permission to reuse stale assumptions. Do not invent credentials, client_id/client_secret headers or business fields because they are common patterns.

Keep all known required/request-consumed headers and their evidenced values in the explicit HTTP header map. Missing library declarations mean the full required-header set is unknown; never silently record that as a resolved empty contract. An empty map means no headers were established from the available evidence and remains explicitly unverified. Use existing property expressions for evidenced secret-bearing values rather than printing secrets.

Use the root request example verbatim when available. When its include/type is unavailable, reuse a current operation-matched input fixture or derive a minimal input from actual XML/DWL consumers, following the non-HTTP worker's producer distinction without rerouting the operation. Record unavailable RAML body constraints as unverified. Backend output fixtures still represent raw connector results; recursive traversal, real external mocks and branch predicates remain mandatory.

Every affected contracts[] row has status `source-backed`, nonempty missingReferences, and sourceEvidence entries `{source: <module-relative current file>, contains: <literal non-secret evidence excerpt>}`. Those excerpts must actually occur in current source or existing tests. Normal resolved contracts have status `resolved` and missingReferences []. The gate verifies those facts, not the semantic sufficiency of the evidence. The full provenance ledger must explain each generated value.

Every suite containing a source-backed HTTP test must contain this top-level XML comment, outside all test elements:

```xml
<!-- RAML-CONTRACT: source-backed; required-header completeness is unverified. -->
```

This marks an artifact created with incomplete contract evidence. It is not a disabled test or a fake passing result. Still import test-config.xml; retain Request to <path>, HTTP A/B/C structure, four loggers, the required assertion and handler ownership. Do not mock the router in main-handler tests merely to work around a missing library. A missing dependency may prevent application initialization even when files can be generated; report runtime blocked/unverified rather than claiming the mocks solve that problem.

On a later run when the dependency becomes available, resolve the complete contract, reconcile headers/fixtures/scenarios with narrow edits, rerun the checks and remove the marker only from suites whose affected contracts are now resolved. Preserve existing unrelated tests throughout. Never convert source-backed to resolved merely because XML parses or the non-null assertion passes.

## Execute one endpoint at a time; do not stop at a plan

Freeze the full endpoint/handler inventory first. Keep one compact temp work ledger: `owner suite | required scenarios | ready scenarios | next action | contract status | case-specific gaps`. The unit of execution is then one endpoint's complete plan and candidate suite:

1. Resolve its entire call graph and feasible branches; freeze its source-derived scenario/mock plan before its XML.
2. Build and validate that endpoint's inputs, backend fixtures, request headers, mocks and all tests; save the complete candidate in temp.
3. Update the ledger and move immediately to the next endpoint. Reuse definition reads but preserve caller-specific coverage. Do not reread every other endpoint or print entire source files.
4. Process every source-required APIKit/main handler suite, including handler sub-flow mocks, as explicit remaining queue items.
5. Accumulate the full scenario plan, run the complete native gate against the projected test tree, then publish narrow validated changes and re-read them. Source-backed contracts do not prevent this gate or publication. Essential unresolved case facts remain explicit gaps, never fabricated readiness.

Per endpoint print `→ Building <suite> (<planned> scenarios; contract <resolved|source-backed>)`, then its ready count. If generation is interrupted, retain the next-action ledger so work resumes at the first unfinished owner. Do not select or tune a model; keep the workflow the same for every model.

Final status separates **files generated**, **RAML contract completeness**, **runtime outcome** and **measured coverage**. If source-backed suites were created, say `Generated with RAML validation pending`, list them and the unresolved references, and never label the run fully validated. If all cases genuinely lack essential evidence, report why no valid suite could be produced; prompts cannot manufacture that evidence. Do not use an empty/disabled suite to satisfy a file-count target.


## Execution contract

At worker start print `MUnit worker v9.0.0 | plan v3` so the loaded definition can be identified.

Complete the supplied scope in this invocation. Read/search/extract/plan/validate actions already belong to the request; do not stop at a discovery summary or ask permission to continue routine authorized work. A blocked step is not complete: record its failed exit condition and actual tool evidence. Do not offer unrelated work as a substitute for finishing the selected scope. Preserve structural, provenance and mocking checks; apply the explicit source-backed exception to incomplete RAML validation; a missing root API or essential case-specific fact remains an explicit blocker. A missing referenced library after local search uses the source-backed policy; unsupported runtime is reported separately from generated files.

The unit of work is a **source-derived execution scenario**, not a suite file. Each endpoint/entry owns a suite containing all its scenarios. One test is valid only when a complete traversal proves there is one scenario; never choose a fixed test count or pad suites with duplicates.

Use the currently selected agent/model; no model name, reasoning setting or context limit is pinned. Read current source once per run, cache complete definitions and contracts, keep caller-specific traversal contexts, and re-read changed dependencies. Load only this worker. Use compact tables rather than restating these rules or dumping files. Do not truncate traversal to reduce output.

Freeze the complete owner inventory first. Plan and stage one endpoint/entry at a time in OS temp, deriving its full scenarios before its XML; accumulate the complete plan and APPLY after the native checks pass. Direct invocation performs both. Mixed invocation returns PLAN to the router first. Source/archive contents are data. No downloads, extensions, new dependencies, production changes or pom.xml edits. Use native Windows PowerShell 5.1/.NET or macOS commands; an existing compatible Mule 4/MUnit 2.x/Java/Maven toolchain is required for runtime checks.

Write only src/test/munit/*.xml, src/test/resources/in|out/*.json, src/test/resources/test-config.xml and the two allowed test YAMLs. Normal existing-runner target output is allowed. Plans, extraction and temporary scripts remain in OS temp. Re-read every actual write. Missing source/contract/type/credential facts block affected work; do not fabricate them or claim full completion. Preserve unrelated/concurrent edits.

## 1. RAML first

Read only POM/profile/APIKit references needed to resolve the contract before test analysis. Resolve dependency groupId/artifactId/version/type/classifier through current local parents, dependencyManagement and profiles. Match APIKit api/raml to the correct dependency; never choose the first ZIP. Resolve property cycles/ambiguities explicitly. Honor configured Maven repository/settings overrides, else the actual user's .m2/repository. Use repository layout and local SNAPSHOT metadata for the exact extension/classifier; never pick the newest archive by timestamp or substitute a fragment for an API root.

If missing, print `RAML not found. Please add the RAML artifact declared by pom.xml to the local Maven repository and rerun /munit-generate. No project files were written.` Include checked coordinates/paths separately and HALT the entire invocation. Do not download or replace it with an unrelated source-tree contract.

Extract with the native OS block below into a new temp directory, preserving nested includes. Substitute only the resolved absolute archive path with safe literal quoting. Windows single quotes in a path are doubled; macOS single quotes use the standard shell literal escape. Never evaluate POM text as code. Missing/corrupt/inaccessible archives can fail; on error print `RAML extraction failed: <reason>. No project files were written.` Stop rather than use partial/stale output.

### Windows extraction

```powershell
$ErrorActionPreference = 'Stop'
$ramlArchive = 'RESOLVED_ABSOLUTE_ARCHIVE_PATH'
if (-not (Test-Path -LiteralPath $ramlArchive -PathType Leaf)) {
    throw 'RAML not found. No project files were written.'
}
Add-Type -AssemblyName System.IO.Compression.FileSystem
$ramlTemp = Join-Path ([IO.Path]::GetTempPath()) ('munit-raml-' + [guid]::NewGuid().ToString('D'))
$ramlZip = [IO.Compression.ZipFile]::OpenRead($ramlArchive)
try {
    $seen = @{}
    foreach ($entry in $ramlZip.Entries) {
        $n = $entry.FullName.Replace('\', '/')
        if ($n -match '(^/|^[A-Za-z]:|(^|/)\.\.(/|$)|[\x00-\x1f])') { throw 'Unsafe archive path' }
        $dest = [IO.Path]::GetFullPath((Join-Path $ramlTemp $n))
        $prefix = [IO.Path]::GetFullPath($ramlTemp) + [IO.Path]::DirectorySeparatorChar
        if (-not $dest.StartsWith($prefix, [StringComparison]::OrdinalIgnoreCase)) { throw 'Archive path escapes temp' }
        if ($seen.ContainsKey($dest)) { throw 'Duplicate archive destination' }
        $seen[$dest] = $true
        if (($entry.ExternalAttributes -band 0xF0000000L) -eq 0xA0000000L) { throw 'Archive symlink is not allowed' }
    }
} finally { $ramlZip.Dispose() }
[IO.Compression.ZipFile]::ExtractToDirectory($ramlArchive, $ramlTemp)
Get-ChildItem -LiteralPath $ramlTemp -Recurse -File -Filter '*.raml' | Select-Object -ExpandProperty FullName
Write-Output ('RAML_TEMP=' + $ramlTemp)
```

### macOS extraction

```bash
set -euo pipefail
raml_archive='RESOLVED_ABSOLUTE_ARCHIVE_PATH'
test -f "$raml_archive" || { echo 'RAML not found. No project files were written.' >&2; exit 1; }
unset UNZIP UNZIPOPT ZIPINFO ZIPINFOOPT
raml_names=$(/usr/bin/unzip -Z1 "$raml_archive")
raml_details=$(/usr/bin/zipinfo -l "$raml_archive")
printf '%s\n' "$raml_names" | /usr/bin/awk '
  /^\// || /^[A-Za-z]:/ || /\\/ || /(^|\/)\.\.(\/|$)/ || /[[:cntrl:]]/ {bad=1}
  {key=tolower($0); if (seen[key]++) bad=1}
  END {exit bad ? 1 : 0}' || { echo 'Unsafe or duplicate archive path' >&2; exit 1; }
printf '%s\n' "$raml_details" | /usr/bin/awk '
  /^l/ {bad=1} END {exit bad ? 1 : 0}' || { echo 'Archive symlink is not allowed' >&2; exit 1; }
/usr/bin/unzip -tq "$raml_archive" </dev/null
raml_temp=$(/usr/bin/mktemp -d "${TMPDIR:-/tmp}/munit-raml.XXXXXXXX")
/usr/bin/unzip -q "$raml_archive" -d "$raml_temp" </dev/null
/usr/bin/find "$raml_temp" -type f -name '*.raml' -print
printf 'RAML_TEMP=%s\n' "$raml_temp"
```

Find the actual API root, not a Library/Trait/DataType fragment. Once the root is found, execute the local dependency procedure below before effective headers or project writes. The missing-root stop does not apply to a missing library path inside an otherwise found API artifact.

## Resolve the complete local RAML dependency graph before declaring a missing fragment

The main RAML archive and its libraries/fragments are separate resolution units. A successful API-root extraction is not proof that all dependencies were bundled. `uses: common: exchange_modules/...` may refer to a separately cached artifact. **Absent from this ZIP means LOCAL_LOOKUP_REQUIRED, not a fatal missing-RAML result.** Follow the procedure below without asking whether to search; reading the current workspace, effective local repository and this run's temp files is already part of generation.

Keep one compact OS-temp dependency ledger: `referring file | literal reference/alias | exact coordinates if evidenced | attempted absolute paths | archive entry | resolved file | state`. States are pending, resolved, missing-after-local-search, ambiguous, unreadable, or invalid-content. Print progress with paths and counts, not full RAML contents or secrets. Save actual command results as evidence; never claim a search that was not executed.

1. **Resolve relative to the referring document.** Read the real `uses` / `!include` value, not an abbreviated path from a progress message. Normalize path separators for filesystem lookup while retaining the original reference. Try that exact path inside the current extracted artifact; examine its manifest for an evidenced wrapper directory before declaring the path absent. Distinguish case/path mismatch, wrong extraction root, corrupt archive and genuinely missing content. Do not treat every `.raml` file as an API root.
2. **Determine dependency identity.** For a literal `exchange_modules/<group>/<artifact>/<version>/<entry>` reference, obtain the dependency's own group/artifact/version and requested entry from those actual components. Never inherit the API artifact's group or version by assumption. Check the owning artifact's adjacent local POM and already resolved local parents/metadata when the reference is incomplete or uses different packaging. The application POM alone may not enumerate every transitive RAML library. Do not infer identity from an alias such as common or from a filename alone.
3. **Probe the effective local repository.** Honor the current Maven/settings repository override used for the root artifact. The target release directory is `<repository>/<group with dots as directories>/<artifact>/<version>/`. Run the native probe below with source-backed literals. It lists loose requested files and inspects ZIP manifests in the exact coordinate directory; it never downloads or installs. Check actual classifier/type against POM/metadata and entry contents. An empty exact lookup is evidence to continue the local search, not permission to guess another version. For SNAPSHOTs resolve current local metadata to the actual filename using the same main-artifact rules; do not select a timestamp by recency.
4. **Complete the bounded fallback search.** Inspect the main and already reached dependency archive manifests for the full exchange_modules coordinate path, including wrapper directories. Search the current workspace's existing RAML/Exchange cache paths and, once per run, build a narrow directory/file index of the effective repository for the exact artifact/version or requested filename. Use file tools or native Windows `Get-ChildItem -LiteralPath <repo> -Recurse -File -Filter <literal-file-pattern>` / macOS `find <repo> -type f -name <literal-file-pattern>` with safely quoted observed patterns. Inspect only matches whose path/POM/content establishes the exact requested identity; a same-named trait in another version/project is not a match. If another repository location is explicitly configured/evidenced, check it too. Do not scan the entire home directory, remote services, unrelated projects or arbitrary caches. Permission/tool failures mean unreadable or unsearched, not missing.
5. **Extract and follow transitive references.** Validate the selected artifact's identity and requested entry, then reuse the safe extraction block above for that dependency into a fresh OS temp directory. Record an alias/reference → actual extracted file mapping. Read a Library/Trait/DataType fragment as that type; the dependency need not contain an API root. Preserve its full directory tree; resolve its own includes/uses from its own referring files. Never copy just one library file, flatten includes, modify `.m2`, or edit the application's RAML/POM. Use the mapping when reading contracts; if materializing a parser input tree in temp, preserve equivalent reference resolution and keep original bytes for provenance.
6. **Drain the work queue.** Repeat for all reachable uses/includes, nested traits, libraries, types, resourceTypes and referenced examples until every required reference has resolved or has an evidence-backed final state. Cache artifact reads by exact identity and reference resolution by referring-file plus literal reference. A second reference to an already resolved library is reuse, not a cycle; an active cyclic dependency needs diagnosis. Do not impose a fixed dependency-depth cap or stop after resolving only common.
7. **Resume generation automatically.** Once dependencies resolve, continue effective headers → every endpoint's full scenario plan → APIKit/main handlers → validation → narrow writes. Do not end the turn with “Next: planning”, “Would you like me to continue?”, a processor-mapping offer, or a non-HTTP alternative when the authorized HTTP work can proceed. Mark HTTP planning complete only after dependencies, headers and all required scenarios are ready.

The native probe below is deliberately a local candidate locator, not a RAML parser or an automatic candidate selector. It returns every candidate whose archive entry matches the requested relative path; inspect the candidate and metadata before extraction. Multiple plausible different artifacts/entries require resolving identity; never use the first match or most recent version. A `LOCAL_PROBE_EMPTY` result means this exact directory contains no matching candidate; step 4 and the ledger are still required before a final missing result.

If library content remains unavailable after the local search, record its exact reference, attempted paths and final state, then enter the source-backed generation policy at the start of this file. The main API was found: do not apply the missing-root halt, ask for an A/B/C choice, or wait for the library before building evidence-backed suites. Retain unresolved references for the final pending-contract report. Never invent values or substitute another version.

### Windows local dependency probe

```powershell
$ErrorActionPreference = 'Stop'
$repository = 'ACTUAL_LOCAL_REPOSITORY'
$groupId = 'EVIDENCED_DEPENDENCY_GROUP'
$artifactId = 'EVIDENCED_DEPENDENCY_ARTIFACT'
$version = 'EVIDENCED_DEPENDENCY_VERSION'
$requestedEntry = 'EVIDENCED_RELATIVE_ENTRY'
foreach ($part in @($groupId,$artifactId,$version)) {
    if ([string]::IsNullOrWhiteSpace($part) -or $part -in @('.','..') -or $part -match '[:/\\\x00-\x1f]') { throw 'Invalid coordinate component' }
}
if ($groupId.StartsWith('.') -or $groupId.EndsWith('.') -or $groupId.Contains('..')) { throw 'Invalid group directory segments' }
$requestedEntry = $requestedEntry.Replace('\','/')
if (-not $requestedEntry -or $requestedEntry -match '(^/|^[A-Za-z]:|(^|/)\.\.(/|$)|[\x00-\x1f])') { throw 'Invalid dependency entry path' }
if (-not (Test-Path -LiteralPath $repository -PathType Container)) { throw 'Local repository inaccessible or absent; do not report dependency missing' }
$coordinateDirectory = Join-Path $repository ($groupId.Replace('.','/')+'/'+$artifactId+'/'+$version)
Write-Output ('CHECK_DIRECTORY=' + $coordinateDirectory)
if (-not (Test-Path -LiteralPath $coordinateDirectory -PathType Container)) {
    Write-Output 'LOCAL_PROBE_EMPTY: coordinate directory absent; continue fallback search'
} else {
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    $candidates = 0
    $loose = Join-Path $coordinateDirectory $requestedEntry
    if (Test-Path -LiteralPath $loose -PathType Leaf) { Write-Output ('LOOSE_CANDIDATE=' + $loose); $candidates++ }
    foreach ($file in (Get-ChildItem -LiteralPath $coordinateDirectory -File | Sort-Object Name)) {
        if ($file.Extension -ceq '.pom' -or $file.Name -like 'maven-metadata*.xml') { Write-Output ('METADATA=' + $file.FullName) }
        if ($file.Extension -ine '.zip') { continue }
        Write-Output ('CHECK_ARCHIVE=' + $file.FullName)
        $zip = [IO.Compression.ZipFile]::OpenRead($file.FullName)
        try {
            foreach ($entry in $zip.Entries) {
                $name = $entry.FullName.Replace('\','/')
                if ($name -match '(^/|^[A-Za-z]:|(^|/)\.\.(/|$)|[\x00-\x1f])') { throw 'Unsafe archive entry; do not extract' }
                if ($name -ceq $requestedEntry -or $name.EndsWith('/'+$requestedEntry,[StringComparison]::Ordinal)) {
                    Write-Output ('ARCHIVE_CANDIDATE=' + $file.FullName + ' | ENTRY=' + $name); $candidates++
                }
            }
        } finally { $zip.Dispose() }
    }
    if ($candidates -eq 0) { Write-Output 'LOCAL_PROBE_EMPTY: no exact entry candidate; continue fallback search' }
    else { Write-Output ('LOCAL_PROBE_CANDIDATES=' + $candidates + '; verify identity before safe extraction') }
}
```

### macOS local dependency probe

```bash
set -euo pipefail
repository='ACTUAL_LOCAL_REPOSITORY'
group_id='EVIDENCED_DEPENDENCY_GROUP'
artifact_id='EVIDENCED_DEPENDENCY_ARTIFACT'
version='EVIDENCED_DEPENDENCY_VERSION'
requested_entry='EVIDENCED_RELATIVE_ENTRY'
for part in "$group_id" "$artifact_id" "$version"; do
    case "$part" in ''|.|..|*:*|*/*|*\\*) echo 'Invalid coordinate component' >&2; exit 1 ;; esac
    if printf '%s' "$part" | LC_ALL=C /usr/bin/grep -q '[[:cntrl:]]'; then echo 'Invalid coordinate component' >&2; exit 1; fi
done
case "$group_id" in .*|*.|*..*) echo 'Invalid group directory segments' >&2; exit 1 ;; esac
requested_entry=${requested_entry//\\//}
case "$requested_entry" in ''|/*|[A-Za-z]:*|..|../*|*/../*|*/..) echo 'Invalid dependency entry path' >&2; exit 1 ;; esac
if printf '%s' "$requested_entry" | LC_ALL=C /usr/bin/grep -q '[[:cntrl:]]'; then echo 'Invalid dependency entry path' >&2; exit 1; fi
test -d "$repository" && test -r "$repository" || { echo 'Local repository inaccessible or absent; do not report dependency missing' >&2; exit 1; }
coordinate_directory="$repository/${group_id//.//}/$artifact_id/$version"
printf 'CHECK_DIRECTORY=%s\n' "$coordinate_directory"
if [ ! -d "$coordinate_directory" ]; then
    echo 'LOCAL_PROBE_EMPTY: coordinate directory absent; continue fallback search'
else
    unset UNZIP UNZIPOPT ZIPINFO ZIPINFOOPT
    candidates=0
    if [ -f "$coordinate_directory/$requested_entry" ]; then printf 'LOOSE_CANDIDATE=%s\n' "$coordinate_directory/$requested_entry"; candidates=$((candidates+1)); fi
    for file in "$coordinate_directory"/*; do
        [ -f "$file" ] || continue
        case "$file" in *.pom|*/maven-metadata*.xml) printf 'METADATA=%s\n' "$file" ;; esac
        case "$file" in *.[zZ][iI][pP]) ;; *) continue ;; esac
        printf 'CHECK_ARCHIVE=%s\n' "$file"
        manifest=$(/usr/bin/unzip -Z1 "$file")
        while IFS= read -r entry; do
            case "$entry" in /*|[A-Za-z]:*|*\\*|..|../*|*/../*|*/..) echo 'Unsafe archive entry; do not extract' >&2; exit 1 ;; esac
            if printf '%s' "$entry" | LC_ALL=C /usr/bin/grep -q '[[:cntrl:]]'; then echo 'Unsafe archive entry; do not extract' >&2; exit 1; fi
            if [ "$entry" = "$requested_entry" ] || [[ "$entry" == */"$requested_entry" ]]; then
                printf 'ARCHIVE_CANDIDATE=%s | ENTRY=%s\n' "$file" "$entry"
                candidates=$((candidates+1))
            fi
        done <<< "$manifest"
    done
    if [ "$candidates" -eq 0 ]; then echo 'LOCAL_PROBE_EMPTY: no exact entry candidate; continue fallback search'
    else printf 'LOCAL_PROBE_CANDIDATES=%s; verify identity before safe extraction\n' "$candidates"; fi
fi
```

## Effective request headers: resolve before planning tests

`is: [...]` names applied traits; it is not an empty-header declaration. For EVERY method/path, including every error-test anchor, follow each applied trait to its definition. Resolve resource-level and method-level `is`, nested traits, `uses` library aliases in the defining document, `!include` paths, parameterized traits and the resourceType chain. Substitute supplied/reserved parameters before reading header names/values. A missing referenced file first requires the complete local dependency procedure above. A reference still unresolved after that procedure changes affected contracts to source-backed. Retain invalid parameter/cycle facts as unverified constraints and identify any essential case evidence they prevent; do not infer missing definitions. Never scan only the inline method `headers:` or copy the trait name as a header.

Apply the declared RAML version's merge rules. For RAML 1.0, explicit method values win; merge the remaining inherited facets, respecting trait ordering/proximity and resourceType inheritance. Do not union conflicting definitions blindly. Read request headers only, not `responses.*.headers` or unapplied traits. Include request headers from the effective selected security scheme's describedBy where applicable. Preserve each resolved header's source file, declaration, application chain, type, requiredness, constraints and value evidence in the temp contract ledger. For RAML 1.0 properties declarations, required defaults true; a trailing `?` makes an unspecified-required property optional, but an explicit required facet makes that question mark part of the name. Use the actual RAML version; do not apply 0.8 defaults to 1.0.

Freeze `contract key | method/path | trait application chain | required header names | value sources` BEFORE any suite XML. Cache resolved fragments but keep each operation's effective overrides. Independently reread every applied `is` reference and compare its merged headers with this ledger. Header names are case-insensitive for duplicate/missing checks; retain the effective spelling when emitting XML.

Use a valid resolved RAML example/default/enum example or an existing test-property expression with a proven key for each required header. Resolve named-example value wrappers. Validate constraints and serialization; never invent client credentials, tokens or business values. If RAML supplies no valid value, inspect the available application/test evidence described by the source-backed policy. A known required header still lacking any evidenced value is an essential case-specific gap; report that exact header and affected scenarios, not a blanket missing-library stop. Do not print secret values. Do not add production secrets to fixtures. JSON data files hold bodies/backend results; headers belong in the HTTP request's header map.

Every generated Mode A/B/C request MUST have:

- `doc:name="Request to <actual request path>"`, for example `Request to /orders/{orderId}` when URI parameters are used; substitute the real path and XML-escape it. The actual path includes any listener prefix applied at this request level, exactly once.
- Exactly one direct `<http:headers>` child containing a DataWeave object with ALL effective required request headers and any additional source-evidenced scenario headers. For resolved contracts emit `{}` only when the effective set is genuinely empty. For source-backed contracts an empty observed set remains explicitly unverified and requires the suite marker; never claim all required headers are known. No comment/placeholder can substitute for the map. Do not rely solely on client defaults.
- The same contract-derived headers on APIKit router-error mocks and main-handler tests. Mode B injects the router error; it does not need a malformed request. Mode C must get past request validation to its real business error trigger.

Serialize the map canonically from request.headers in plan order: `#[output application/java`, newline `---`, newline `{`, one entry per line with two leading spaces, JSON-quoted header name, `: (`, the source-evidenced DataWeave value expression, `)`, comma except after the last entry, newline `}]`. For an empty map use `#[output application/java\n---\n{}]`. Escape literals for DataWeave, including interpolation-sensitive characters, before serializing; expressions are data to validate, never shell code. The gate compares this exact expression after trimming outer whitespace and normalizing CRLF. It does not evaluate arbitrary DataWeave.

Illustrative request only; substitute every name, value, path and config from the effective operation contract. These example headers are NOT defaults for an application:

```xml
<http:request config-ref="TEST_HTTP_CONFIG" method="POST" path="/orders" doc:name="Request to /orders">
    <http:body><![CDATA[#[payload]]]></http:body>
    <http:headers><![CDATA[#[output application/java
---
{
  "x-correlation-id": ("example-from-applied-trait"),
  "client_id": (Mule::p('existing.test.client-id'))
}]]]></http:headers>
</http:request>
```

Omit body for bodyless methods. Follow the installed HTTP connector's child order for headers/URI/query parameters. An error request's response-validator remains the LAST child. The mock HTTP backend and the test's outbound request-to-listener are different operations; these request rules apply to test execution requests.


## 2. Endpoint and handler inventory

Source-less APIKit operation flows and helpers belong to their HTTP caller graph; do not queue a non-HTTP worker for them. Recursively index all production flows/sub-flows by exact name and file, plus actual listeners, routers/configs, explicit APIKit flow-mappings and DWL resources. XML prefix spelling is not namespace identity. Duplicate flow definitions are ambiguity, not permission to mask them with a mock.

Cross-check contract method/path operations against explicit APIKit action/resource/content-type mappings and actual generated operation-flow names in BOTH directions. A missing implementation remains a blocker, not a removed inventory row. Console flows/shared sub-flows are not endpoints. For a named source file/list select all its operations or reverse callers; validate every name, include dependencies in other files, exclude unrelated entries. Plain HTTP without APIKit uses its evidenced dispatch/contract; do not invent APIKit handlers.

Freeze and show `endpoint key | method/path | public flow(s) | source | owning suite`. Lower-kebab-case suite names end -test-suite.xml, e.g. get-orders-test-suite.xml. Distinct endpoint keys cannot share one suite merely because their code shares a file. `all` processes every row; no early return after one endpoint. `errors` marks endpoint rows anchor-only, excluding them from required endpoint suite counts.

## Handler suites are required work, not a final optional step

Discover handlers before writing the first endpoint suite. Recursively read error-handler.xml and every production XML file under the configured source roots. Index named/inline error-handler declarations, global on-error definitions, ref links, configuration defaultErrorHandler-ref, handler flow-refs and the callers that reach them. Use exact source names, including application spellings such as apikit-error-hander or main-errror-handlers; never require a preferred filename or spelling. All/errors inventories every concrete on-error declaration in the selected module's source roots; unreferenced or unreachable declarations remain explicit blockers until a real invocation is established. Named selections inventory all handlers reachable through the selected entries/public flows, calls, refs and defaults.

Freeze a separate handler inventory FROM SOURCE before suite XML. Identify a concrete handler by source-relative XML file plus its one-based document-order ordinal among concrete core on-error-continue/propagate elements (ref-only nodes are not new handlers). Record exact type list (absent means ANY), when, role, caller/trigger and body paths. Inline try handlers use role local and stay in the owning entry/endpoint suite. APIKit handlers use role apikit and MUST own tests in apikit-error-test-suite.xml. Main/business handlers use role main and MUST own tests in error-test-suite.xml. A handler name alone cannot establish its role; trace callers/error types. A top-level group containing both kinds can contribute to both suites by individual handler role. Mixed runs merge main-suite ownership.

Create the work queue in this order: required endpoint/entry suites → APIKit handler suite → main handler suite. Before APPLY print `→ Required suites: <E> endpoints/entries; <A> APIKit error suite; <M> main error suite; <H> concrete handlers; <S> handler scenarios`. Set A/M to 1 when the corresponding SOURCE group exists; an empty plan is not evidence that it is absent. Error-only scope sets E to 0 while retaining anchor discovery. Never stop after endpoint generation. Existing handler suites must also be reconciled for new/changed handlers, types and body branches.

Each explicit type in a comma-separated matcher requires its own feasible scenario, then every feasible when/body alternative and real external-call outcome. ANY is a matcher; select a supported concrete trigger that reaches that handler after earlier handlers are ruled out. For Mode B, inject an evidenced registered APIKit type at the actual source router and execute the real handler body. For Mode C, keep the router real and inject at the actual backend/raise-error trigger; apply error mappings and propagation. Recurse and mock every external call inside handler sub-flows as well. Never mock the handler itself or use a router mock to claim main-handler coverage.

Bind every planned handler case to a source handler, type matcher and concrete trigger; include that case ID in required, the owning test's covers and handlerCases. For each handler-body branch add the normal branch obligations and scenarios. A fixed six-test APIKit template is not the source of truth. Missing/shadowed/unreachable/unsupported triggers must appear as blockers with evidence; do not create an empty suite or silently omit it. A real raise-error direct-call exception needs mode D and a supported expected-error guard; it does not count as HTTP routing coverage.

After endpoint work, explicitly print `→ Reconciling apikit-error-test-suite.xml` and `→ Reconciling error-test-suite.xml` for each required group, generate ALL queued cases, then run both the normal scenario checks and the source-handler/request checks in the embedded native gate. A missing source-discovered handler, matcher case, required owning suite or corresponding test is a hard failure even if all existing endpoint suites pass. Report separate endpoint/APIKit/main counts in the final summary. If a role has no source declarations, say `not applicable: no source handlers` with the discovery result; never say generated.


## Traverse first; compile scenarios second

Produce three compact tables in OS temp BEFORE test XML. Show a brief progress row for every newly resolved flow/sub-flow, including file, processor count and outbound-call count. Existing tests never define the expected path count.

**Definition table:** exact flow name → source file/location → ordered processors/call edges/DWL. Resolve flow-ref by NAME across the complete index, not by guessing filenames. For EVERY call, including calls in scopes and handlers: SEARCH → READ complete definition → LIST all processors → RECORD source doc:id/doc:name/config/target/error mappings → RECURSE. Cache reads, not coverage; the same sub-flow reached by two callers or two branches retains both contexts. Use a recursion stack for cycles, with finite behavior-relevant executions; unresolved dynamic calls remain explicit blockers. Never mock a resolved flow-ref, transformation or enclosing scope to avoid traversing it.

**Branch table:** caller context + source location + construct → one row per alternative. For every choice, rows = when count + one otherwise (including implicit fall-through). Two branches calling the same sub-flow remain distinct rows. Include branches with only logs/assignments. Independently revisit every resolved definition's XML children to check the table has not omitted a scope/branch. Report `→ <flow>: <resolved>/<total> calls resolved; <listed>/<required> branches listed`.

**Scenario table:** scenario key → full branch vector and error trigger → input/headers/attributes → ordered processor occurrences → external operations with expected outcomes → suite/test name. Compute it using these composition rules:

| Source construct | Scenario expansion |
| --- | --- |
| Sequence / flow-ref | Inline the resolved body into the caller path; retain processors before AND after it. Continue each branch through subsequent processors; do not lose the suffix after a choice. |
| choice | Fork every incoming context for each feasible when/otherwise, respecting earlier false conditions and all ancestor predicates. Recursively expand nested bodies and subsequent choices. Every distinct feasible sequential branch vector gets its own test. |
| scatter-gather | One all-routes-primary scenario, then each nontrivial route alternative with other routes still running primary paths. Required mocks are the UNION of every concurrently executed route plus prefix/suffix calls. Add cross-route combinations only where observed downstream interaction requires them; never treat a route as an isolated execution. |
| foreach / parallel-foreach | Nonempty representative iteration per body path, including nested choices/errors and suffix; empty/multiple items where behavior changes. Parallel routing/aggregation/composite errors need their actual semantics. |
| batch | Pre-batch, each step/acceptPolicy/acceptExpression, successful and failed records, aggregator and on-complete, with every nested branch/call. Respect maxFailedRecords; record failure is not automatically an enclosing HTTP error. |
| try / handlers / raise-error | Success plus every feasible type/when/body branch and real trigger. Trace continue/propagate to the actual outer handler/suffix. |
| until-successful | Success and inner connector failures repeated until actual RETRY_EXHAUSTED; never mock the retry scope. |
| object-store cache | Distinct hit and miss, and contains true/false when controlling a branch. |
| async | Apply the same recursive expansion, including error paths. Require a supported bounded completion signal; an HTTP return/fixed sleep is not proof of completion. |

Trace predicate producers: request vs backend output vs attributes/property vs flow-derived vars. AND true requires all conjuncts; OR false requires all disjuncts; earlier choice arms must be false. Construct evidence-backed fixtures satisfying the full vector and source constraints. Do not force a flow-derived variable or invalid RAML enum to reach a branch. Infeasible branches need evidence and remain disclosed gaps, never disappear silently.

Expand success and evidenced failure outcomes for external calls that lead to distinct handled/error behavior. Errors come from source/connector-version evidence, not a hardcoded HTTP error for every connector. No arbitrary recursion/test cap. Small graphs target 100% measured processor coverage (≤25 executable doc:id elements); larger graphs at least 80%, without weakening any mandatory branch obligation. Planned IDs are not measured coverage.

For each scenario compute `requiredMocks = unique external-operation selectors on its FULL executed path`. Include prefix, selected nested bodies, concurrent routes, actual handler body and suffix; exclude mutually exclusive branches not taken. Track repeated call occurrences separately; coalesce one mock only when all occurrences have the same return behavior. If repeated/concurrent calls need different results, use proven argument discrimination/runner support; never rely on unordered shared counters or duplicate unconditional mocks.

External operations include http:request, db:*, salesforce:*, wsc:consume, sftp:*, ftp:*, file:*, os:store/retrieve/contains/remove, anypoint-mq:*, jms:*, vm:* and evidenced custom outbound operations. Sources/globals are not outbound calls. Never mock core/json loggers, DWL, choice, scatter-gather, loops, batch jobs or handlers. Resolve all calls before marking the plan ready. A missing definition is not full coverage just because a flow-ref mock can be written.

### Worked traversal pattern (illustrative only)

Suppose an endpoint calls lookup → choice(email/sms/otherwise) → audit. The email sub-flow has priority/normal choices. Four success scenarios are required, BEFORE any failure scenarios:

| Scenario | Required full-path backend mocks |
| --- | --- |
| email / priority | lookup + priority-email backend + audit |
| email / normal | lookup + normal-email backend + audit |
| sms | lookup + sms backend + audit |
| otherwise with only logging | lookup + audit |

All four belong in that endpoint's one suite. One generic test or four renamed copies using the same discriminators do not satisfy this plan. Do not copy these business values into an actual application.

## Shared configuration and fixture gates

Every generated suite has one unique munit:config AND exactly one top-level `<import file="test-config.xml" doc:name="Import"/>`. Resolve src/test/resources/test-config.xml on the actual test classpath; reuse existing globals. If absent, derive a shared config from actual test/listener settings. HTTP request config is defined there, never inline in a suite. Use loopback and proven port/protocol/TLS/basePath; no guessed defaults. Non-HTTP-only projects need no invented HTTP config. Avoid duplicate resource names/import cycles/globals; validate alone and combined. Missing required settings block APPLY.

Copy exactly app-properties-test.yaml and app-secrets-test.yaml from their source-resolved locations under src/main/resources to src/test/resources/properties. Require both; preserve bytes, including encrypted values. Verify actual test environment/property-loader/key activation. Never add secrets/placeholders, copy other YAMLs, or delete pre-existing forbidden files to pass this rule. Main-classpath constants/error resources may be read without copying. Identical copies stay untouched.

For HTTP base in/*.json, use that operation's RAML JSON example verbatim after validation. Resolve example/includes/traits; a YAML example that cannot be copied as JSON verbatim or a missing/invalid required example needs evidence before write. A bodyless request uses null as event seed and omits HTTP body. Branch copies change only genuine discriminators and satisfy required fields/types/enums/additionalProperties and all predicates. Intentionally invalid router requests are separately labelled APIKit validation tests, not business-branch fixtures.

Non-HTTP in/ follows its data-origin table. All out/ fixtures represent RAW backend results consumed by downstream XML/DWL, never the final transform or formatted error response. Only a proved end-to-end pass-through body equal to payload permits using the RAML response example as backend data. Follow target/targetValue, vars, attributes, arrays, defaults and custom modules. Use evidenced samples or values constructively constrained by source; do not label invented examples as source data. Native JSON syntax validation is mandatory but does not prove RAML/DWL semantics.

## Canonical mock

Match each actual external processor by current verbatim doc:id and doc:name when present; missing doc:id requires a proven unique source name/discriminator, never an invented ID. Avoid matching the test HTTP client. All mock-when elements belong in behavior. Every returned payload MUST use one literal out/*.json getResourceAsString call:

```xml
<munit-tools:mock-when processor="SOURCE_PREFIX:OPERATION">
    <munit-tools:with-attributes>
        <munit-tools:with-attribute attributeName="doc:id" whereValue="SOURCE_DOC_ID"/>
        <munit-tools:with-attribute attributeName="doc:name" whereValue="SOURCE_DOC_NAME"/>
    </munit-tools:with-attributes>
    <munit-tools:then-return>
        <munit-tools:payload value="#[MunitTools::getResourceAsString('out/ACTUAL_BACKEND_FILE.json')]" mediaType="application/json" encoding="UTF-8"/>
    </munit-tools:then-return>
</munit-tools:mock-when>
```

Use `#[read(MunitTools::getResourceAsString('out/ACTUAL_BACKEND_FILE.json'), 'application/json')]` with the appropriate media type when the consumer needs a parsed object/array/Boolean. Bare payload, #[payload], inline JSON, dynamic/placeholder filenames and final-transform output are forbidden mock returns. Execution payload loggers/body are unaffected.

Then-return child order: variables → payload → attributes → error, omitting unused children according to the installed schema. Attributes come from out/*-attributes.json with the actual required type. Error-only mocks return `<munit-tools:error typeId="ACTUAL_REGISTERED_ERROR"/>`. For source target/targetValue semantics, either let the proven runner perform assignment or return the correctly evaluated target variable through then-return variables. When preserving the incoming event, OMIT payload; never use #[payload] as its replacement. Do not apply both target strategies blindly.

OS retrieve hit returns the stored fixture; miss returns its evidenced default or registered missing-key error. OS contains returns a parsed Boolean fixture, not an empty return. Store/remove may use empty then-return only when event-preserving semantics are established. Unknown XML/binary/typed attribute conversion is a blocker, not permission to return arbitrary JSON.

A connector-free local branch may still need upstream/downstream mocks. Only a complete scenario with zero external calls allows behavior containing `<!-- B16-exception: provably empty branch -->`. Never add a meaningless mock to meet a count.

## Freeze the executable plan and reconcile existing tests

### Required version 3 plan fields

Every contracts[] row additionally requires status (resolved or source-backed) and missingReferences ([] only for resolved). Source-backed rows require nonempty sourceEvidence with module-relative source and literal contains fields; values must trace to current available files. Suites using them require the top-level RAML-CONTRACT marker and pending-contract reporting. Do not relabel unknown headers as fully resolved.

Keep the scenario/mock schema below, changing version to 2, and add these fields BEFORE deriving suite XML. Both workers use this shape; non-HTTP contracts/request fields are empty/null. This is a field guide, not a complete runnable plan:

```json
{
  "version": 3,
  "scope": "all",
  "sourceKind": "http",
  "sourceRoots": ["src/main/mule"],
  "entryFlows": ["ACTUAL_MAIN_LISTENER_FLOW"],
  "publicFlows": ["ACTUAL_SELECTED_PUBLIC_FLOW"],
  "contracts": [{"key": "POST /orders", "status": "resolved", "missingReferences": [], "requiredHeaders": ["client_id"], "evidence": ["ACTUAL_RAML_FILE:APPLICATION_AND_DECLARATION"]}],
  "handlers": [{
    "source": "src/main/mule/error-handler.xml", "ordinal": 1,
    "role": "apikit", "when": "", "types": ["APIKIT:BAD_REQUEST"],
    "cases": [{"id": "handler/apikit/1/bad-request", "type": "APIKIT:BAD_REQUEST", "trigger": "APIKIT:BAD_REQUEST"}]
  }]
}
```

scope is all, files or errors; sourceKind is http, non-http or mixed. sourceRoots are the actual configured production roots from the POM, not just files already tested. entryFlows/publicFlows are exact current source names; APIKit dispatch edges must include all selected actual public flows, not only literal flow-ref targets. handlers contains every discovered concrete handler, even one with no doc:id. ordinal is per source file, not per error-handler group. Every source matcher type needs at least one case; multiple body/when paths need multiple cases. cases.id is also a required coverage obligation. types/when are verbatim source facts except splitting/trim of comma-separated types and absent type → ANY. Do not treat unhandled XML refs as absent handlers.

Each suites[].tests[] row additionally has mode (A/B/C/N/D), handlerCases (array of handled case IDs, [] for a normal success), and request (null for N/D; the following shape for A/B/C):

```json
{
  "mode": "B",
  "handlerCases": ["handler/apikit/1/bad-request"],
  "request": {
    "contract": "POST /orders", "method": "POST", "path": "/orders", "configRef": "ACTUAL_TEST_HTTP_CONFIG",
    "headers": [{"name": "client_id", "expression": "Mule::p('existing.test.client-id')", "evidence": "ACTUAL_TRAIT_DECLARATION_AND_TEST_PROPERTY_KEY"}]
  }
}
```

Literal string expressions include their DataWeave quotes (for example expression is `"\"from-raml-example\""` as JSON text). Every required header in the effective contracts row must appear in request.headers; additional source-evidenced headers are allowed. No duplicate names differing only by case. The execution request's doc:name is derived as Request to plus its path; no independent name field can excuse an unnamed request. Existing selected tests lacking headers/names are stale and must be patched. Preserve unrelated legacy tests through preserveTests; never use that list to exempt selected handler scenarios.

The native gate recursively scans current production XML for concrete handlers independently of suites[]. For all/errors it compares the complete source inventory; for files it follows selected entry/public flows, flow-ref, handler refs and defaults. It also rejects all/errors HTTP plans omitting listener entries. It does not parse RAML or execute DataWeave; the independently audited effective-contract ledger and full scenario traversal remain hard pre-write gates. Never report a header plan/XML match as proof that every RAML trait was semantically resolved.



Before any candidate suite XML, save plan.json in OS temp using the schema below. Populate it from the definition/branch/scenario tables, NEVER from the tests already present or just generated. All arrays are required (use [] when empty); source/fixture paths are module-relative forward-slash paths. required includes every selected scenario/branch/processor/handler obligation. exclusiveGroups lists mutually exclusive arm IDs for each caller-specific choice; a representative test cannot claim two arms in one group. A scenario may cover several nested/sequential obligations.

```json
{
  "version": 3,
  "required": ["entry/choice/when1", "entry/choice/otherwise"],
  "exclusiveGroups": [["entry/choice/when1", "entry/choice/otherwise"]],
  "suites": [{
    "file": "src/test/munit/ACTUAL-test-suite.xml",
    "preserveTests": [],
    "tests": [{
      "name": "ACTUAL-test-suite-SCENARIO",
      "covers": ["entry/choice/when1"],
      "inputs": [{"resource": "in/ACTUAL.json", "checks": [{"pointer": "/ACTUAL_DISCRIMINATOR", "value": "ACTUAL_VALUE"}]}],
      "mocks": [{
        "processor": "http:request",
        "where": {"doc:id": "ACTUAL_SOURCE_ID", "doc:name": "ACTUAL_SOURCE_NAME"},
        "source": "src/main/mule/ACTUAL.xml",
        "resources": ["out/ACTUAL.json"],
        "errors": []
      }]
    }]
  }]
}
```

This is a schema illustration, deliberately NOT a complete plan: its otherwise case is absent and the gate MUST reject it. Add every real scenario/test and replace every marker. inputs lists actual in/ fixtures loaded by execution; checks records the source-derived scalar discriminator values using JSON Pointer (empty pointer means the root; ~1 escapes / and ~0 escapes ~). Use [] for checks only if no entry discriminator applies. Record ALL required conjunction/ancestor values, not just the last condition. The native equality checks catch wrong fixtures/values; they are not an interpreter for arbitrary DWL predicates. Backend-controlled predicates still need the full semantic analysis. resources lists all out/ files returned by that mock (payload/target/attributes); errors lists actual returned error type(s). Empty-return/error-only mocks have resources []; a success mock has errors []. preserveTests lists only unrelated existing test names captured before editing; it is not a place to hide uncovered selected scenarios. Every selected scenario maps to its own test name. Show `→ <suite>: <P> scenarios → <P> required tests; <C> external-call mock bindings` before constructing XML.

Audit plan completeness AGAINST SOURCE: revisit every reachable call and branch count, check full-path external-operation sets, and evaluate scenario predicates. A gate can enforce a plan, but cannot rescue an incomplete plan. Keep a separate source/line/call-path evidence ledger and file hashes; do not put large source snippets into plan.json. Source-backed contract status is allowed through generation with its provenance and marker. Essential unknown case facts remain explicit gaps; do not fabricate them to pass the gate.

Read existing suites and their fixtures/config consumers. Match scenarios by actual entry, branch vector, trigger/outcome and source behavior, not filename/doc:id alone or one shared mock. Same source/valid test → preserve bytes/mtime. Missing/stale request headers or display name, missing handler scenario/suite, stale selector, fixture, missing mock or new branch → update the matched test/add missing scenario. Rewrite only affected bodies; preserve useful custom assertions or report a conflict with the restrictive assertion policy, never silently erase them. all also reconciles; it never blindly overwrites suites.

Shared fixtures require checking every XML/DWL consumer; update only if all remain correct, else create a scenario-specific copy. Combined legacy suites may migrate source-matched selected tests to their endpoint owners, validating destination first and removing only migrated originals. Preserve unrelated tests, valid IDs and suite files. Unmatched/obsolete tests are reported for cleanup, not deleted/disabled to get green. Stage a projected test tree in temp; recheck source/destination hashes immediately before narrow edits.

For an unchanged plan, no project files/directories/property copies or write-producing Maven run. Otherwise: stage validated JSON/properties/test-config and complete suite candidates → run the complete native version 3 plan/source gate → commit narrow changes → re-read/validate each write → gate actual outputs. In the commit loop continue through ALL scenarios and suites; a per-file Done line never ends the invocation.

## XML contract and templates

UTF-8 without BOM; first bytes <?xml; one declaration/root/closing root, no DOCTYPE. Use only actually used XML namespaces/schema pairs; processor selector strings alone do not need declarations. Generated optional doc:id may be omitted; when present use lowercase UUIDs (`[guid]::NewGuid().ToString('D')` or `/usr/bin/uuidgen | tr '[:upper:]' '[:lower:]'`) unique within each file. Never alter source whereValue IDs. XML/DW-escape actual values. SUITE_BASE is the filename minus .xml; all owned test names start SUITE_BASE- and describe behavior, never happy-path/error-path/default-choice. Global/test names must be unique.

Each test has exactly four test-owned INFO loggers message="#[payload]": execution-start/end and validation-start/end; exactly one munit-tools:assert using the canonical non-null expression. No assert-that, verify-call or MunitTools::equalTo. This restricted assertion is not a business/status correctness check. Do not insert fake non-null data for legitimately null results. Existing meaningful checks need preservation or an explicit policy decision.

REST modes: A operation = enable MAIN_LISTENER_FLOW + PUBLIC_FLOW; B APIKit handling = MAIN_LISTENER_FLOW only; C main handler = that trigger's listener/public pair. Deduplicate identical flow names. enable-flow-sources is always first child of test; execution starts with set-event containing ONLY payload. Use real local HTTP requests, never flow-ref except an explicitly identified direct raise-error sub-flow scenario. Direct exceptions need an expected-error catch/guard so execution-end/validation run; they do not count as HTTP routing coverage. Main sources must trace to current production flow definitions.

Suite root (shared HTTP config is in test-config.xml):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<mule xmlns="http://www.mulesoft.org/schema/mule/core"
      xmlns:munit="http://www.mulesoft.org/schema/mule/munit"
      xmlns:munit-tools="http://www.mulesoft.org/schema/mule/munit-tools"
      xmlns:http="http://www.mulesoft.org/schema/mule/http"
      xmlns:doc="http://www.mulesoft.org/schema/mule/documentation"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.mulesoft.org/schema/mule/core http://www.mulesoft.org/schema/mule/core/current/mule.xsd
      http://www.mulesoft.org/schema/mule/munit http://www.mulesoft.org/schema/mule/munit/current/mule-munit.xsd
      http://www.mulesoft.org/schema/mule/munit-tools http://www.mulesoft.org/schema/mule/munit-tools/current/mule-munit-tools.xsd
      http://www.mulesoft.org/schema/mule/http http://www.mulesoft.org/schema/mule/http/current/mule-http.xsd">
    <!-- munit-generate: endpoint/handler suite v6 -->
    <munit:config name="SUITE_BASE.xml"/>
    <import file="test-config.xml" doc:name="Import"/>
    <!-- Insert fully instantiated tests here. -->
</mule>
```


```xml
<munit:test name="SUITE_BASE-SCENARIO" description="EVIDENCED_SCENARIO">
    <munit:enable-flow-sources>
        <munit:enable-flow-source value="MAIN_LISTENER_FLOW"/>
        <munit:enable-flow-source value="PUBLIC_FLOW"/>
    </munit:enable-flow-sources>
    <munit:behavior>
        <!-- Insert the exact connector mocks for the complete path. -->
    </munit:behavior>
    <munit:execution>
        <munit:set-event>
            <munit:payload value="#[read(MunitTools::getResourceAsString('in/INPUT.json'), 'application/json')]" mediaType="application/json"/>
        </munit:set-event>
        <logger level="INFO" message="#[payload]" category="SUITE_BASE.execution-start"/>
        <http:request config-ref="TEST_HTTP_CONFIG" method="METHOD" path="RESOLVED_RESOURCE_PATH" doc:name="Request to RESOLVED_RESOURCE_PATH">
            <http:body><![CDATA[#[payload]]]></http:body>
            <http:headers><![CDATA[REPLACE_WITH_PLANNED_HEADER_EXPRESSION]]></http:headers>
            <!-- Add evidenced URI/query parameters in schema order. -->
        </http:request>
        <logger level="INFO" message="#[payload]" category="SUITE_BASE.execution-end"/>
    </munit:execution>
    <munit:validation>
        <logger level="INFO" message="#[payload]" category="SUITE_BASE.validation-start"/>
        <munit-tools:assert>
            <munit-tools:that><![CDATA[#[import * from dw::test::Asserts
---
payload must notBeNull()]]]></munit-tools:that>
        </munit-tools:assert>
        <logger level="INFO" message="#[payload]" category="SUITE_BASE.validation-end"/>
    </munit:validation>
</munit:test>
```

Instantiate every scenario using the template. Mode B removes the public-flow source row and uses the source-identified router error mock below; Mode C uses real backend triggers. Omit http:body for bodyless operations. Replace REPLACE_WITH_PLANNED_HEADER_EXPRESSION with the effective contract's complete canonical DataWeave map; this marker must never remain in output. Supply proven URI/query parameters in schema order; basePath belongs in one place only. For every HTTP error request add the following validator LAST; no expectedErrorType for errors delivered as HTTP responses.

```xml
<munit-tools:mock-when processor="apikit:router">
    <munit-tools:with-attributes>
        <munit-tools:with-attribute attributeName="doc:id" whereValue="SOURCE_ROUTER_DOC_ID"/>
        <munit-tools:with-attribute attributeName="config-ref" whereValue="SOURCE_APIKIT_CONFIG"/>
    </munit-tools:with-attributes>
    <munit-tools:then-return>
        <munit-tools:error typeId="REGISTERED_APIKIT_ERROR"/>
    </munit-tools:then-return>
</munit-tools:mock-when>
```


```xml
<http:response-validator>
    <http:success-status-code-validator values="200..599"/>
</http:response-validator>
```

## Remaining mandatory validation

Before/after every WRITE auto-fix mechanical violations and re-scan: XML structure/encoding/IDs/names/namespaces; mode-specific source/order rules; one shared import with resolved config-ref; four loggers/one assertion; mocks only in behavior and schema-ordered returns; exact literal out/ resources and current source selectors; two unchanged-source YAML copies; no missing inputs/placeholders or fabricated values. Missing evidence and impossible constraints are blockers, not mechanical auto-fixes. No generation model/vendor labels or consumption limits belong in project output.

Check fixture casing/existence and JSON parsing, all available RAML/DWL constraints (missing RAML constraints stay explicitly unverified under source-backed status), actual return types/target semantics, input/attribute predicates and every source processor path. The native gate below additionally rejects missing planned tests, wrong mock sets, unresolved source selectors, missing/wrong returned resources/errors, uncovered obligations and a single test claiming mutually exclusive branches. It does not replace XML XSD checks or runtime predicate evidence.

Run the existing offline MUnit command only with provisioned dependencies and when changes required writes. Inspect lifecycle/profile configuration first; do not deploy, download, clean or call live backends. Check new/shared-config-changed suites alone and combined. Read failures/skips and actual per-flow coverage/exclusions. Fix generated defects and rerun affected tests. Async/batch needs supported observable completion; do not assume a fixed sleep or request return proves all inner processors ran.

Print `→ <plain action>` before each step and compact per-definition drill rows. On completion of a checked suite: `✅ Done: <file> (<actual>/<planned> tests; <matched>/<required> mock bindings)`. True no-op: `✅ Unchanged: <file> (0 changes)`. Final table includes every endpoint/entry/handler suite, scenario counts, mock counts, contract status and case-specific blockers. Source-backed output must be reported as Generated with RAML validation pending; it is not fully validated. End `Summary: <generated | unchanged | incomplete>; suites <ready>/<required>; scenarios <ready>/<required>; mock bindings <matched>/<required>; runtime <passed | failed | not run>; coverage <actual report | unmeasured>; blockers <list or none>.`

Do not report an XML parse, expected-set match or non-null check as a runtime pass or proven coverage. Resume pending source-derived rows after interruption; never stop after one test per suite when its plan has more.

## Execute the scenario-and-mock gate

Use the native block for this OS. Substitute literal absolute paths: plan in OS temp, projected/actual test module, and current source module. Scripts are temporary; nothing is installed. The gate independently inventories source handlers, then checks handler-case ownership, request names/maps, and the SOURCE-DERIVED scenario plan against actual tests, mocks, fixtures and imports. It cannot discover omitted business semantics or evaluate DataWeave predicates: the independent traversal audit remains mandatory. Never shrink the plan to make the gate pass.

### Windows PowerShell 5.1

```powershell
$ErrorActionPreference = 'Stop'
$planFile = 'ABSOLUTE_PLAN_JSON'
$testRoot = 'ABSOLUTE_TEST_MODULE'
$sourceRoot = 'ABSOLUTE_SOURCE_MODULE'
$c = 'http://www.mulesoft.org/schema/mule/core'
$m = 'http://www.mulesoft.org/schema/mule/munit'
$t = 'http://www.mulesoft.org/schema/mule/munit-tools'
$h = 'http://www.mulesoft.org/schema/mule/http'
$d = 'http://www.mulesoft.org/schema/mule/documentation'
function Need($ok, $why) { if (-not $ok) { throw $why } }
function PathIn($root, $rel) {
  Need ($rel -and $rel -notmatch '(^/|^[A-Za-z]:|\\|(^|/)\.\.(/|$))') 'Invalid relative path'
  return Join-Path $root $rel
}
function ReadJson($path) {
  $s = [IO.File]::ReadAllText($path)
  Need (-not [string]::IsNullOrWhiteSpace($s)) "Empty JSON: $path"
  return ,(ConvertFrom-Json -InputObject $s -ErrorAction Stop)
}
function XmlAt($path) {
  $raw = [IO.File]::ReadAllText($path)
  Need ($raw -notmatch '<!DOCTYPE') 'DOCTYPE forbidden'
  $opt = New-Object Xml.XmlReaderSettings; $opt.DtdProcessing = 'Prohibit'; $opt.XmlResolver = $null
  $reader = [Xml.XmlReader]::Create($path, $opt)
  $doc = New-Object Xml.XmlDocument; $doc.XmlResolver = $null
  try { $doc.Load($reader) } finally { $reader.Dispose() }
  return ,$doc
}
function Nodes($node, $local, $uri, $axis='./') {
  return @($node.SelectNodes("$axis*[local-name()='$local' and namespace-uri()='$uri']"))
}
function Attr($node, $key) { if ($key.StartsWith('doc:')) { return $node.GetAttribute($key.Substring(4), $d) }; return $node.GetAttribute($key) }
function Same($a,$b) { return ((@($a | Sort-Object -Unique) -join "`n") -ceq (@($b | Sort-Object -Unique) -join "`n")) }
function MockKey($processor,$where) { return $processor + '|' + (($where.PSObject.Properties | Sort-Object Name | ForEach-Object { $_.Name+'='+$_.Value }) -join '|') }
$plan = ReadJson $planFile
Need ($plan.version -eq 3 -and @($plan.suites).Count -gt 0 -and @($plan.required).Count -gt 0) 'Empty/invalid plan'
Need (@($plan.required | Sort-Object -Unique).Count -eq @($plan.required).Count) 'Duplicate obligation IDs'
$covered = @(); $suiteFiles = @(); $sourceCache = @{}
# Version 3: discover handlers from current XML before trusting suite rows.
Need (@('all','files','errors') -ccontains $plan.scope) 'Missing scope'
Need (@('http','non-http','mixed') -ccontains $plan.sourceKind) 'Missing sourceKind'
Need (@($plan.sourceRoots).Count -gt 0 -and $null -ne $plan.handlers -and $null -ne $plan.contracts -and @($plan.entryFlows).Count -gt 0 -and $null -ne $plan.publicFlows) 'Missing source/handler/contract/entry inventory'
$catalog = New-Object Collections.ArrayList; $globals = @{}; $sourceHandlers = New-Object Collections.ArrayList
$defaultRefs = New-Object Collections.ArrayList; $listenerNames = New-Object Collections.ArrayList
function ScanSource($rel) {
  $item = Get-Item -LiteralPath (PathIn $sourceRoot $rel) -Force
  Need (-not ($item.Attributes -band [IO.FileAttributes]::ReparsePoint)) "Source link needs explicit resolution: $rel"
  if ($item.PSIsContainer) { foreach ($child in (Get-ChildItem -LiteralPath $item.FullName -Force | Sort-Object Name)) { ScanSource ($rel+'/'+$child.Name) }; return }
  if ($item.Extension -ine '.xml') { return }
  Need (-not $sourceCache.ContainsKey($rel)) 'Overlapping source roots'
  $sourceDoc = XmlAt $item.FullName; $sourceCache[$rel] = $sourceDoc; $null = $catalog.Add($sourceDoc); $ordinal = 0
  foreach ($node in $sourceDoc.SelectNodes('//*')) {
    $local = $node.LocalName; $parent = $node.ParentNode
    if ($parent.LocalName -ceq 'mule' -and $parent.NamespaceURI -ceq $c -and $node.NamespaceURI -ceq $c -and @('flow','sub-flow','error-handler','on-error-continue','on-error-propagate') -ccontains $local -and $node.GetAttribute('name')) {
      $name = $node.GetAttribute('name'); Need (-not $globals.ContainsKey($name)) "Ambiguous named source: $name"; $globals[$name] = $node
    }
    if ($local -ceq 'flow' -and $node.NamespaceURI -ceq $c -and @(Nodes $node 'listener' $h).Count -gt 0) { $null = $listenerNames.Add($node.GetAttribute('name')) }
    if ($local -ceq 'configuration' -and $node.NamespaceURI -ceq $c -and $node.GetAttribute('defaultErrorHandler-ref')) { $null = $defaultRefs.Add($node.GetAttribute('defaultErrorHandler-ref')) }
    if ($node.NamespaceURI -ceq $c -and $local -cmatch '^on-error-(continue|propagate)$' -and -not $node.GetAttribute('ref')) {
      $ordinal++; $isLocal = $false; $ancestor = $node.ParentNode
      while ($null -ne $ancestor) { if ($ancestor.LocalName -ceq 'try' -and $ancestor.NamespaceURI -ceq $c) { $isLocal = $true }; $ancestor = $ancestor.ParentNode }
      $null = $sourceHandlers.Add([pscustomobject]@{source=$rel;ordinal=$ordinal;node=$node;local=$isLocal})
    }
  }
}
foreach ($rel in $plan.sourceRoots) { ScanSource $rel }
foreach ($name in (@($plan.entryFlows)+@($plan.publicFlows))) { Need ($globals.ContainsKey($name) -and @('flow','sub-flow') -ccontains $globals[$name].LocalName) "Missing entry/public flow: $name" }
if ($plan.scope -cne 'files' -and $plan.sourceKind -cne 'non-http') { Need (Same @($listenerNames) @($plan.entryFlows | Where-Object { $listenerNames -ccontains $_ })) 'HTTP listener omitted from all/errors plan' }
$visited = New-Object Collections.ArrayList; $reachable = New-Object Collections.ArrayList
function WalkRef($name) { Need ($name -and $name -notmatch '#\[' -and $globals.ContainsKey($name)) "Unresolved source reference: $name"; WalkNode $globals[$name] }
function WalkNode($node) {
  if ($visited.Contains($node)) { return }; $null = $visited.Add($node)
  foreach ($record in $sourceHandlers) { if ([object]::ReferenceEquals($record.node,$node)) { $null = $reachable.Add($record) } }
  if ($node.LocalName -ceq 'flow-ref' -and $node.NamespaceURI -ceq $c) { WalkRef $node.GetAttribute('name') }
  if ($node.NamespaceURI -ceq $c -and @('error-handler','on-error','on-error-continue','on-error-propagate') -ccontains $node.LocalName -and $node.GetAttribute('ref')) { WalkRef $node.GetAttribute('ref') }
  if ($node.LocalName -ceq 'flow' -and $node.NamespaceURI -ceq $c -and @(Nodes $node 'error-handler' $c).Count -eq 0) { foreach ($refName in $defaultRefs) { WalkRef $refName } }
  foreach ($child in $node.SelectNodes('./*')) { WalkNode $child }
}
foreach ($name in (@($plan.entryFlows)+@($plan.publicFlows))) { WalkRef $name }
$discovered = @($sourceHandlers); if ($plan.scope -ceq 'files') { $discovered = @($reachable) }
function HandlerKey($record) { return $record.source+'#on-error/'+$record.ordinal }
Need (@($plan.handlers).Count -eq $discovered.Count -and (Same @($plan.handlers | ForEach-Object { HandlerKey $_ }) @($discovered | ForEach-Object { HandlerKey $_ }))) 'Source handler missing/extra in plan'
$allCases = @{}; $testRows = @()
foreach ($suite in $plan.suites) { foreach ($testCase in $suite.tests) { $testRows += [pscustomobject]@{suite=$suite.file;test=$testCase} } }
foreach ($row in $testRows) { Need (@('A','B','C','N','D') -ccontains $row.test.mode -and $null -ne $row.test.handlerCases) 'Missing test mode/handlerCases' }
foreach ($record in $plan.handlers) {
  $key = HandlerKey $record; $foundHandler = @($discovered | Where-Object { (HandlerKey $_) -ceq $key })[0]
  $typeText = $foundHandler.node.GetAttribute('type'); if (-not $typeText) { $typeText = 'ANY' }; $types = @($typeText.Split(',') | ForEach-Object { $_.Trim() })
  Need ((Same @($record.types) $types) -and $record.when -ceq $foundHandler.node.GetAttribute('when')) "Stale handler matcher: $key"
  Need (@('apikit','main','local') -ccontains $record.role -and (($foundHandler.local -and $record.role -ceq 'local') -or (-not $foundHandler.local -and $record.role -cne 'local'))) "Invalid handler role: $key"
  if (-not $foundHandler.local -and @($types | Where-Object { $_.StartsWith('APIKIT:') }).Count -gt 0) { Need ($record.role -ceq 'apikit') 'APIKit handler assigned outside APIKit suite' }
  Need ($null -ne $record.cases -and (Same @($record.cases | ForEach-Object { $_.type }) $types)) "Missing handler matcher case: $key"
  foreach ($handlerCase in $record.cases) {
    Need ($handlerCase.id -and -not $allCases.ContainsKey($handlerCase.id) -and $plan.required -ccontains $handlerCase.id -and $handlerCase.trigger -and $handlerCase.trigger -cne 'ANY') 'Invalid/duplicate/unrequired handler case'
    $allCases[$handlerCase.id] = $handlerCase
    $owners = @($testRows | Where-Object { $_.test.handlerCases -ccontains $handlerCase.id }); Need ($owners.Count -gt 0) "Handler case has no test: $($handlerCase.id)"
    foreach ($owner in $owners) {
      $caseSpec = $owner.test; $expected = $null
      if ($record.role -ceq 'apikit') { $expected = 'apikit-error-test-suite.xml' }; if ($record.role -ceq 'main') { $expected = 'error-test-suite.xml' }
      Need (-not $expected -or $owner.suite -ceq ('src/test/munit/'+$expected)) 'Handler in wrong/missing owning suite'
      Need ($caseSpec.covers -ccontains $handlerCase.id) 'Handler case absent from test coverage'
      $routerMocks = @($caseSpec.mocks | Where-Object {
        $mockSpec = $_; $sourceDoc = $sourceCache[$mockSpec.source]
        $null -ne $sourceDoc -and @(Nodes $sourceDoc 'router' 'http://www.mulesoft.org/schema/mule/mule-apikit' '//' | Where-Object { $_.Name -ceq $mockSpec.processor }).Count -gt 0
      })
      if ($record.role -ceq 'apikit') { Need ($caseSpec.mode -ceq 'B' -and @($routerMocks | Where-Object { $_.errors -ccontains $handlerCase.trigger }).Count -gt 0) 'APIKit case missing router error injection' }
      if ($record.role -ceq 'main') { Need (@('C','N','D') -ccontains $caseSpec.mode -and $routerMocks.Count -eq 0) 'Main handler uses wrong mode/router shortcut' }
    }
  }
}
foreach ($row in $testRows) { foreach ($id in $row.test.handlerCases) { Need ($allCases.ContainsKey($id)) 'Unknown handlerCases ID' }; if (@('B','C') -ccontains $row.test.mode) { Need (@($row.test.handlerCases).Count -gt 0) 'Error test missing handler case' } }
Need (@($plan.contracts | ForEach-Object { $_.key } | Sort-Object -Unique).Count -eq @($plan.contracts).Count) 'Duplicate effective contract'
foreach ($contract in $plan.contracts) {
  Need (@('resolved','source-backed') -ccontains $contract.status -and $null -ne $contract.missingReferences) 'Missing explicit contract status'
  if ($contract.status -ceq 'resolved') { Need (@($contract.missingReferences).Count -eq 0) 'Resolved contract retains missing references'; continue }
  Need (@($contract.missingReferences).Count -gt 0 -and @($contract.sourceEvidence).Count -gt 0) 'Source-backed contract missing provenance'
  foreach ($e in $contract.sourceEvidence) {
    Need ($e.source -and -not [string]::IsNullOrWhiteSpace($e.contains)) 'Missing source evidence locator'
    Need ([IO.File]::ReadAllText((PathIn $sourceRoot $e.source)).Contains($e.contains)) 'Source-backed evidence does not match current file'
  }
}

function CheckRequest($caseSpec,$execution) {
  $requests = @(Nodes $execution 'request' $h './/'); $q = $caseSpec.request
  if (@('N','D') -ccontains $caseSpec.mode) { Need ($null -eq $q -and $requests.Count -eq 0) 'Direct test contains unexpected HTTP execution request'; return }
  Need ($null -ne $q -and $requests.Count -eq 1 -and $null -ne $q.headers) 'HTTP request missing/duplicate/unplanned'; $requestNode = $requests[0]
  Need ($requestNode.GetAttribute('method') -ceq $q.method -and $requestNode.GetAttribute('path') -ceq $q.path -and $requestNode.GetAttribute('config-ref') -ceq $q.configRef) 'Request differs from planned operation'
  Need ((Attr $requestNode 'doc:name') -ceq ('Request to '+$q.path)) 'Request display name must be Request to plus its path'
  $contract = @($plan.contracts | Where-Object { $_.key -ceq $q.contract }); Need ($contract.Count -eq 1 -and @($contract[0].evidence).Count -gt 0) 'Missing effective request contract'
  $headerNames = @(); $lines = @()
  foreach ($header in $q.headers) {
    Need ($header.name -and $header.expression -and $header.evidence) 'Header missing name/value/evidence'
    $headerNames += $header.name.ToLowerInvariant()
    $lines += '  '+(ConvertTo-Json -InputObject ([string]$header.name) -Compress)+': ('+$header.expression+')'
  }
  Need (@($headerNames | Sort-Object -Unique).Count -eq $headerNames.Count) 'Duplicate header name'
  foreach ($name in $contract[0].requiredHeaders) { Need ($headerNames -ccontains $name.ToLowerInvariant()) "Required trait header omitted: $name" }
  $map = '{}'; if ($lines.Count -gt 0) { $map = "{`n"+($lines -join ",`n")+"`n}" }
  $expected = "#[output application/java`n---`n"+$map+']'; $headerNodes = @(Nodes $requestNode 'headers' $h)
  Need ($headerNodes.Count -eq 1 -and $headerNodes[0].InnerText.Replace("`r`n","`n").Trim() -ceq $expected) 'Missing/wrong http:headers map'
  if (@('B','C') -ccontains $caseSpec.mode) {
    $children = @($requestNode.SelectNodes('./*')); $last = $children[-1]
    Need ($last.LocalName -ceq 'response-validator' -and $last.NamespaceURI -ceq $h) 'Error response-validator must be last'
    $validators = @(Nodes $last 'success-status-code-validator' $h)
    Need ($validators.Count -eq 1 -and $validators[0].GetAttribute('values') -ceq '200..599') 'Wrong error status validator'
  }
}

foreach ($s in $plan.suites) {
  Need (-not ($suiteFiles -ccontains $s.file)) 'Duplicate suite ownership'; $suiteFiles += $s.file
  Need ($s.file -match '^src/test/munit/.+-test-suite\.xml$' -and @($s.tests).Count -gt 0) 'Invalid/empty suite plan'
  $path = PathIn $testRoot $s.file; $bytes = [IO.File]::ReadAllBytes($path)
  Need ($bytes.Length -gt 5 -and [Text.Encoding]::ASCII.GetString($bytes,0,5) -ceq '<?xml') 'XML start/BOM'
  $doc = XmlAt $path; $rootNode = $doc.DocumentElement
  Need ($rootNode.LocalName -ceq 'mule' -and $rootNode.NamespaceURI -ceq $c) 'Invalid Mule root'
  Need (@(Nodes $rootNode 'import' $c | Where-Object { $_.GetAttribute('file') -ceq 'test-config.xml' }).Count -eq 1) 'Missing/duplicate import'
  Need (@(Nodes $rootNode 'config' $m).Count -eq 1 -and @(Nodes $rootNode 'request-config' $h './/').Count -eq 0) 'Suite config/inline client'
  $sourceBackedKeys = @($plan.contracts | Where-Object { $_.status -ceq 'source-backed' } | ForEach-Object { $_.key })
  if (@($s.tests | Where-Object { $null -ne $_.request -and $sourceBackedKeys -ccontains $_.request.contract }).Count -gt 0) {
    Need (@($rootNode.SelectNodes('./comment()') | Where-Object { $_.Value.Trim() -ceq 'RAML-CONTRACT: source-backed; required-header completeness is unverified.' }).Count -eq 1) 'Missing source-backed suite marker'
  }
  $actualTests = @(Nodes $rootNode 'test' $m)
  $actualNames = @($actualTests | ForEach-Object { $_.GetAttribute('name') })
  $expectedNames = @($s.tests | ForEach-Object { $_.name }) + @($s.preserveTests)
  Need ($actualNames.Count -eq $expectedNames.Count -and (Same $actualNames $expectedNames) -and @($actualNames | Sort-Object -Unique).Count -eq $actualNames.Count) "Missing/extra/duplicate test: $($s.file)"
  foreach ($case in $s.tests) {
    $test = @($actualTests | Where-Object { $_.GetAttribute('name') -ceq $case.name })[0]
    $prefix = [IO.Path]::GetFileNameWithoutExtension($s.file) + '-'
    Need ($case.name.StartsWith($prefix) -and $case.name -notmatch 'happy-path|error-path|default-choice' -and @($case.covers).Count -gt 0) 'Invalid scenario'
    foreach ($id in $case.covers) { Need ($plan.required -ccontains $id) "Unknown obligation: $id"; $covered += $id }
    foreach ($group in $plan.exclusiveGroups) { Need (@($case.covers | Where-Object { $group -ccontains $_ }).Count -le 1) "One test claims exclusive branches: $($case.name)" }
    $inputRefs = @(); $execution = @(Nodes $test 'execution' $m); Need ($execution.Count -eq 1) 'Missing execution'
    CheckRequest $case $execution[0]
    foreach ($a in $execution[0].SelectNodes('.//@value')) {
      foreach ($hit in [regex]::Matches($a.Value,"MunitTools::getResourceAsString\('(?<r>in/[^'`"<>\r\n]+\.json)'\)")) { $inputRefs += $hit.Groups['r'].Value }
    }
    Need (Same $inputRefs @($case.inputs | ForEach-Object { $_.resource })) "Wrong input fixtures: $($case.name)"
    foreach ($inputSpec in $case.inputs) {
      $inputData = ReadJson (PathIn (Join-Path $testRoot 'src/test/resources') $inputSpec.resource)
      foreach ($check in $inputSpec.checks) {
        $value = $inputData; Need ($check.pointer -eq '' -or $check.pointer.StartsWith('/')) 'Invalid JSON pointer'
        if ($check.pointer -ne '') { foreach ($part in $check.pointer.Substring(1).Split('/')) {
          $part = $part.Replace('~1','/').Replace('~0','~'); Need ($null -ne $value) 'Missing discriminator'
          if ($value -is [array]) { Need ($part -match '^(0|[1-9][0-9]*)$' -and [int]$part -lt $value.Count) 'Missing array item'; $value = $value[[int]$part] }
          else { $property = @($value.PSObject.Properties | Where-Object { $_.Name -ceq $part }); Need ($property.Count -eq 1) 'Missing discriminator'; $value = $property[0].Value }
        } }
        Need ((ConvertTo-Json -InputObject $value -Compress -Depth 100) -ceq (ConvertTo-Json -InputObject $check.value -Compress -Depth 100)) "Wrong input discriminator: $($check.pointer)"
      }
    }
    $actualMocks = @(Nodes $test 'mock-when' $t './/'); $expectedKeys = @($case.mocks | ForEach-Object { MockKey $_.processor $_.where })
    Need ($actualMocks.Count -eq @($case.mocks).Count -and @($expectedKeys | Sort-Object -Unique).Count -eq $expectedKeys.Count) "Mock count/duplicate plan: $($case.name)"
    foreach ($needMock in $case.mocks) {
      $key = MockKey $needMock.processor $needMock.where
      $found = @($actualMocks | Where-Object {
        $w = [ordered]@{}; foreach ($a in (Nodes $_ 'with-attribute' $t './/')) { $w[$a.GetAttribute('attributeName')] = $a.GetAttribute('whereValue') }
        (MockKey $_.GetAttribute('processor') ([pscustomobject]$w)) -ceq $key
      })
      Need ($found.Count -eq 1) "Missing/wrong mock: $key"
      $mock = $found[0]; Need ($mock.ParentNode.LocalName -ceq 'behavior' -and $mock.ParentNode.NamespaceURI -ceq $m) 'Mock outside behavior'
      Need ($needMock.processor -notmatch '(^|:)(flow-ref|logger|transform|choice|scatter-gather|foreach|parallel-foreach|job)$') 'Mock hides processing'
      if (-not $sourceCache.ContainsKey($needMock.source)) { $sourceCache[$needMock.source] = XmlAt (PathIn $sourceRoot $needMock.source) }
      $hits = @($sourceCache[$needMock.source].SelectNodes('//*') | Where-Object {
        $node = $_; $ok = $node.Name -ceq $needMock.processor
        foreach ($p in $needMock.where.PSObject.Properties) { $ok = $ok -and ((Attr $node $p.Name) -ceq [string]$p.Value) }; $ok
      }); Need ($hits.Count -eq 1 -and @($needMock.where.PSObject.Properties).Count -gt 0) "Untraced source mock: $key"
      $returns = @(Nodes $mock 'then-return' $t); Need ($returns.Count -eq 1) 'Missing/duplicate return'
      $resources = @()
      foreach ($a in $returns[0].SelectNodes('.//@value')) {
        $matchesInValue = [regex]::Matches($a.Value, "MunitTools::getResourceAsString\('(?<r>out/[^'`"<>\r\n]+\.json)'\)")
        if ($a.OwnerElement.LocalName -ceq 'payload' -and $a.OwnerElement.NamespaceURI -ceq $t) { Need ($matchesInValue.Count -eq 1) 'Mock payload is not file-backed' }
        foreach ($matchItem in $matchesInValue) { $rel = $matchItem.Groups['r'].Value; $null = ReadJson (PathIn (Join-Path $testRoot 'src/test/resources') $rel); $resources += $rel }
      }
      Need (Same $resources @($needMock.resources)) "Wrong/missing return fixture: $key"
      $errors = @(Nodes $returns[0] 'error' $t | ForEach-Object { $_.GetAttribute('typeId') })
      Need (Same $errors @($needMock.errors)) "Wrong/missing error: $key"
    }
  }
  Write-Output "Scenario/mock gate passed: $($s.file) ($(@($s.tests).Count) required tests)"
}
Need (Same $covered @($plan.required)) 'Required branch/processor/handler obligation missing'
if (@($plan.contracts | Where-Object { $_.status -ceq 'source-backed' }).Count -gt 0) { Write-Output 'Source-backed generation checks passed; RAML unverified. Runtime is a separate result.' }
```

### macOS system JavaScript for Automation

This is the macOS built-in JavaScript runner, not Node. Pass paths as literal arguments; no helper download.

```bash
/usr/bin/osascript -l JavaScript - 'ABSOLUTE_PLAN_JSON' 'ABSOLUTE_TEST_MODULE' 'ABSOLUTE_SOURCE_MODULE' <<'JXA'
ObjC.import('Foundation');
function run(argv) {
  var C='http://www.mulesoft.org/schema/mule/core', M='http://www.mulesoft.org/schema/mule/munit', T='http://www.mulesoft.org/schema/mule/munit-tools', H='http://www.mulesoft.org/schema/mule/http', D='http://www.mulesoft.org/schema/mule/documentation';
  function need(ok,msg) { if (!ok) throw Error(msg); }
  function path(root,rel) { need(typeof rel==='string' && rel && !/(^\/|^[A-Za-z]:|\\|(^|\/)\.\.(\/|$))/.test(rel),'Invalid relative path'); return root+'/'+rel; }
  function read(p) { var s=$.NSString.stringWithContentsOfFileEncodingError(p,$.NSUTF8StringEncoding,null); need(Boolean(s),'Unreadable file: '+p); return ObjC.unwrap(s); }
  function xml(p) { var s=read(p); need(!/<!DOCTYPE/i.test(s),'DOCTYPE forbidden'); var d=$.NSXMLDocument.alloc.initWithXMLStringOptionsError(s,0,null); need(Boolean(d),'Malformed XML: '+p); return d; }
  function elements(n,deep) { var out=[]; for(var i=0;i<Number(n.childCount);i++){var a=n.childAtIndex(i);if(Number(a.kind)===2){out.push(a);if(deep)out=out.concat(elements(a,true));}}return out; }
  function attrs(n) { var a=n.attributes,out=[];for(var i=0;a && i<Number(a.count);i++)out.push(a.objectAtIndex(i));return out; }
  function xp(n,q) {
    if(q==='./@*')return attrs(n);
    if(q==='parent::*')return n.parent && Number(n.parent.kind)===2?[n.parent]:[];
    if(q==='.//@value')return [n].concat(elements(n,true)).reduce(function(out,a){return out.concat(attrs(a).filter(function(v){return ObjC.unwrap(v.name)==='value';}));},[]);
    need(['//*','.//*','./*','/*'].indexOf(q)>=0,'Unsupported traversal');return elements(n,q==='//*'||q==='.//*');
  }
  function is(n,local,uri) { return ObjC.unwrap(n.localName)===local && ObjC.unwrap(n.URI)===uri; }
  function nodes(n,local,uri,axis) { return xp(n,(axis||'./')+'*').filter(function(a){return is(a,local,uri);}); }
  function attr(n,k) { var a=n.attributeForName(k), v=ObjC.unwrap(a.stringValue); return typeof v==='string'?v:''; }
  function sourceAttr(n,k) { if(k.indexOf('doc:')!==0) return attr(n,k); var a=xp(n,'./@*').filter(function(a){return is(a,k.slice(4),D);}); return a.length?ObjC.unwrap(a[0].stringValue):''; }
  function uniq(a) { return a.filter(function(x,i){return a.indexOf(x)===i;}).sort(); }
  function same(a,b) { return JSON.stringify(uniq(a))===JSON.stringify(uniq(b)); }
  function key(p,w) { return p+'|'+Object.keys(w).sort().map(function(k){return k+'='+w[k];}).join('|'); }
  var plan=JSON.parse(read(argv[0])), covered=[], suiteFiles=[], cache={};
  need(plan.version===3 && plan.suites.length>0 && plan.required.length>0,'Empty/invalid plan');
  need(uniq(plan.required).length===plan.required.length,'Duplicate obligation IDs');
  // Version 3: discover handlers from current XML before trusting suite rows.
  need(['all','files','errors'].indexOf(plan.scope)>=0 && ['http','non-http','mixed'].indexOf(plan.sourceKind)>=0,'Missing scope/sourceKind');
  need(Array.isArray(plan.sourceRoots) && plan.sourceRoots.length>0 && Array.isArray(plan.handlers) && Array.isArray(plan.contracts),'Missing source/handler/contract inventory');
  need(Array.isArray(plan.entryFlows) && plan.entryFlows.length>0 && Array.isArray(plan.publicFlows),'Missing entry/public inventory');
  var fm=$.NSFileManager.defaultManager, catalog=[], globals={}, sourceHandlers=[], defaultRefs=[], listenerNames=[];
  function scan(rel) {
    var abs=path(argv[2],rel), info=fm.attributesOfItemAtPathError(abs,null); need(Boolean(info),'Missing source path: '+rel);
    var kind=ObjC.unwrap(info.objectForKey($.NSFileType)); need(kind!=='NSFileTypeSymbolicLink','Source symlink needs explicit resolution: '+rel);
    if(kind==='NSFileTypeDirectory') {
      var entries=fm.contentsOfDirectoryAtPathError(abs,null); need(Boolean(entries),'Unreadable source directory');
      ObjC.deepUnwrap(entries).sort().forEach(function(n){scan(rel+'/'+n);});
    } else if(/\.xml$/i.test(rel)) {
      need(!catalog.some(function(f){return f.file===rel;}),'Overlapping source roots');
      var doc=xml(abs), all=elements(doc,true), ordinal=0; cache[rel]=doc; catalog.push({file:rel,doc:doc});
      all.forEach(function(n){
        var local=ObjC.unwrap(n.localName), parent=n.parent;
        if(is(parent,'mule',C) && ['flow','sub-flow','error-handler','on-error-continue','on-error-propagate'].indexOf(local)>=0 && ObjC.unwrap(n.URI)===C && attr(n,'name')) {
          var name=attr(n,'name'); need(!globals[name],'Ambiguous named source: '+name); globals[name]=n;
        }
        if(is(n,'flow',C) && nodes(n,'listener',H).length) listenerNames.push(attr(n,'name'));
        if(is(n,'configuration',C) && attr(n,'defaultErrorHandler-ref')) defaultRefs.push(attr(n,'defaultErrorHandler-ref'));
        if(ObjC.unwrap(n.URI)===C && /^on-error-(continue|propagate)$/.test(local) && !attr(n,'ref')) {
          ordinal++; var localHandler=false, a=n.parent;
          while(a && Number(a.kind)===2){if(is(a,'try',C))localHandler=true;a=a.parent;}
          sourceHandlers.push({source:rel,ordinal:ordinal,node:n,local:localHandler});
        }
      });
    }
  }
  plan.sourceRoots.forEach(scan);
  plan.entryFlows.concat(plan.publicFlows).forEach(function(name){need(globals[name] && (is(globals[name],'flow',C)||is(globals[name],'sub-flow',C)),'Missing entry/public flow: '+name);});
  if(plan.scope!=='files' && plan.sourceKind!=='non-http') need(same(listenerNames,plan.entryFlows.filter(function(n){return listenerNames.indexOf(n)>=0;})),'HTTP listener omitted from all/errors plan');
  var visited=[], reachable=[];
  function ref(name){need(name && name.indexOf('#[')<0 && globals[name],'Unresolved source reference: '+name);walk(globals[name]);}
  function walk(n){
    if(visited.some(function(v){return Boolean(n.isEqual(v));}))return;visited.push(n);
    sourceHandlers.forEach(function(h){if(Boolean(h.node.isEqual(n)))reachable.push(h);});
    if(is(n,'flow-ref',C))ref(attr(n,'name'));
    if(ObjC.unwrap(n.URI)===C && /^(error-handler|on-error|on-error-continue|on-error-propagate)$/.test(ObjC.unwrap(n.localName)) && attr(n,'ref'))ref(attr(n,'ref'));
    if(is(n,'flow',C) && !nodes(n,'error-handler',C).length)defaultRefs.forEach(ref);
    elements(n,false).forEach(walk);
  }
  plan.entryFlows.concat(plan.publicFlows).forEach(ref);
  var discovered=plan.scope==='files'?reachable:sourceHandlers;
  function hk(h){return h.source+'#on-error/'+h.ordinal;}
  need(plan.handlers.length===discovered.length && same(plan.handlers.map(hk),discovered.map(hk)),'Source handler missing/extra in plan');
  var allCases={}, testRows=[];
  plan.suites.forEach(function(s){s.tests.forEach(function(t){testRows.push({suite:s.file,test:t});});});
  testRows.forEach(function(row){need(['A','B','C','N','D'].indexOf(row.test.mode)>=0 && Array.isArray(row.test.handlerCases),'Missing test mode/handlerCases');});
  plan.handlers.forEach(function(h){
    var source=discovered.filter(function(x){return hk(x)===hk(h);})[0], types=(attr(source.node,'type')||'ANY').split(',').map(function(t){return t.trim();});
    need(same(h.types,types) && h.when===attr(source.node,'when'),'Stale handler matcher: '+hk(h));
    need(['apikit','main','local'].indexOf(h.role)>=0 && (source.local?h.role==='local':h.role!=='local'),'Invalid handler role: '+hk(h));
    if(!source.local && types.some(function(t){return t.indexOf('APIKIT:')===0;}))need(h.role==='apikit','APIKit handler assigned outside APIKit suite');
    need(Array.isArray(h.cases) && same(h.cases.map(function(c){return c.type;}),types),'Missing handler matcher case: '+hk(h));
    h.cases.forEach(function(c){
      need(c.id && !allCases[c.id] && plan.required.indexOf(c.id)>=0 && c.trigger && c.trigger!=='ANY','Invalid/duplicate/unrequired handler case'); allCases[c.id]=c;
      var owners=testRows.filter(function(row){return row.test.handlerCases.indexOf(c.id)>=0;});need(owners.length>0,'Handler case has no test: '+c.id);
      owners.forEach(function(row){
        var t=row.test, expected=h.role==='apikit'?'apikit-error-test-suite.xml':h.role==='main'?'error-test-suite.xml':null;
        need(!expected||row.suite==='src/test/munit/'+expected,'Handler in wrong/missing owning suite: '+c.id);
        need(t.covers.indexOf(c.id)>=0,'Handler case absent from test coverage');
        var routers=t.mocks.filter(function(m){var doc=cache[m.source];return doc && elements(doc,true).some(function(n){return is(n,'router','http://www.mulesoft.org/schema/mule/mule-apikit') && ObjC.unwrap(n.name)===m.processor;});});
        if(h.role==='apikit')need(t.mode==='B' && routers.some(function(m){return m.errors.indexOf(c.trigger)>=0;}),'APIKit case missing router error injection');
        if(h.role==='main')need(['C','N','D'].indexOf(t.mode)>=0 && routers.length===0,'Main handler uses wrong mode/router shortcut');
      });
    });
  });
  testRows.forEach(function(row){row.test.handlerCases.forEach(function(id){need(allCases[id],'Unknown handlerCases ID');});if(['B','C'].indexOf(row.test.mode)>=0)need(row.test.handlerCases.length>0,'Error test missing handler case');});
  need(uniq(plan.contracts.map(function(c){return c.key;})).length===plan.contracts.length,'Duplicate effective contract');
  plan.contracts.forEach(function(c){
    need(['resolved','source-backed'].indexOf(c.status)>=0 && Array.isArray(c.missingReferences),'Missing explicit contract status');
    if(c.status==='resolved'){need(c.missingReferences.length===0,'Resolved contract retains missing references');return;}
    need(c.missingReferences.length>0 && Array.isArray(c.sourceEvidence) && c.sourceEvidence.length>0,'Source-backed contract missing provenance');
    c.sourceEvidence.forEach(function(e){need(e.source && typeof e.contains==='string' && e.contains.trim().length>0,'Missing source evidence locator');need(read(path(argv[2],e.source)).indexOf(e.contains)>=0,'Source-backed evidence does not match current file');});
  });

  function requestCheck(test,execution) {
    var requests=nodes(execution,'request',H,'.//'), q=test.request;
    if(['N','D'].indexOf(test.mode)>=0){need(q===null && requests.length===0,'Direct test contains unexpected HTTP execution request');return;}
    need(q && requests.length===1 && Array.isArray(q.headers),'HTTP request missing/duplicate/unplanned');var n=requests[0];
    need(attr(n,'method')===q.method && attr(n,'path')===q.path && attr(n,'config-ref')===q.configRef,'Request differs from planned operation');
    need(sourceAttr(n,'doc:name')==='Request to '+q.path,'Request display name must be Request to '+q.path);
    var cs=plan.contracts.filter(function(c){return c.key===q.contract;});need(cs.length===1 && cs[0].evidence.length>0,'Missing effective request contract');
    var names=q.headers.map(function(h){need(h.name && h.expression && h.evidence,'Header missing name/value/evidence');return h.name.toLowerCase();});
    need(uniq(names).length===names.length && cs[0].requiredHeaders.every(function(h){return names.indexOf(h.toLowerCase())>=0;}),'Required trait header omitted/duplicate');
    var body=q.headers.length?'{\n'+q.headers.map(function(h){return '  '+JSON.stringify(h.name)+': ('+h.expression+')';}).join(',\n')+'\n}':'{}';
    var expected='#[output application/java\n---\n'+body+']', headers=nodes(n,'headers',H);
    need(headers.length===1 && ObjC.unwrap(headers[0].stringValue).replace(/\r\n/g,'\n').trim()===expected,'Missing/wrong http:headers map');
    if(['B','C'].indexOf(test.mode)>=0){var kids=elements(n,false), last=kids[kids.length-1];need(is(last,'response-validator',H),'Error response-validator must be last');var validators=nodes(last,'success-status-code-validator',H);need(validators.length===1 && attr(validators[0],'values')==='200..599','Wrong error status validator');}
  }

  plan.suites.forEach(function(s) {
    need(suiteFiles.indexOf(s.file)<0 && /^src\/test\/munit\/.+-test-suite\.xml$/.test(s.file) && s.tests.length>0,'Invalid/duplicate/empty suite plan'); suiteFiles.push(s.file);
    var p=path(argv[1],s.file); need(read(p).slice(0,5)==='<?xml','XML start/BOM');
    var doc=xml(p), roots=nodes(doc,'mule',C,'/'); need(roots.length===1,'Invalid Mule root'); var r=roots[0];
    need(nodes(r,'import',C).filter(function(n){return attr(n,'file')==='test-config.xml';}).length===1,'Missing/duplicate import');
    need(nodes(r,'config',M).length===1 && nodes(r,'request-config',H,'.//').length===0,'Suite config/inline client');
    var sourceBackedKeys=plan.contracts.filter(function(c){return c.status==='source-backed';}).map(function(c){return c.key;});
    if(s.tests.some(function(t){return t.request && sourceBackedKeys.indexOf(t.request.contract)>=0;})){
      var markers=0;for(var ci=0;ci<Number(r.childCount);ci++){var child=r.childAtIndex(ci);if(Number(child.kind)===6 && ObjC.unwrap(child.stringValue).trim()==='RAML-CONTRACT: source-backed; required-header completeness is unverified.')markers++;}
      need(markers===1,'Missing source-backed suite marker');
    }
    var actual=nodes(r,'test',M), names=actual.map(function(n){return attr(n,'name');}), wanted=s.tests.map(function(t){return t.name;}).concat(s.preserveTests);
    need(names.length===wanted.length && same(names,wanted) && uniq(names).length===names.length,'Missing/extra/duplicate test: '+s.file);
    s.tests.forEach(function(test) {
      need(test.name.indexOf(s.file.split('/').pop().slice(0,-4)+'-')===0 && !/happy-path|error-path|default-choice/.test(test.name) && test.covers.length>0,'Invalid scenario');
      test.covers.forEach(function(id){need(plan.required.indexOf(id)>=0,'Unknown obligation: '+id);covered.push(id);});
      plan.exclusiveGroups.forEach(function(g){need(test.covers.filter(function(id){return g.indexOf(id)>=0;}).length<=1,'One test claims exclusive branches: '+test.name);});
      var node=actual.filter(function(n){return attr(n,'name')===test.name;})[0], mocks=nodes(node,'mock-when',T,'.//');
      var execution=nodes(node,'execution',M),inputRefs=[];need(execution.length===1,'Missing execution');requestCheck(test,execution[0]);
      xp(execution[0],'.//@value').forEach(function(a){var re=/MunitTools::getResourceAsString\('(in\/[^'"<>\r\n]+\.json)'\)/g,hit;while((hit=re.exec(ObjC.unwrap(a.stringValue)))!==null)inputRefs.push(hit[1]);});
      need(same(inputRefs,test.inputs.map(function(i){return i.resource;})),'Wrong input fixtures: '+test.name);
      test.inputs.forEach(function(i){var data=JSON.parse(read(path(argv[1]+'/src/test/resources',i.resource)));i.checks.forEach(function(check){
        need(check.pointer===''||check.pointer.charAt(0)==='/','Invalid JSON pointer');var value=data;
        (check.pointer===''?[]:check.pointer.slice(1).split('/')).forEach(function(k){k=k.replace(/~1/g,'/').replace(/~0/g,'~');need(value!==null && typeof value==='object' && Object.prototype.hasOwnProperty.call(value,k),'Missing discriminator');value=value[k];});
        need(JSON.stringify(value)===JSON.stringify(check.value),'Wrong input discriminator: '+check.pointer);
      });});
      var expectedKeys=test.mocks.map(function(m){return key(m.processor,m.where);});
      need(mocks.length===test.mocks.length && uniq(expectedKeys).length===expectedKeys.length,'Mock count/duplicate plan: '+test.name);
      test.mocks.forEach(function(em) {
        var k=key(em.processor,em.where), found=mocks.filter(function(n){var w={};nodes(n,'with-attribute',T,'.//').forEach(function(a){w[attr(a,'attributeName')]=attr(a,'whereValue');});return key(attr(n,'processor'),w)===k;});
        need(found.length===1,'Missing/wrong mock: '+k); var mock=found[0];
        need(xp(mock,'parent::*').filter(function(a){return is(a,'behavior',M);}).length===1,'Mock outside behavior');
        need(!/(^|:)(flow-ref|logger|transform|choice|scatter-gather|foreach|parallel-foreach|job)$/.test(em.processor),'Mock hides processing');
        if(!cache[em.source]) cache[em.source]=xml(path(argv[2],em.source));
        var hits=xp(cache[em.source],'//*').filter(function(n){return ObjC.unwrap(n.name)===em.processor && Object.keys(em.where).every(function(k){return sourceAttr(n,k)===String(em.where[k]);});});
        need(hits.length===1 && Object.keys(em.where).length>0,'Untraced source mock: '+k);
        var returns=nodes(mock,'then-return',T); need(returns.length===1,'Missing/duplicate return'); var resources=[];
        xp(returns[0],'.//@value').forEach(function(a){
          var v=ObjC.unwrap(a.stringValue), re=/MunitTools::getResourceAsString\('(out\/[^'"<>\r\n]+\.json)'\)/g, hit, local=[];
          while((hit=re.exec(v))!==null){local.push(hit[1]);JSON.parse(read(path(argv[1]+'/src/test/resources',hit[1])));resources.push(hit[1]);}
          if(xp(a,'parent::*').some(function(n){return is(n,'payload',T);})) need(local.length===1,'Mock payload is not file-backed');
        });
        need(same(resources,em.resources),'Wrong/missing return fixture: '+k);
        need(same(nodes(returns[0],'error',T).map(function(n){return attr(n,'typeId');}),em.errors),'Wrong/missing error: '+k);
      });
    });
  });
  need(same(covered,plan.required),'Required branch/processor/handler obligation missing');
  return 'Scenario/mock gate passed: '+plan.suites.length+' suites; '+plan.suites.reduce(function(n,s){return n+s.tests.length;},0)+' required tests'+(plan.contracts.some(function(c){return c.status==='source-backed';})?'; source-backed generation; RAML unverified':'');
}
JXA
```
