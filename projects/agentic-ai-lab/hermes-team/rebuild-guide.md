# Hermes Lean Agent Team Rebuild Guide

This guide rebuilds the validated four-profile Hermes team from repository files. The repository is the source of truth; do not rely on remembered UI settings or profile-only state.

Validated environment: native Windows with PowerShell. Commands below are PowerShell commands run from the repository root unless a section says otherwise. Git Bash equivalents may work on this machine, but PowerShell is the validated rebuild path.

Validated Hermes version: `Hermes Agent v0.21.1 (2026.9.7)`. Rebuild behavior, defaults, command output, and skill inventory may differ on newer Hermes versions. Re-verify before treating this guide as authoritative for a later release.

Source files:

- `team-config.md`
- `README.md`
- `change-log.md`
- `odin-soul.md`
- `brokkr-soul.md`
- `veritas-soul.md`
- `bao-soul.md`
- `skills/software-development/lean-code-review/SKILL.md`

## Validated team

| Profile | Role | Validated model | Provider | Notes |
|---|---|---|---|---|
| `odin` | Chief Agent / Orchestrator | `gpt-5.6-sol` | `openai-codex` | routes to the smallest necessary specialist set |
| `brokkr` | Developer Agent | `gpt-5.5` | `openai-codex` | implementation, code, config, debugging |
| `veritas` | Quality Assurance Agent | `gpt-5.6-sol` | `openai-codex` | independent QA and requirement checks |
| `bao` | Research Agent | `deepseek/deepseek-v4-pro` | `nous` | research, current docs, fact verification |

Validated authentication method: profile-scoped OAuth/subscription credentials. Provider authentication is profile-specific: run setup/auth commands with `-p <profile>` and verify that each profile has credentials for the provider it is configured to use. Validated required credentials are `openai-codex` OAuth/subscription auth for Odin, Brokkr, and Veritas, and `nous` OAuth for Bao. Nous remains available to the team, but the final validated architecture uses it primarily for Bao research to reduce cost. Do not describe `openai-codex` as raw API-key usage.

## 1. Create the profiles

Run from the machine where Hermes is installed:

```powershell
hermes profile create odin --description "Chief orchestrator for the Hermes Lean Agent Team; breaks goals into the smallest necessary specialist tasks and integrates results."
hermes profile create brokkr --description "Developer specialist for implementation, code changes, configuration, debugging, and repository documentation."
hermes profile create veritas --description "Independent QA specialist for tests, requirements validation, regression review, and unsupported-claim checks."
hermes profile create bao --description "Research specialist for current documentation, external facts, technical comparison, and source-grounded findings."
```

If rebuilding over existing profiles, inspect first and avoid destructive cleanup unless approved:

```powershell
hermes profile list
hermes profile show odin
hermes profile show brokkr
hermes profile show veritas
hermes profile show bao
```

## 2. Configure models and providers

Use `hermes config set`; do not hand-edit `config.yaml`.

```powershell
hermes -p odin config set model.default gpt-5.6-sol
hermes -p odin config set model.provider openai-codex
hermes -p odin config set model.base_url https://chatgpt.com/backend-api/codex

hermes -p brokkr config set model.default gpt-5.5
hermes -p brokkr config set model.provider openai-codex
hermes -p brokkr config set model.base_url https://chatgpt.com/backend-api/codex

hermes -p veritas config set model.default gpt-5.6-sol
hermes -p veritas config set model.provider openai-codex
hermes -p veritas config set model.base_url https://chatgpt.com/backend-api/codex

hermes -p bao config set model.default deepseek/deepseek-v4-pro
hermes -p bao config set model.provider nous
hermes -p bao config set model.base_url https://inference-api.nousresearch.com/v1
```

Keep `agent.max_turns: 150` for Odin, Veritas, and Bao if matching the validated profiles. Brokkr was validated without a profile-level `agent.max_turns` override.

```powershell
hermes -p odin config set agent.max_turns 150
hermes -p veritas config set agent.max_turns 150
hermes -p bao config set agent.max_turns 150

# Brokkr was validated without this override. On a fresh profile this key may already be absent.
hermes -p brokkr config get agent.max_turns
if ($LASTEXITCODE -eq 0) { hermes -p brokkr config unset agent.max_turns }
```

