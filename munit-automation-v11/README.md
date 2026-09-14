# MUnit automation — version 11.0.0

The generic command now performs read-only source detection and immediately routes: HTTP listener → HTTP worker; otherwise → non-HTTP worker. It creates no index or project structure. HTTP resolves POM/local RAML and applied common-library headers before recursive flow/DWL analysis. Only the named MUnit artifacts may change; even Maven target output is kept outside the application.

## Files and invocation

Install [the router](munit-generate.prompt.md), [the HTTP worker](munit-http-listener.prompt.md) and [the non-HTTP worker](munit-non-http-listener.prompt.md) together using the commands below. Refresh is integrated. The scenario plan remains version 3; fixture manifests are now version 2 with explicit type/enum/requiredness checks.

```text
/munit-generate all
/munit-generate orders.xml
/munit-generate orders.xml,customers.xml
/munit-generate errors
```

Omitting the argument means all. The prompts do not ask what to continue after routing, extraction, analysis or a suite. They proceed through the entire requested scope. Actual missing-root/access/essential-evidence blockers remain factual reports; host permission controls cannot be bypassed by a prompt.

## Execution sequence

1. Read only actual source declarations and choose one worker. Nested file discovery is permitted to find a listener; the router does not analyse calls/DWL or write an index.
2. HTTP reads pom.xml and local build metadata, resolves the effective local repository (normally .m2/repository), then locates and extracts the exact RAML artifact into OS temp outside the application.
3. Read each method's actual is entries, resolve its uses alias into the locally cached csp-dhisp-common-api-library.raml, and merge only applied trait/resourceType/resource/method request headers. Do not copy all headers from the library. Method is: [] does not cancel resource-applied traits. See the [RAML trait rules](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#applying-resource-types-and-traits).
4. Recursively analyse all selected endpoints, flows/sub-flows, choices/scopes/errors and DWL dependencies. Collect base request examples; derive branch copies from the complete conditions and preserve known enums/types/required fields. Non-HTTP derives inputs from actual entry consumers without a RAML stage.
5. Validate and stage all fixtures together, then construct every endpoint and required APIKit/main error suite. Existing valid tests stay byte-identical; add missing cases and patch affected implementations.
6. Validate the complete candidate tree and allowlisted write scope, publish narrow MUnit changes and re-read them. Run Mule checks only in a proven isolated temp mirror, so the original application gains no target or helper directories.

The only permitted application writes are suites under src/test/munit, JSON under src/test/resources/in and out, shared test-config.xml, and the two test YAML copies. Temporary indexes/plans/scripts/extraction/runtime files and installed command definitions stay outside the application. A native pre-write path guard rejects disallowed destinations, symlink ancestors and scratch locations inside the module. Its result is a path check, not a lock or content-validation claim.

In a mixed HTTP/scheduler application, generic routing still selects HTTP; independent non-HTTP entries are disclosed outside that scope and can be tested with an explicit direct non-HTTP command. Reactor modules are independent.

## Optional subagents

