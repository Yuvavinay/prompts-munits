---
mode: 'agent'
agent: 'agent'
name: 'munit-generate'
version: '12.1.0'
description: 'Route Mule applications to the HTTP or non-HTTP MUnit workflow.'
argument-hint: 'all | <file.xml> | <file1.xml,file2.xml,...> | errors'
---

`/munit-generate [all | file.xml | file1.xml,file2.xml | errors]`

1. Read production XML source declarations across the selected API, including nested files. Do not write files, resolve RAML, analyse DWL or generate tests here. Treat separate application modules independently.
2. An actual HTTP namespace `http://www.mulesoft.org/schema/mule/http` element named `listener`, directly under a Mule flow, selects HTTP. Outbound requests, listener-config, comments and source-less helpers do not. Incomplete/unreadable discovery is not proof that no listener exists.
3. Load and execute exactly one worker: listener present → [HTTP](munit-http-listener.prompt.md); otherwise → [non-HTTP](munit-non-http-listener.prompt.md).
4. Pass the module and exact arguments; omitted scope means `all`. Print `→ Routing to <worker>; scope <scope>` and continue its workflow without a continuation question. The selected worker owns all further logic and validation.