## 3. Authenticate providers

Use profile-scoped OAuth/subscription auth. Do not store credentials in repository files. Run setup/auth commands per profile; successful auth for one profile does not prove that another profile can use the same provider. `openai-codex` is not raw API-key usage in this rebuild.

Required validated credentials:

```powershell
hermes -p odin setup model
hermes -p brokkr auth add openai-codex --type oauth
hermes -p veritas auth add openai-codex --type oauth
hermes -p bao auth add nous --type oauth
```

Hermes v0.21.1 migration lesson: changing Odin's model/provider config alone did not make Odin runnable. Odin lacked profile-scoped `openai-codex` auth, and `hermes -p odin setup model` correctly created the profile auth and completed the migration.

Optional credentials that existed during validation:

```powershell
hermes -p brokkr auth add nous --type oauth
hermes -p veritas auth add nous --type oauth
```

Verify without printing secrets:

```powershell
hermes -p odin auth list openai-codex
hermes -p brokkr auth list openai-codex
hermes -p brokkr auth list nous
hermes -p veritas auth list openai-codex
hermes -p veritas auth list nous
hermes -p bao auth list nous
```

## 4. Install SOUL files

Copy the repository SOUL files into the matching profile homes as `SOUL.md`. These commands are PowerShell commands.

```powershell
Copy-Item -LiteralPath "projects\agentic-ai-lab\hermes-team\odin-soul.md" -Destination "$env:LOCALAPPDATA\hermes\profiles\odin\SOUL.md" -Force
Copy-Item -LiteralPath "projects\agentic-ai-lab\hermes-team\brokkr-soul.md" -Destination "$env:LOCALAPPDATA\hermes\profiles\brokkr\SOUL.md" -Force
Copy-Item -LiteralPath "projects\agentic-ai-lab\hermes-team\veritas-soul.md" -Destination "$env:LOCALAPPDATA\hermes\profiles\veritas\SOUL.md" -Force
Copy-Item -LiteralPath "projects\agentic-ai-lab\hermes-team\bao-soul.md" -Destination "$env:LOCALAPPDATA\hermes\profiles\bao\SOUL.md" -Force
```

Verify the installed files by hash:

```powershell
$SoulPairs = @(
  @("projects\agentic-ai-lab\hermes-team\odin-soul.md", "$env:LOCALAPPDATA\hermes\profiles\odin\SOUL.md"),
  @("projects\agentic-ai-lab\hermes-team\brokkr-soul.md", "$env:LOCALAPPDATA\hermes\profiles\brokkr\SOUL.md"),
  @("projects\agentic-ai-lab\hermes-team\veritas-soul.md", "$env:LOCALAPPDATA\hermes\profiles\veritas\SOUL.md"),
  @("projects\agentic-ai-lab\hermes-team\bao-soul.md", "$env:LOCALAPPDATA\hermes\profiles\bao\SOUL.md")
)
foreach ($Pair in $SoulPairs) {
  $SourceHash = (Get-FileHash -Algorithm SHA256 -LiteralPath $Pair[0]).Hash
  $DestHash = (Get-FileHash -Algorithm SHA256 -LiteralPath $Pair[1]).Hash
  if ($SourceHash -ne $DestHash) { throw "SOUL mismatch: $($Pair[0]) -> $($Pair[1])" }
}
```

During validation, Odin, Veritas, and Bao matched their repository SOUL files. Brokkr's live `SOUL.md` did not match the repository file, so a rebuild should reinstall Brokkr's SOUL from `brokkr-soul.md` before runtime verification.

## 4a. Install Veritas code review skill

Install the repository-managed `lean-code-review` skill into the Veritas profile. This keeps Veritas' review behavior rebuildable from Git instead of depending on manual profile edits.

