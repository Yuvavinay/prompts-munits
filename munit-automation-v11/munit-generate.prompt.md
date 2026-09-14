---
mode: 'agent'
agent: 'agent'
name: 'munit-generate'
version: '11.0.0'
description: 'Detect the API source type without writes, then execute the HTTP or non-HTTP worker.'
argument-hint: 'all | <file.xml> | <file1.xml,file2.xml,...> | errors'
tools: ['agent', 'read', 'search', 'edit', 'execute']
---

# Source-type router only

`/munit-generate [all | file.xml | file1.xml,file2.xml | errors]`

1. Print `MUnit generator v11.0.0 | plan v3`, then `→ Detecting API source type (read-only)`. First inspect the current API's production XML source declarations, without writing ANY file or directory, even a temporary index. Use existing read/search tools. Enumerate nested production XML paths only as necessary to locate sources; do not expand calls, inventory DWL, plan tests, extract examples, scaffold a project or create settings. Do not use tests/build output as source evidence. Resolve module boundaries from the open application; classify reactor modules separately.
2. Search the WHOLE selected API, even if the argument names a helper file. Confirm an actual HTTP namespace `http://www.mulesoft.org/schema/mule/http` element with local name `listener` directly under a core `flow`. Prefixes may differ. An outbound request, listener-config, comment, filename or source-less public/sub-flow is not a listener. A no-listener result requires all applicable production source declarations to be readable and checked; unresolved roots or parse errors are incomplete discovery, never a non-HTTP guess. Read a build-root declaration only if needed to identify the actual source locations; defer POM dependency/repository analysis to the worker.
3. Route immediately from that evidence:

| Source evidence | Installed worker to read and execute |
| --- | --- |
| One or more HTTP listeners | [HTTP worker](munit-http-listener.prompt.md) |
| No HTTP listener in the API | [Non-HTTP worker](munit-non-http-listener.prompt.md) |

4. Print `→ Route: <HTTP|non-HTTP>; scope: <argument>; listener evidence: <file/flow or none>`. Omitted argument means `all`; announce that default and continue. Pass only module/source evidence and the exact scope to the worker. Load its installed file using read tools and perform its full workflow in this invocation. Do not end after routing, ask what to continue, or send the user a link instead of executing it.
5. The HTTP worker first reads pom.xml for coordinates/repository configuration, locates/extracts local RAML, resolves the applied common-library traits and headers, and ONLY THEN performs recursive flow/DWL/scenario analysis. The non-HTTP worker skips RAML and derives inputs from entry consumers. All detailed analysis, fixture writing and reconciliation rules belong to those workers, not this router.
6. In an API containing HTTP plus independent scheduler/subscriber entries, the binary rule selects HTTP. List independent non-HTTP entries as outside that HTTP scope; do not launch both workers automatically or mark them covered. Explicit direct non-HTTP invocation remains available. Helpers shared by actual callers retain each caller context without becoming invented entry sources.

Never install definitions inside the Mule application during generation. Do not create .github, .vscode, .agents, plans, scripts, target, caches or any production directory. The selected worker is responsible for its narrowly allowed MUnit writes and all validation. Existing correct tests remain unchanged; the worker adds missing cases and patches affected tests.

Use enabled tools and the user's selected model without overrides. Do not require subagents, new extensions or settings. Workers may use available subagents for bounded read-only analysis AFTER their prerequisites, with the parent as sole writer; otherwise they continue serially. No routine continuation questions. A genuine missing root contract/access/essential-evidence blocker is reported precisely, without fabricated values or a menu of unrelated work. Host permission controls remain in force.
