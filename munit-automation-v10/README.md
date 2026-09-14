# MUnit automation — version 10.0.0

Version 10 makes the generic router a binary API-level decision: an actual HTTP listener selects REST; no HTTP listener selects non-HTTP. It adds native recursive XML/resource discovery, explicit caller-aware XML/DWL traversal, recursive RAML example collection and a single validated JSON fixture batch before suite XML. Existing-test reconciliation and version 9's source-backed missing-library policy remain.

## Files and invocation

Use [the router](munit-generate.prompt.md), [the HTTP worker](munit-http-listener.prompt.md), and [the non-HTTP worker](munit-non-http-listener.prompt.md). Refresh remains integrated. The temporary plan schema is now version 3 and requires explicit contract status/provenance.

```text
/munit-generate all
/munit-generate orders.xml
/munit-generate orders.xml,customers.xml
/munit-generate errors
```

Without an argument, the router asks once for scope. With an explicit scope it executes the work, plans all selected owners, stages all JSON together, then constructs each endpoint and required error suite, and does not end with a discovery-only report or an A/B/C continuation menu.

## Routing and recursive workflow

The whole production module is scanned even when one helper XML is selected. Namespace-aware discovery does not mistake outbound requests or listener configurations for sources. A mixed module selects REST; independent scheduler/subscriber entries are reported outside REST scope and can be tested explicitly with `/munit-non-http-listener <file.xml>`. Reactor modules are classified separately.

1. Index XML files and nested elements recursively; resolve every selected filename through callers/APIKit ownership.
2. For REST, extract the POM-selected RAML and recursively resolve dependencies, traits, resource types, types and all selected request examples. Missing root RAML still halts project writes.
3. Expand each flow/sub-flow, scope, branch, error handler and DWL dependency in its caller context. Reuse file reads; preserve repeated-call and branch obligations. Track active cycles separately from completed reads.
4. Prepare every selected request example, branch variant and raw backend fixture. Validate all available constraints, then run one native JSON batch preflight/staging command. RAML response examples do not become backend fixtures without pass-through proof.
5. Construct all owning XML suites from the complete plan, validate and apply narrow differences. Existing valid tests and fixture bytes remain unchanged.

Commands are embedded in each worker, so it remains independently executable. Discovery emits a compact summary and an OS-temp index, not a source dump. The index and batch scripts are deterministic helpers; RAML/DWL semantic interpretation remains an agent task, supported by explicit provenance and validation gates. This reduces repeated reads but does not claim measured token savings or universal model reliability.

Custom DWL imports/mapping calls and classpath resources follow the project's actual resource layout. See [DataWeave modules](https://docs.mulesoft.com/dataweave/latest/dataweave-create-module) and [readUrl](https://docs.mulesoft.com/dataweave/2.9/dw-core-functions-readurl). Full example/reference semantics follow the [RAML 1.0 specification](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md).

## Missing-library behavior retained from version 9

| Observation | Current action |
| --- | --- |
| Main API archive found, common library missing from that ZIP | Search exact local dependency coordinates and bounded fallbacks. |
| Library still unavailable locally | Continue building source-backed suites using resolved RAML portions, current operation-matched tests/fixtures, XML/DWL consumers and test properties. |
| Callable helpers listed as non-HTTP | Keep them in the HTTP graph unless an actual non-HTTP inbound source is evidenced. |
| Existing valid tests | Preserve them; add missing cases or narrowly update stale requests/mocks/fixtures. |
| Library is restored on a later run | Reconcile the full RAML contract, revalidate, and remove pending markers only where resolved. |

Each affected suite carries this top-level comment:

```xml
<!-- RAML-CONTRACT: source-backed; required-header completeness is unverified. -->
```

The native gate accepts explicit source-backed status only with unresolved-reference records, literal provenance found in current files and the suite marker. Headers still need evidenced values, HTTP requests retain Request to <path>, and recursive scenarios/mocks and APIKit/main error ownership remain required. A same-named old test is not automatically valid evidence.

