# Codex Autoresearch Windows Skill

Codex Autoresearch packaged for the Codex Windows desktop/client experience.

This repository is the Windows-friendly skill distribution of Codex Autoresearch:
clone it, install it into `%USERPROFILE%\.codex\skills\codex-autoresearch`, restart
Codex, and invoke `$codex-autoresearch` from any project.

## What It Does

Codex Autoresearch turns a measurable engineering or research goal into an
autonomous improve-verify loop:

1. Scan the target repo.
2. Propose a metric, scope, verify command, and guard command.
3. Ask for foreground or background mode.
4. Make one focused change.
5. Verify mechanically.
6. Keep improvements, discard regressions, and log every iteration.
7. Repeat until stopped or the configured target is reached.

It supports interactive runs, detached background runs, resume/status/stop, and
non-interactive `codex exec` automation.

## Workspace Rule

Results directory stays in the launch context. If Codex starts inside a Git
repo, the default workspace root is that repo root; otherwise the default
workspace root is the current launch directory. Codex should not silently widen
the workspace root to a parent directory just because sibling repos or old
artifacts exist.

The confirmation summary should always show the chosen Results directory:

```text
Results directory: ./autoresearch-results/
```

## Windows Quick Install

Prerequisites:

- Codex Windows client / Codex CLI installed and signed in.
- Git in `PATH`.
- Python 3 in `PATH`.
- PowerShell.

Install from a fresh clone:

```powershell
git clone https://github.com/yourskenny/codex-autoresearch-windows-skill.git
cd codex-autoresearch-windows-skill
powershell -ExecutionPolicy Bypass -File .\scripts\install_windows_skill.ps1
```

Restart Codex after installation.

Verify that Codex can discover it:

```powershell
codex debug prompt-input '$codex-autoresearch test'
```

You should see `codex-autoresearch` in the available skills list.

## One-Line Install From GitHub

For a disposable install directory:

```powershell
$src = Join-Path $env:TEMP "codex-autoresearch-windows-skill"; Remove-Item -Recurse -Force $src -ErrorAction SilentlyContinue; git clone https://github.com/yourskenny/codex-autoresearch-windows-skill.git $src; powershell -ExecutionPolicy Bypass -File "$src\scripts\install_windows_skill.ps1"
```

## Use It

Open Codex in any Git repo and say:

```text
$codex-autoresearch
I want to reduce the failing tests to zero.
```

For background runs:

```text
$codex-autoresearch
Improve type safety overnight. Use background mode after confirmation.
```

For CI-style automation:

```powershell
codex exec --dangerously-bypass-approvals-and-sandbox -C C:\path\to\repo @"
$codex-autoresearch
Mode: exec
Goal: Reduce type errors
Scope: src/**/*.ts
Metric: type error count
Direction: lower
Verify: npm run typecheck
Iterations: 10
"@
```

## What Gets Installed

The installer copies this repo to:

```text
%USERPROFILE%\.codex\skills\codex-autoresearch
```

It also installs/repairs the managed user-level autoresearch hooks through:

```powershell
python "%USERPROFILE%\.codex\skills\codex-autoresearch\scripts\autoresearch_hooks_ctl.py" install
```

The hooks help future foreground/background sessions recover the active run
context safely. They are managed under `%USERPROFILE%\.codex\autoresearch-hooks`.

## Update

From the cloned source directory:

```powershell
git pull
powershell -ExecutionPolicy Bypass -File .\scripts\install_windows_skill.ps1
```

Restart Codex after updating.

## Migrate To Another Windows Machine

1. Install Codex, Git, and Python 3.
2. Clone this repository.
3. Run `scripts\install_windows_skill.ps1`.
4. Restart Codex.
5. Run `codex debug prompt-input '$codex-autoresearch test'`.

No build step is required.

## Repository Layout

```text
SKILL.md                  Skill entrypoint loaded by Codex
agents/openai.yaml        Codex UI metadata and default prompt
references/               Mode workflows and runtime protocols
scripts/                  Helper scripts for state, hooks, runtime, and checks
tests/                    Python unit tests and smoke invariant checks
docs/                     Longer operator docs and Windows port notes
```

## Verification

Run the full test suite:

```powershell
python -m unittest discover -s tests -q
```

Check hook readiness:

```powershell
python .\scripts\autoresearch_hooks_ctl.py status
```

Expected:

```json
{"ready_for_future_sessions": true}
```

## Notes For Windows Users

- Use `codex.cmd` or `codex` from PowerShell. The helper code prefers PATHEXT
  executables on Windows so it avoids extensionless shim permission errors.
- Bash e2e scripts are kept for upstream compatibility, but Windows users should
  rely on the PowerShell installer and Python test suite.
- Exec-mode audit artifacts are intentional output, not junk:
  `autoresearch-results/results.tsv`, `context.json`, the repo git-local pointer,
  and `.prev` archives must remain after a run.

## Upstream

This Windows client skill distribution is based on Codex Autoresearch by
`leo-lilinxiao`, with Windows-specific runtime, install, and verification work.

## License

MIT. See [LICENSE](LICENSE).
