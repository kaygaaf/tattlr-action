# tattlr scan action

Runs [tattlr](https://github.com/kaygaaf/tattlr) in a GitHub Actions job and tells you what your coding agent actually did there.

If a workflow runs Claude Code, Codex, Gemini CLI, OpenClaw or Cursor's agent, those tools leave session logs in the runner's home folder. This action reads them after the agent is done and flags things like reading secrets, `curl | sh`, force pushes, sending env vars to a webhook, or editing its own config. You get the rap sheet in the job summary, a SARIF file for code scanning, and a failed step if you want one.

Nothing is uploaded anywhere unless you turn on `upload-sarif`, and then only to your own repo's code scanning.

## Usage

Put it after the step that runs your agent, in the same job:

```yaml
name: agent

on:
  issue_comment:
    types: [created]

jobs:
  agent:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      security-events: write   # only needed for upload-sarif
    steps:
      - uses: actions/checkout@v4

      # Your agent step goes here, for example anthropics/claude-code-action
      # or a plain `codex exec` / `gemini -p` run.

      - uses: kaygaaf/tattlr-action@v1
        if: always()           # also check runs where the agent step failed
        with:
          fail-on: high
          upload-sarif: true
```

A job summary looks like this:

> ## tattlr rap sheet
>
> 1 session, 8 tool calls, 5 flagged calls, Oct 22 2026.
>
> | Agent | Sessions | Calls | Flagged | Crit | High | Med | Low | Score | Verdict |
> |---|--:|--:|--:|--:|--:|--:|--:|--:|---|
> | codex | 1 | 8 | 5 | 1 | 4 | 4 | 0 | 69 | menace |

followed by the top offenses and the worst moments with the exact commands, secrets masked.

## Inputs

| Input | Default | What it does |
|---|---|---|
| `version` | `latest` | tattlr version to install, like `0.2.0`. Needs 0.2.0 or newer. |
| `path` | runner home | Home folder to look for agent logs in. |
| `since` | | Only activity after this: `2h`, `1d`, `2026-10-01`. |
| `agent` | | Only these agents, comma separated: `claude`, `codex`, `gemini`, `openclaw`, `cursor`. |
| `project` | | Only sessions whose project path contains this. |
| `fail-on` | | Fail the step at this severity or worse: `low`, `medium`, `high`, `critical`. Empty never fails. |
| `format` | `text` | What goes to the job log: `text`, `json`, `sarif` or `markdown`. |
| `summary` | `true` | Write the rap sheet to the job summary. |
| `sarif-file` | `$RUNNER_TEMP/tattlr.sarif` | Where the SARIF report goes. |
| `upload-sarif` | `false` | Send the SARIF report to code scanning. Needs `security-events: write`. |
| `category` | `tattlr` | Code scanning category, for repos with more than one tattlr run. |

## Outputs

| Output | What it is |
|---|---|
| `findings` | Number of findings. |
| `exit-code` | tattlr's exit code. `1` means something hit `fail-on`. |
| `sarif-file` | Path to the SARIF report. |
| `version` | The tattlr version that ran. |

## How it installs tattlr

It downloads the release archive for the runner's OS and CPU from [the tattlr releases](https://github.com/kaygaaf/tattlr/releases), checks its sha256 against `checksums.txt` from the same release, and stops if they don't match. Linux, macOS and Windows runners work, x64 and arm64. Pin `version` if you want the same binary every run.

## Notes

- The agent has to run in the same job, since each job gets a fresh runner.
- Findings for files inside your checkout link to that file in code scanning. Everything else (a read of `~/.aws/credentials`, say) points at the agent's session log.
- On your own machine you can do a lot more with tattlr, like block these calls live. See the [tattlr README](https://github.com/kaygaaf/tattlr).

## Support

If this caught something nasty, you can buy me a coffee at [ko-fi.com/kayorama](https://ko-fi.com/kayorama).

## License

MIT