Files generated with incomplete RAML evidence are not claimed to have complete required headers, full contract validation or passing runtime tests. A missing library may also prevent application initialization. If essential input/property/trigger facts cannot be established from any available evidence, the agent must identify those cases instead of fabricating values or writing empty/disabled suites. The missing main API stop remains. No prompt can guarantee correct suites for every application or every model.

The dependency probe remains a candidate locator, not a RAML parser. [RAML reference resolution](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#resolving-includes), [Maven layout](https://maven.apache.org/repositories/layout.html)

The first progress line should identify `MUnit generator v10.0.0 | plan v3 | missing-library policy: source-backed`. If it does not, check the loaded file, active Prompt/Skill format and VS Code profile; a stale definition or ignored instruction may still control the run.

## One-time installation or update

Extract this package outside the Mule project. Open a terminal in the directory containing the three prompt files and paste the appropriate block below. Replace all three definitions together, then start a new chat so cached older instructions do not control the run. Select Agent and Auto (or your existing model selection). Existing authorized VS Code chat and the application's provisioned toolchain remain prerequisites; these commands install no extensions or dependencies.

Default is the VS Code user prompt directory. If previously installed as skills, change Format/format to Skill/skill; keep one active format per command. Use Destination/destination for a custom VS Code profile. Updates back up differing files and leave identical definitions untouched. The installer changes prompt definitions, not application tests.

### Windows: paste into Windows PowerShell 5.1

```powershell
function Install-MUnitCommands {
    param([ValidateSet('Prompt','Skill')][string]$Format='Prompt', [string]$Destination='', [switch]$Update)
    $ErrorActionPreference = 'Stop'
    $bundle = (Get-Location).Path
    $profileRoot = [Environment]::GetFolderPath('UserProfile')
    $promptRoot = Join-Path ([Environment]::GetFolderPath('ApplicationData')) 'Code/User/prompts'
    $skillRoot = Join-Path $profileRoot '.copilot/skills'
    $useDefault = [string]::IsNullOrWhiteSpace($Destination)
    if ($useDefault) { if ($Format -eq 'Prompt') { $Destination=$promptRoot } else { $Destination=$skillRoot } }
    $utf8 = New-Object Text.UTF8Encoding($false)
    $jobs = @()
    foreach ($name in @('munit-http-listener','munit-non-http-listener','munit-generate')) {
        $source = Join-Path $bundle ($name+'.prompt.md')
        if (-not (Test-Path -LiteralPath $source -PathType Leaf)) { throw "Missing bundled file: $source" }
        $content = [IO.File]::ReadAllText($source)
        $parts = [regex]::Match($content,'\A---\r?\n(?<header>.*?)\r?\n---\r?\n(?<body>[\s\S]*)\z',[Text.RegularExpressions.RegexOptions]::Singleline)
        if (-not $parts.Success -or $parts.Groups['header'].Value -notmatch "(?m)^name: '$name'\r?$" -or $parts.Groups['header'].Value -notmatch "(?m)^version: '10\.0\.0'\r?$") { throw "Invalid bundled metadata: $source" }
        if ($Format -eq 'Skill') {
            $header=$parts.Groups['header'].Value
            $description=[regex]::Match($header,'(?m)^description:.*').Value.TrimEnd("`r")
            $body=$parts.Groups['body'].Value -replace '\((munit-[a-z-]+)\.prompt\.md\)','(../$1/SKILL.md)'
            $content="---`nname: '$name'`n$description`nmetadata:`n  version: '10.0.0'`n---`n"+$body
            $target=Join-Path $Destination ($name+'/SKILL.md'); $other=Join-Path $promptRoot ($name+'.prompt.md')
        } else { $target=Join-Path $Destination ($name+'.prompt.md'); $other=Join-Path $skillRoot ($name+'/SKILL.md') }
        if ($useDefault -and (Test-Path -LiteralPath $other)) { throw "Other command format exists: $other. Keep one active format." }
        $bytes=$utf8.GetBytes($content); $changed=$true
        if (Test-Path -LiteralPath $target) {
            $item=Get-Item -LiteralPath $target -Force
            if ($item.PSIsContainer -or ($item.Attributes -band [IO.FileAttributes]::ReparsePoint)) { throw "Destination is not a regular file: $target" }
            $changed=[Convert]::ToBase64String([IO.File]::ReadAllBytes($target)) -cne [Convert]::ToBase64String($bytes)
            if ($changed -and -not $Update) { throw "Existing definition differs: $target. Use -Update to back up and replace it." }
        }
        $jobs += [pscustomobject]@{target=$target;bytes=$bytes;changed=$changed}
    }
    foreach ($job in $jobs) {
        if (-not $job.changed) { Write-Output ('Unchanged: '+$job.target); continue }
        $directory=[IO.Path]::GetDirectoryName($job.target)
        [IO.Directory]::CreateDirectory($directory) | Out-Null
        if (Test-Path -LiteralPath $job.target) { $backup=$job.target+'.backup-'+[guid]::NewGuid().ToString('D'); [IO.File]::Copy($job.target,$backup,$false); Write-Output ('Backup: '+$backup) }
        [IO.File]::WriteAllBytes($job.target,$job.bytes)
        if ([Convert]::ToBase64String([IO.File]::ReadAllBytes($job.target)) -cne [Convert]::ToBase64String($job.bytes)) { throw 'Installed content verification failed' }
        Write-Output ('Installed: '+$job.target)
    }
    Write-Output 'Start a new VS Code chat if needed. Select Agent and Auto, then run /munit-generate all.'
}
Install-MUnitCommands -Format Prompt -Update
```

### macOS: paste into a terminal

Run the block using the system Bash (for example enter `/bin/bash` first). PowerShell is not required on macOS.

```bash
install_munit_commands() {
set -euo pipefail
format=prompt
destination=''
update=false
while [ "$#" -gt 0 ]; do
    case "$1" in
        --format) [ "$#" -ge 2 ] || exit 2; format=$2; shift 2 ;;
        --destination) [ "$#" -ge 2 ] && [ -n "$2" ] || exit 2; destination=$2; shift 2 ;;
        --update) update=true; shift ;;
        --help) echo 'Usage: /bin/bash install-munit-command.sh [--format skill|prompt] [--destination DIRECTORY] [--update]'; exit 0 ;;
        *) echo "Unknown argument: $1" >&2; exit 2 ;;
    esac