```powershell
$LeanReviewSkillSource = "projects\agentic-ai-lab\hermes-team\skills\software-development\lean-code-review\SKILL.md"
$LeanReviewSkillDestDir = "$env:LOCALAPPDATA\hermes\profiles\veritas\skills\software-development\lean-code-review"
New-Item -ItemType Directory -Force -Path $LeanReviewSkillDestDir | Out-Null
Copy-Item -LiteralPath $LeanReviewSkillSource -Destination (Join-Path $LeanReviewSkillDestDir "SKILL.md") -Force
```

Verify the installed skill by hash and by Hermes skill discovery:

```powershell
$SourceHash = (Get-FileHash -Algorithm SHA256 -LiteralPath $LeanReviewSkillSource).Hash
$DestHash = (Get-FileHash -Algorithm SHA256 -LiteralPath (Join-Path $LeanReviewSkillDestDir "SKILL.md")).Hash
if ($SourceHash -ne $DestHash) { throw "lean-code-review skill mismatch" }
hermes -p veritas skills list --enabled-only
```

Expected result: the enabled Veritas skill list includes `lean-code-review`.

## 5. Trim skills and plugins

The validated profiles use a lean skill surface with `plugins.enabled: []` and explicit CLI skill disables under `skills.platform_disabled.cli`. Do not invent role-based skill trimming; apply the exact lists below for the v0.21.1 validated rebuild, then verify with `hermes -p <profile> config get skills.platform_disabled.cli`.

PowerShell helper for applying lists:

```powershell
function Set-HermesCliDisabledSkills {
  param(
    [Parameter(Mandatory=$true)][string]$Profile,
    [Parameter(Mandatory=$true)][string[]]$Skills
  )
  hermes -p $Profile config set skills.platform_disabled.cli ($Skills | ConvertTo-Json -Compress)
  hermes -p $Profile config set plugins.enabled '[]'
}
```

Odin disabled CLI skills:

```powershell
Set-HermesCliDisabledSkills -Profile odin -Skills @(
  "airtable",
  "architecture-diagram",
  "ascii-video",
  "baoyu-infographic",
  "box",
  "claude-design",
  "design-md",
  "document-to-action-items",
  "docx",
  "email-inbox-triage",
  "gif-search",
  "google-workspace",
  "himalaya",
  "humanizer",
  "manim-video",
  "maps",
  "meeting-action-items",
  "notion",
  "obsidian",
  "p5js",
  "pdf",
  "popular-web-designs",
  "powerpoint",
  "product-price-monitor",
  "songsee",
  "songwriting-and-ai-music",
  "teams-meeting-pipeline",
  "weekly-review-planning",
  "xlsx",
  "youtube-content"
)
```

Brokkr disabled CLI skills:

```powershell
Set-HermesCliDisabledSkills -Profile brokkr -Skills @(
  "airtable",
  "architecture-diagram",
  "arxiv",
  "ascii-video",
  "baoyu-infographic",
  "blocked-page-recovery",
  "box",
  "claude-design",
  "competitor-news-monitor",
  "design-md",
  "document-to-action-items",
  "docx",
  "email-inbox-triage",
  "gif-search",
  "google-workspace",
  "grounded-citations",
  "himalaya",
  "humanizer",
  "llm-wiki",
  "manim-video",
  "maps",
  "meeting-action-items",
  "notion",
  "obsidian",
  "p5js",
  "pdf",
  "popular-web-designs",
  "powerpoint",
  "product-price-monitor",
  "songsee",
  "songwriting-and-ai-music",
  "teams-meeting-pipeline",
  "weekly-review-planning",
  "xlsx",
  "youtube-content"
)
```

Veritas disabled CLI skills:

```powershell
Set-HermesCliDisabledSkills -Profile veritas -Skills @(
  "airtable",
  "architecture-diagram",
  "arxiv",
  "ascii-video",
  "baoyu-infographic",
  "blocked-page-recovery",
  "box",
  "claude-code",
  "claude-design",
  "codex",
  "competitor-news-monitor",
  "computer-use",
  "design-md",
  "document-to-action-items",
  "docx",
  "email-inbox-triage",
  "gif-search",
  "google-workspace",
  "grounded-citations",
  "himalaya",
  "humanizer",
  "llm-wiki",
  "manim-video",
  "maps",
  "meeting-action-items",
  "notion",
  "obsidian",
  "opencode",
  "p5js",
  "pdf",
  "popular-web-designs",
  "powerpoint",
  "product-price-monitor",
  "songsee",
  "songwriting-and-ai-music",
  "teams-meeting-pipeline",
  "weekly-review-planning",
  "xlsx",
  "youtube-content"
)
```

