# bop GitHub Action

AI-powered code review for pull requests using [bop](https://github.com/delightfulhammers/bop).

## Private Beta

bop is currently in **private beta**. To use this action, you must:

1. **Accept a beta invitation** — You'll receive an email with an invitation link
2. **Provide your own API keys** — bop uses multiple LLM providers (see setup below)
3. **Use personal repositories only** — Organization repositories are not yet supported

## Quick Start

### 1. Accept Your Invitation

Click the link in your beta invitation email to create your account. This automatically sets up your authentication for the GitHub Action.

### 2. Get API Keys

bop uses three LLM providers for diverse review perspectives. You'll need API keys for each:

| Provider | Get API Key | Purpose |
|----------|-------------|---------|
| **Anthropic** | [console.anthropic.com](https://console.anthropic.com/) | Security analysis (Claude) |
| **OpenAI** | [platform.openai.com](https://platform.openai.com/) | Architecture review (GPT) |
| **Google AI** | [aistudio.google.com](https://aistudio.google.com/) | Performance analysis (Gemini) |

### 3. Add Secrets to Your Repository

Go to your repository **Settings → Secrets and variables → Actions** and add:

| Secret Name | Value |
|-------------|-------|
| `ANTHROPIC_API_KEY` | Your Anthropic API key |
| `OPENAI_API_KEY` | Your OpenAI API key |
| `GEMINI_API_KEY` | Your Google AI API key |

### 4. Create the Workflow

Create `.github/workflows/code-review.yml`:

```yaml
name: Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    # IMPORTANT: Skip fork PRs - secrets are not available to forks
    if: github.event.pull_request.head.repo.full_name == github.repository
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      id-token: write  # Required for bop authentication
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history needed for diff analysis

      - uses: delightfulhammers/bop-github-action@main
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 5. Open a Pull Request

Create a PR in your repository. bop will automatically review it and post findings as inline comments.

---

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
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
    OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
    GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
  with:
    block-threshold: critical
    fail-on-findings: true
```

### Use Specific Reviewers

```yaml
- uses: delightfulhammers/bop-github-action@main
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
    OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
    GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
  with:
    reviewers: security,architecture
```

### Always Block Security Issues

```yaml
- uses: delightfulhammers/bop-github-action@main
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
    OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
    GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
  with:
    always-block-categories: security
```

## Requirements

- **Beta access** — You must have accepted a bop beta invitation
- **Personal repository** — Organization repositories are not yet supported during beta
- **API keys** — All three LLM provider keys must be configured as secrets
- **Permissions** — `contents: read`, `pull-requests: write`, and `id-token: write`
- **Full checkout** — Use `fetch-depth: 0` for accurate diff analysis

## Troubleshooting

### "Authentication failed" or "Tenant not found"

- Ensure you've accepted your beta invitation at [app.delightfulhammers.com](https://app.delightfulhammers.com)
- Verify the repository is under your personal GitHub account, not an organization
- Check that `id-token: write` permission is set

### "API key not configured" or empty reviews

- Verify all three API keys are set in repository secrets
- Check the secret names match exactly: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`
- Ensure the `env:` block is present in your workflow

### Fork PRs show no review

This is expected. For security, GitHub does not expose secrets to workflows triggered by fork PRs. The `if:` condition in the example workflow skips fork PRs intentionally.

## Feedback

We want to hear from you! Report issues or share feedback:

- **CLI:** Run `bop feedback` in your terminal
- **Email:** [support@delightfulhammers.com](mailto:support@delightfulhammers.com)

## Beta Limitations

During the private beta:

- **Personal repos only** — `github.com/your-username/*` works; organization repos do not
- **BYOK required** — You must provide your own LLM API keys
- **Temporary** — Beta access will transition to paid plans (30+ days notice)

## License

MIT