done
case "$format" in skill|prompt) ;; *) echo 'Format must be skill or prompt' >&2; exit 2 ;; esac
bundle=$PWD
skill_root="$HOME/.copilot/skills"
prompt_root="$HOME/Library/Application Support/Code/User/prompts"
use_default=false
if [ -z "$destination" ]; then
    use_default=true
    if [ "$format" = skill ]; then destination=$skill_root; else destination=$prompt_root; fi
fi
stage_dir=$(/usr/bin/mktemp -d "${TMPDIR:-/tmp}/munit-install.XXXXXXXX")
target_stage=''
cleanup() {
    /bin/rm -f -- "$stage_dir/munit-http-listener" "$stage_dir/munit-non-http-listener" "$stage_dir/munit-generate"
    /bin/rmdir -- "$stage_dir"
    if [ -n "$target_stage" ]; then /bin/rm -f -- "$target_stage"; fi
}
trap cleanup EXIT
names=(munit-http-listener munit-non-http-listener munit-generate)
targets=()
changes=()
# Preflight all three destinations before publishing any file; router is last.
for name in "${names[@]}"; do
    source="$bundle/$name.prompt.md"
    [ -f "$source" ] || { echo "Missing bundled file: $source" >&2; exit 1; }
    /usr/bin/awk -v expected="$name" '
      NR == 1 {if ($0 != "---") bad=1; next}
      !closed && $0 == "---" {closed=1; next}
      !closed && $0 == "name: '\''" expected "'\''" {name_ok=1}
      !closed && $0 == "version: '\''10.0.0'\''" {version_ok=1}
      END {if (bad || !closed || !name_ok || !version_ok) exit 1}
    ' "$source" || { echo "Invalid bundled name/version: $source" >&2; exit 1; }
    if [ "$format" = skill ]; then
        target="$destination/$name/SKILL.md"
        other="$prompt_root/$name.prompt.md"
        /usr/bin/awk '
          NR == 1 {print; next}
          !closed && $0 == "---" {print "metadata:"; print "  " version; print "---"; closed=1; next}
          !closed {if ($0 ~ /^(name|description):/) print; if ($0 ~ /^version:/) version=$0; next}
          {print}
        ' "$source" | /usr/bin/sed -E 's@\((munit-[a-z-]+)\.prompt\.md\)@(../\1/SKILL.md)@g' > "$stage_dir/$name"
    else
        target="$destination/$name.prompt.md"
        other="$skill_root/$name/SKILL.md"
        /bin/cp -- "$source" "$stage_dir/$name"
    fi
    if [ "$use_default" = true ] && [ -e "$other" ]; then
        echo "Other command format exists at $other. Keep one active format; relocate it before switching." >&2; exit 1
    fi
    if [ -L "$target" ] || { [ -e "$target" ] && [ ! -f "$target" ]; }; then
        echo "Destination is not a regular file: $target" >&2; exit 1
    fi
    changed=true
    if [ -f "$target" ]; then
        if /usr/bin/cmp -s "$target" "$stage_dir/$name"; then changed=false
        elif [ "$update" = false ]; then
            echo "Existing content differs: $target. Use --update to preserve a backup and replace the installed skill." >&2; exit 1
        fi
    fi
    targets+=("$target")
    changes+=("$changed")