Bao disabled CLI skills:

```powershell
Set-HermesCliDisabledSkills -Profile bao -Skills @(
  "airtable",
  "architecture-diagram",
  "ascii-video",
  "baoyu-infographic",
  "box",
  "claude-code",
  "claude-design",
  "codebase-inspection",
  "codex",
  "computer-use",
  "design-md",
  "document-to-action-items",
  "docx",
  "dogfood",
  "email-inbox-triage",
  "gif-search",
  "github",
  "google-workspace",
  "hermes-agent-skill-authoring",
  "himalaya",
  "humanizer",
  "inspecting-hermes-desktop-dom",
  "manim-video",
  "maps",
  "meeting-action-items",
  "node-inspect-debugger",
  "notion",
  "obsidian",
  "opencode",
  "p5js",
  "pdf",
  "popular-web-designs",
  "powerpoint",
  "product-price-monitor",
  "requesting-code-review",
  "simplify-code",
  "songsee",
  "songwriting-and-ai-music",
  "spike",
  "systematic-debugging",
  "teams-meeting-pipeline",
  "test-driven-development",
  "weekly-review-planning",
  "xlsx",
  "youtube-content"
)
```

Verify the current count and profile state:

```powershell
hermes -p odin config get skills.platform_disabled.cli
hermes -p brokkr config get skills.platform_disabled.cli
hermes -p veritas config get skills.platform_disabled.cli
hermes -p bao config get skills.platform_disabled.cli
hermes -p odin config get plugins.enabled
hermes -p brokkr config get plugins.enabled
hermes -p veritas config get plugins.enabled
hermes -p bao config get plugins.enabled
hermes profile show odin
hermes profile show brokkr
hermes profile show veritas
hermes profile show bao
```

Validated profile skill counts were 59 for Odin and 58 each for Brokkr, Veritas, and Bao. Treat those counts as a sanity check, not as a substitute for reviewing the disabled skill list.

## 6. Enable Kanban routing

Persistent named-profile routing uses Hermes Kanban. Do not use `delegate_task` when the task specifically requires Brokkr, Veritas, or Bao.

Reason: `delegate_task` creates a temporary child agent under the parent runtime. It does not load the named specialist profile, provider, model, SOUL, memory, or profile environment automatically.

Use `delegate_task` only for lightweight temporary child agents where persistent specialist identity is not required.

Kanban requirements:

- the profiles must exist on disk,
- the Kanban board must be initialized,
- the Hermes gateway/dispatcher must be running,
- `gateway.multiplex_profiles` must be `true` when dispatching named profiles through one gateway,
- task assignees must match real profile names.

Commands:

```powershell
hermes kanban init
hermes kanban assignees
hermes config set gateway.multiplex_profiles true
hermes gateway status
hermes gateway run      # foreground option
# or
hermes gateway install  # Windows Scheduled Task / service-style install
```

Validated routing examples:

```powershell
hermes kanban create "Developer task title" --assignee brokkr
hermes kanban create "QA task title" --assignee veritas
hermes kanban create "Research task title" --assignee bao
```

Default workflows remain:

- Routine implementation: Odin -> Brokkr -> Odin
- Implementation needing independent verification: Odin -> Brokkr -> Veritas -> Odin
- Research: Odin -> Bao -> Odin
- Research followed by implementation: Odin -> Bao -> Brokkr -> Veritas -> Odin

## 7. Verify runtime identity

Create a small Kanban task for each specialist asking it to report active profile, model, provider, and whether its SOUL behavior is loaded. Confirm results before using the team for real work. Odin must explicitly confirm that the persistent Odin `SOUL.md` is loaded; config values alone are not sufficient.

