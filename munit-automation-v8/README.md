# MUnit automation — version 8.0.0

This update addresses premature stopping when a RAML library is absent from the main ZIP and misclassification of source-less APIKit/business flows as non-HTTP entries. It preserves version 7's required header maps, Request to <path> names, error suites, recursive scenario/mock checks, and targeted refresh.

## Use

The three implementation files are [the router](munit-generate.prompt.md), [the HTTP worker](munit-http-listener.prompt.md), and [the non-HTTP worker](munit-non-http-listener.prompt.md). All worker commands remain embedded in the Markdown. The native plan schema remains version 2.

```text
/munit-generate
/munit-generate all
/munit-generate orders.xml
/munit-generate orders.xml,customers.xml
/munit-generate errors
```

No argument asks for scope. An explicit scope authorizes the full generation/refresh process; the agent should not stop after a plan summary or offer unrelated options when it can continue.

## What the reported stop means

The report shows that the main API RAML was found. A referenced common library was absent from that extraction; the report does not establish whether it is also absent from the effective local repository. Version 8 requires exact-coordinate lookup, archive inspection, a bounded local fallback, safe extraction and recursive dependency resolution before asking for missing content. A library that is genuinely unavailable still blocks generation; inventing required headers would defeat the contract checks.

RAML references resolve in the context of their referring document; a separately extracted library keeps its own include tree and a temp reference mapping. Maven coordinates determine repository paths. [RAML specification](https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md#resolving-includes), [Maven repository layout](https://maven.apache.org/repositories/layout.html)

The local probe lists candidate archives/entries; it is not a RAML parser, transitive resolver or automatic identity selector. The worker must execute the complete dependency procedure and verify each candidate. Corrupt/unreadable/ambiguous dependencies receive distinct diagnostics. No download, Maven dependency installation, production edit or `.m2` write is introduced.

The report also listed callable business flows as non-HTTP/public flows. A flow without an inbound source is a callable dependency. The router now requires an actual source element before creating non-HTTP work and retains HTTP-owned helpers inside endpoint traversal. The reported flow names are diagnostic examples, not hardcoded routing rules.

Different outcomes between models do not establish which generated tests are correct. The instructions remain model-agnostic and do not select a provider. Contract provenance and actual runtime results remain the evidence of correctness.

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
        if (-not $parts.Success -or $parts.Groups['header'].Value -notmatch "(?m)^name: '$name'\r?$" -or $parts.Groups['header'].Value -notmatch "(?m)^version: '8\.0\.0'\r?$") { throw "Invalid bundled metadata: $source" }
        if ($Format -eq 'Skill') {
            $header=$parts.Groups['header'].Value
            $description=[regex]::Match($header,'(?m)^description:.*').Value.TrimEnd("`r")
            $body=$parts.Groups['body'].Value -replace '\((munit-[a-z-]+)\.prompt\.md\)','(../$1/SKILL.md)'
            $content="---`nname: '$name'`n$description`nmetadata:`n  version: '8.0.0'`n---`n"+$body
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
      !closed && $0 == "version: '\''8.0.0'\''" {version_ok=1}
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

See [VALIDATION.md](VALIDATION.md) for executed checks and unverified areas. This package was updated using retained local QA copies of the version 7 prompts because the previous outputs directory was empty at the start of this update. Generation rules and native scenario gates were retained; the installer is included and its tested behavior is recorded separately.

No access to the user's failing application, Maven cache or other system was used. The quoted library version and backend names are not built-in defaults. Windows execution, actual agent adherence, complete RAML interpretation and real Mule runtime behavior remain unverified. Static checks do not prove coverage or business correctness; the required non-null assertion alone cannot establish either. No model quality or token/cost saving is claimed.