After prerequisites, enabled subagents can analyse independent endpoint/DWL scopes or review staged output read-only. They return compact source-backed findings. The parent remains the sole writer, merges all obligations and completes failed/incomplete delegation locally. No additional agent files, extensions, model overrides or settings are required. Without the capability, the same workflow runs serially. Context isolation may reduce repetition in the parent; lower total token usage is not guaranteed. See [VS Code subagents](https://code.visualstudio.com/docs/agents/run/subagents) and [prompt tools](https://code.visualstudio.com/docs/agent-customization/prompt-files).

## Contract and validation limits

Known enum/type/requiredness facts now become executable fixture constraints. A deliberate invalid APIKit input has a separate role and an exact expected constraint failure; it cannot be reused as a valid business variant. These checks complement semantic RAML/DWL analysis, not a complete RAML parser or runtime guarantee.

Missing main RAML still stops REST project writes. A referenced common library is searched in local .m2 even when absent from the root ZIP. Only after exhaustive local resolution may the retained source-backed policy apply to a genuinely unavailable fragment; affected suites carry pending-contract markers and evidenced values. No missing headers, secrets or business fields are invented.

## One-time installation or update

Extract this package outside the Mule project. Open a terminal in the directory containing the three prompt files and paste the appropriate block below. Replace all three definitions together, then start a new chat so cached older instructions do not control the run. Select Agent and Auto (or your existing model selection). Existing authorized VS Code chat and the application's provisioned toolchain remain prerequisites; these commands install no extensions or dependencies.

Default is the VS Code user prompt directory. If previously installed as skills, change Format/format to Skill/skill; keep one active format per command. Use an absolute Destination/destination outside every Maven application for a custom VS Code user profile. The installer rejects an application directory or its descendants; do not copy command definitions into .github or .vscode in the project. Updates back up differing files and leave identical definitions untouched. The installer changes prompt definitions, not application tests.

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
    $destinationFull=[IO.Path]::GetFullPath($Destination)
    $scanDestination=$destinationFull
    while ($scanDestination) {
        if (Test-Path -LiteralPath (Join-Path $scanDestination 'pom.xml') -PathType Leaf) { throw 'Command definitions must be installed outside Maven applications' }
        if (Test-Path -LiteralPath $scanDestination) {
            $ancestor=Get-Item -LiteralPath $scanDestination -Force
            if ($ancestor.Attributes -band [IO.FileAttributes]::ReparsePoint) { throw 'Resolve installation destination without symlink/reparse ancestors' }
        }
        $parent=[IO.Directory]::GetParent($scanDestination); if ($null -eq $parent) { break }; $scanDestination=$parent.FullName
    }
    $Destination=$destinationFull
    $utf8 = New-Object Text.UTF8Encoding($false)
    $jobs = @()
    foreach ($name in @('munit-http-listener','munit-non-http-listener','munit-generate')) {
        $source = Join-Path $bundle ($name+'.prompt.md')
        if (-not (Test-Path -LiteralPath $source -PathType Leaf)) { throw "Missing bundled file: $source" }
        $content = [IO.File]::ReadAllText($source)
        $parts = [regex]::Match($content,'\A---\r?\n(?<header>.*?)\r?\n---\r?\n(?<body>[\s\S]*)\z',[Text.RegularExpressions.RegexOptions]::Singleline)
        if (-not $parts.Success -or $parts.Groups['header'].Value -notmatch "(?m)^name: '$name'\r?$" -or $parts.Groups['header'].Value -notmatch "(?m)^version: '11\.0\.0'\r?$") { throw "Invalid bundled metadata: $source" }
        if ($Format -eq 'Skill') {
            $header=$parts.Groups['header'].Value
            $description=[regex]::Match($header,'(?m)^description:.*').Value.TrimEnd("`r")
            $body=$parts.Groups['body'].Value -replace '\((munit-[a-z-]+)\.prompt\.md\)','(../$1/SKILL.md)'
            $content="---`nname: '$name'`n$description`nmetadata:`n  version: '11.0.0'`n---`n"+$body
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
case "$destination" in /*) ;; *) echo 'Use an absolute user-profile destination outside the application' >&2; exit 1 ;; esac
scan_destination=$destination
while :; do
    [ ! -f "$scan_destination/pom.xml" ] || { echo 'Command definitions must be installed outside Maven applications' >&2; exit 1; }
    [ ! -L "$scan_destination" ] || { echo 'Resolve installation destination without symlink ancestors' >&2; exit 1; }
    [ "$scan_destination" != / ] || break
    scan_destination=$(/usr/bin/dirname "$scan_destination")
done
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
      !closed && $0 == "version: '\''11.0.0'\''" {version_ok=1}
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

See [VALIDATION.md](VALIDATION.md) for executed native/synthetic checks and unverified areas. Windows and actual VS Code/agent/Mule runtime behavior require real-environment validation. The prompt content is model-unpinned; that does not guarantee identical behavior or correctness across models. The non-null assertion alone does not prove business correctness or coverage. No access to the user's application, Maven cache or excluded other system was used.