Expected validated runtime identities:

- Bao: profile `bao`, model `deepseek/deepseek-v4-pro`, provider `nous`, Bao SOUL loaded.
- Brokkr: profile `brokkr`, model `gpt-5.5`, provider `openai-codex`, Brokkr profile configuration loaded.
- Veritas: profile `veritas`, model `gpt-5.6-sol`, provider `openai-codex`, Veritas SOUL loaded.
- Odin: profile `odin`, model `gpt-5.6-sol`, provider `openai-codex`, persistent Odin SOUL loaded.

Useful verification commands:

```powershell
hermes --version
hermes profile list
hermes profile show odin
hermes profile show brokkr
hermes profile show veritas
hermes profile show bao
hermes kanban assignees
hermes gateway status
hermes -p odin auth list openai-codex
hermes -p brokkr auth list openai-codex
hermes -p brokkr auth list nous
hermes -p veritas auth list openai-codex
hermes -p veritas auth list nous
hermes -p bao auth list nous
hermes -p odin config get model.default
hermes -p odin config get model.provider
hermes -p brokkr config get model.default
hermes -p brokkr config get model.provider
hermes -p veritas config get model.default
hermes -p veritas config get model.provider
hermes -p bao config get model.default
hermes -p bao config get model.provider
```

## 8. Known cleanup issues

- Stale aliases: profile aliases are wrapper scripts such as `odin`, `brokkr`, `veritas`, and `bao`. Verify them with `hermes profile show <profile>`. Remove and recreate if they point at the wrong profile:

```powershell
$Profile = "odin"   # repeat for brokkr, veritas, and bao as needed
hermes profile alias $Profile --remove
hermes profile alias $Profile
```

- Old `llamacpp` provider state: validation found stale local endpoint probe cache for `llamacpp` under Hermes cache state, while the rebuilt profiles use `openai-codex` or `nous`. Do not copy old local-provider config into these profiles. If a rebuilt profile unexpectedly selects a local provider, reset it with the model/provider commands above.

- Brokkr SOUL drift: validation found Brokkr's live `SOUL.md` did not match `brokkr-soul.md`. Reinstall from the repository and re-run the hash check.

- Gateway state: `hermes profile list` may show individual profile gateways stopped while a gateway is running under another profile. Kanban dispatch still requires a running gateway/dispatcher; verify with `hermes gateway status` and `hermes kanban assignees` before assigning work.

- Gateway update/multiplex state: after a Hermes update, a running gateway may still serve pre-update modules until restarted. Restart the gateway before validating dispatch, and keep `gateway.multiplex_profiles` set to `true` so named profiles can be dispatched through the gateway.

- Windows migration warning: native Windows rebuilds should follow the PowerShell path above. Treat shell/path differences as orchestration and platform constraints to re-validate, not as hidden failures.

## 9. Final rebuild checklist

- [ ] Four profiles exist: `odin`, `brokkr`, `veritas`, `bao`.
- [ ] Hermes version is checked; this guide was validated on `v0.21.1`.
- [ ] Windows PowerShell is used for the commands above, or non-PowerShell commands are explicitly translated and re-validated.
- [ ] Models and providers match the validated table.
- [ ] Profile-scoped OAuth/subscription auth is configured and verified per profile for each configured provider.
- [ ] Repository SOUL files are installed and hash-clean.
- [ ] `lean-code-review` is installed for Veritas from the repository and appears in `hermes -p veritas skills list --enabled-only`.
- [ ] Odin's persistent `SOUL.md` is confirmed loaded at runtime.
- [ ] Unneeded CLI skills are disabled exactly as listed for each profile.
- [ ] `plugins.enabled` is `[]` for each profile unless intentionally changed.
- [ ] Gateway/dispatcher has been restarted after any Hermes update and is running with `gateway.multiplex_profiles = true`.
- [ ] Kanban assignees show all four profiles on disk.
- [ ] Runtime identity tasks confirm profile, model, provider, and SOUL behavior.
- [ ] No stale aliases or old local `llamacpp` provider settings are being used.

Do not commit rebuild changes automatically. Review the working tree first.
