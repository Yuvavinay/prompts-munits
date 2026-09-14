# MUnit automation — version 12.0.0

Three self-contained prompt files: [routing only](munit-generate.prompt.md), [HTTP generation/refresh](munit-http-listener.prompt.md) and [non-HTTP generation/refresh](munit-non-http-listener.prompt.md). Use the installer below once, select Agent and Auto in the existing VS Code chat, and run `/munit-generate all`, a source XML filename/list, or `errors`. Omitted scope is all. Replace all three older definitions together and start a fresh chat.

## Main API versus header library

The HTTP worker selects the MAIN RAML artifact from pom.xml and the APIKit reference, locates its exact coordinates in the effective local Maven repository, and extracts it. The known csp-dhisp-common-api-library.raml filename never selects or replaces that artifact. After extraction, the worker follows each method's actual is/uses references into locally cached trait libraries to resolve effective request headers. Only applied traits and inherited/explicit declarations contribute headers.

Then it recursively analyses every selected operation, flow/sub-flow, DWL dependency, branch and handler; validates all fixture examples/variants together; generates every owning suite; and reconciles existing tests without unnecessary writes. No default one-test-per-suite behavior. APIKit and main-handler suites remain mandatory when source ownership requires them. Non-HTTP derives entry inputs from XML/DWL consumers and does not inherit REST execution rules.

## What was simplified

The router now contains only detection, two links and handoff. Each worker has one workflow, one source-derived ledger, one invariant table and one acceptance checklist. Large duplicate native discovery, version-3 plan-validation and path-guard frameworks were removed. The allowed write scope and required output rules remain explicit. Safe extraction/local probes and executable fixture type/enum/requiredness gates remain embedded; small XML parsing commands replace the large XML framework.

This is an instruction-driven workflow, not a complete deterministic RAML/DWL compiler. Source-to-scenario/mock/header/handler completeness and publication boundaries are enforced by the agent's acceptance procedure, rather than the former large native plan/path scripts. Do not claim equivalent automated enforcement or reuse old v11 validation counts. All scripts are contained in the worker Markdown; no external helper files, model pinning or new dependencies are required.

Only the specified MUnit suite/JSON/shared-config/two-property artifacts may change in the application. Scratch and runtime output stay outside it. Existing valid tests/fixtures remain unchanged; missing scenarios are added and stale selected content is patched. Optional enabled subagents only analyse/review; the parent alone writes and can complete the same work serially. No routine continuation menus; real missing-root/access/essential-evidence issues are reported precisely.

## Validation status and production use

See [VALIDATION.md](VALIDATION.md). This revision was checked using platform-neutral Python authoring/structural checks only. No macOS-specific command, PowerShell, installer, VS Code or Mule command was executed for this revision. Commands retained from v11 were compared as text where indicated; that is not a fresh platform execution test.

Use the supplied workflow as a reviewed release candidate. Production readiness cannot be certified without validating it against the actual application, provisioned Mule/MUnit schemas/runtime and supported development environments. No application was supplied or run here. Prompt brevity and an unpinned model do not guarantee coverage, lower total tokens or identical model behavior.

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
        if (-not $parts.Success -or $parts.Groups['header'].Value -notmatch "(?m)^name: '$name'\r?$" -or $parts.Groups['header'].Value -notmatch "(?m)^version: '12\.0\.0'\r?$") { throw "Invalid bundled metadata: $source" }
        if ($Format -eq 'Skill') {
            $header=$parts.Groups['header'].Value
            $description=[regex]::Match($header,'(?m)^description:.*').Value.TrimEnd("`r")
            $body=$parts.Groups['body'].Value -replace '\((munit-[a-z-]+)\.prompt\.md\)','(../$1/SKILL.md)'
            $content="---`nname: '$name'`n$description`nmetadata:`n  version: '12.0.0'`n---`n"+$body
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
      !closed && $0 == "version: '\''12.0.0'\''" {version_ok=1}
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

