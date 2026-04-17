# pqs-action

[![GitHub Marketplace](https://img.shields.io/badge/GitHub_Marketplace-PQS_Check-2ea44f?logo=github)](https://github.com/marketplace/actions/pqs-check)

Score and gate AI prompts in GitHub Actions. Fail pull requests when a prompt drops below your quality threshold.

## Add this to your workflow

```yaml
- uses: OnChainAIIntel/pqs-action@v0.1
  with:
    api-key: ${{ secrets.PQS_API_KEY }}
    dir: ./prompts        # optional (default: prompts)
    threshold: 60         # optional (default: 60)
    format: markdown      # optional (default: markdown)
```

That's it. On every PR, the action scores every `.md` / `.txt` / `.prompt` file under `./prompts` on 8 dimensions. Any file below `60/80` fails the build.

## Getting your API key

1. Sign up at [https://pqs.onchainintel.net/pricing](https://pqs.onchainintel.net/pricing) — Solo, Team, and Business tiers are supported.
2. Copy your API key from your [account page](https://pqs.onchainintel.net/account) — it starts with `pqs_live_`.
3. Add it to your repo's secrets as **`PQS_API_KEY`** (Settings → Secrets and variables → Actions → New repository secret).

## Why PQS?

**A Quality Gate For Prompts.**
Before they break production. Paste yours below and see.

This Action runs `pqs check` on every pull request so a prompt that regresses from a B to a D fails the build before it merges.

## Inputs

| Name          | Required | Default     | Description                                                                                  |
|---------------|----------|-------------|----------------------------------------------------------------------------------------------|
| `api-key`     | yes      | —           | PQS API key. Must start with `pqs_live_`. Store as a repo secret.                            |
| `dir`         | no       | `prompts`   | Directory scanned recursively for `.md` / `.txt` / `.prompt` files.                          |
| `file`        | no       | `''`        | Single file to score. Takes precedence over `dir` when both are supplied.                    |
| `threshold`   | no       | `60`        | Minimum total score out of 80. Any prompt below this fails the job.                          |
| `format`      | no       | `markdown`  | Output format written to the job log: `text`, `json`, or `markdown`.                         |
| `cli-version` | no       | `latest`    | Pin a specific `pqs-quality` version, e.g. `0.1.0`. Defaults to the latest published release. |

## Outputs

| Name        | Description                                                                         |
|-------------|-------------------------------------------------------------------------------------|
| `exit-code` | Numeric exit code from `pqs check`: `0` pass, `1` threshold fail, `2` api/network, `3` auth. |

Use the output to branch downstream steps:

```yaml
- uses: OnChainAIIntel/pqs-action@v0.1
  id: pqs
  continue-on-error: true
  with:
    api-key: ${{ secrets.PQS_API_KEY }}
- if: steps.pqs.outputs.exit-code == '1'
  run: echo "One or more prompts below threshold."
```

## Examples

### 1. Directory scan on every PR

The most common setup: gate a `prompts/` directory and fail the PR when anything regresses below `60/80`.

```yaml
name: PQS check

on:
  pull_request:
    paths:
      - "prompts/**"

jobs:
  pqs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: OnChainAIIntel/pqs-action@v0.1
        with:
          api-key: ${{ secrets.PQS_API_KEY }}
          dir: ./prompts
          threshold: 60
```

### 2. Single file

Scoring one high-value system prompt — for example, the one that drives your production agent.

```yaml
name: Gate system prompt

on:
  pull_request:
    paths:
      - "src/agent/system.md"

jobs:
  pqs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: OnChainAIIntel/pqs-action@v0.1
        with:
          api-key: ${{ secrets.PQS_API_KEY }}
          file: src/agent/system.md
          threshold: 70   # stricter gate on your single most-important prompt
```

### 3. Custom threshold tuning

Start permissive, then ratchet up as your prompt hygiene improves.

```yaml
jobs:
  pqs-baseline:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: OnChainAIIntel/pqs-action@v0.1
        with:
          api-key: ${{ secrets.PQS_API_KEY }}
          dir: ./prompts
          threshold: 50   # initial baseline: block only F-grade prompts

  pqs-strict:
    runs-on: ubuntu-latest
    if: github.base_ref == 'main'
    steps:
      - uses: actions/checkout@v4
      - uses: OnChainAIIntel/pqs-action@v0.1
        with:
          api-key: ${{ secrets.PQS_API_KEY }}
          file: src/agent/system.md
          threshold: 70   # only stricter on critical path
```

Recommended thresholds:

- `50` — block the genuinely broken. Good first-time setting.
- `60` — production-grade. Requires a B.
- `70` — elite. Use on a few critical prompts, not everything.

### 4. Post the results as a PR comment

This Action doesn't expose the markdown report as an output in v0.1. To post it as a PR comment, invoke the `pqs-quality` CLI directly — same scoring, just with stdout capture.

```yaml
name: PQS check with PR comment

on:
  pull_request:

jobs:
  pqs:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm install -g pqs-quality@latest

      - name: Score prompts
        id: score
        continue-on-error: true
        env:
          PQS_API_KEY: ${{ secrets.PQS_API_KEY }}
        run: |
          set +e
          {
            echo 'body<<EOF'
            pqs check --dir ./prompts --threshold 60 --format markdown
            echo 'EOF'
          } >> "$GITHUB_OUTPUT"
          echo "exit-code=$?" >> "$GITHUB_OUTPUT"

      - name: Post comment
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `${{ steps.score.outputs.body }}`
            });

      - name: Fail if any prompt below threshold
        if: steps.score.outputs.exit-code != '0'
        run: exit 1
```

Exposing the markdown report as a direct Action output is planned for v0.2.

## Links

- CLI: [pqs-quality on npm](https://www.npmjs.com/package/pqs-quality) · [source](https://github.com/OnChainAIIntel/pqs-quality)
- Website & pricing: [pqs.onchainintel.net/pricing](https://pqs.onchainintel.net/pricing)
- Bugs: [github.com/OnChainAIIntel/pqs-action/issues](https://github.com/OnChainAIIntel/pqs-action/issues)

## License

MIT © Ken Burbary