done
for ((i=0; i<${#names[@]}; i++)); do
    target=${targets[$i]}
    if [ "${changes[$i]}" = false ]; then echo "Unchanged: $target"; continue; fi
    directory=$(dirname -- "$target")
    /bin/mkdir -p -- "$directory"
    target_stage=$(/usr/bin/mktemp "$directory/.munit-install.XXXXXXXX")
    /bin/cp -- "$stage_dir/${names[$i]}" "$target_stage"
    /usr/bin/cmp -s "$target_stage" "$stage_dir/${names[$i]}"
    if [ -f "$target" ]; then
        backup="$target.backup-$(/usr/bin/uuidgen)"
        /bin/cp -p -- "$target" "$backup"
        echo "Backup: $backup"
    fi
    /bin/mv -f -- "$target_stage" "$target"
    target_stage=''
    /usr/bin/cmp -s "$target" "$stage_dir/${names[$i]}"
    echo "Installed: $target"
done
echo 'Reload VS Code if needed. Select Agent and Auto; run /munit-generate all.'
echo 'This installer updates skill definitions only. Selected application tests are reconciled by the workers.'

}
install_munit_commands --format prompt --update
```

## Validation and limits

See [VALIDATION.md](VALIDATION.md) for executed checks and unverified areas. This version retains version 9 contract/provenance checks and adds native recursive discovery and fixture-batch checks. Handler validation follows the selected source graph; a separate full inventory accounts for independently owned and unowned declarations. Installation commands remain embedded and model selection remains unpinned.

No access to the user's failing application, Maven cache or other system was used. The quoted library version and backend names are not built-in defaults. Windows execution, actual agent adherence, complete RAML interpretation and real Mule runtime behavior remain unverified. Static checks do not prove coverage or business correctness; the required non-null assertion alone cannot establish either. No model quality or token/cost saving is claimed.
