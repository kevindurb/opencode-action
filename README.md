# OpenCode Action

A composite GitHub Action that installs [OpenCode](https://opencode.ai) and runs it
non-interactively with [`opencode run`](https://opencode.ai/docs/cli/), so an agent can
work on your repository inside a workflow.

## Usage

```yaml
permissions:
  contents: write
  pull-requests: write

jobs:
  opencode:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5

      - uses: kevindurb/opencode-action@v1
        with:
          prompt: Add a "Project layout" section to the README.
          model: openrouter/anthropic/claude-sonnet-5.5
        env:
          OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY }}
```

A more complete workflow that commits the agent's changes and opens a pull request is in
[`examples/opencode.yml`](examples/opencode.yml).

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `prompt` | yes | | Prompt to send to OpenCode. |
| `model` | no | config default | Model as `provider/model#variant`, e.g. `openrouter/anthropic/claude-sonnet-5.5`. |
| `agent` | no | config default | Agent to use, e.g. `build` or `plan`. |
| `version` | no | `latest` | OpenCode version to install, or `none` to use an `opencode` already on `PATH`. |
| `working-directory` | no | `.` | Directory to run OpenCode in, relative to the workspace. |
| `auto` | no | `true` | Auto-approve permissions that are not explicitly denied. Needed for unattended tool use. |
| `args` | no | | Extra arguments appended to `opencode run`. |

## Credentials

OpenCode reads provider credentials from the environment. Pass whichever provider key you
use to the step, for example:

```yaml
- uses: kevindurb/opencode-action@v1
  with:
    prompt: Explain this repository.
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

Store the key as a repository or organization secret. Never expose it to workflows that run
untrusted code, such as `pull_request_target`.

## Configuration

OpenCode reads `opencode.json` or `opencode.jsonc` from the repository (or any parent
directory), so you can commit a project configuration to set the default model, agent,
permissions, and so on:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "model": "openrouter/anthropic/claude-sonnet-5.5",
}
```

## Notes

- On a fresh runner the action installs the CLI with OpenCode's official install script and
  puts it on `PATH` for the remaining steps in the job.
- `--auto` keeps the agent from blocking on permission prompts. Explicitly deny anything you
  do not want it to touch in `opencode.jsonc`, and scope the job's `permissions` as tightly as
  possible.
- Changes made by the agent are ordinary working-tree changes; commit and push them in a
  later step if you want them to land.
