# bop GitHub Action

AI-powered code review for pull requests using [bop](https://github.com/delightfulhammers/bop).

## Usage

```yaml
name: Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: delightfulhammers/bop-github-action@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `base-ref` | Base branch to compare against | `main` |
| `post-findings` | Post findings as PR review comments | `true` |
| `reviewers` | Comma-separated list of reviewers | (all configured) |
| `block-threshold` | Minimum severity to request changes (`critical`, `high`, `medium`, `low`, `none`) | `none` |
| `fail-on-findings` | Fail the action if findings exceed block-threshold | `false` |
| `always-block-categories` | Categories that always block (e.g., `security,bug`) | |
| `version` | bop version to install | `latest` |
| `github-token` | GitHub token for posting comments | `${{ github.token }}` |

## Outputs

| Output | Description |
|--------|-------------|
| `findings-count` | Total number of findings |
| `critical-count` | Number of critical severity findings |
| `high-count` | Number of high severity findings |
| `medium-count` | Number of medium severity findings |
| `low-count` | Number of low severity findings |

## Examples

### Block on Critical Findings

```yaml
- uses: delightfulhammers/bop-github-action@main
  with:
    block-threshold: critical
    fail-on-findings: true
```

### Use Specific Reviewers

```yaml
- uses: delightfulhammers/bop-github-action@main
  with:
    reviewers: security,architecture
```

### Always Block Security Issues

```yaml
- uses: delightfulhammers/bop-github-action@main
  with:
    always-block-categories: security
```

## Requirements

- The action must run on `pull_request` events
- Repository must be checked out with `fetch-depth: 0` for full git history
- `pull-requests: write` permission is required to post review comments

## License

MIT
