---
name: github-actions-setup
description: Configure GitHub Actions workflows using self-hosted runners, available secrets, and CI/CD best practices. Use when creating or updating GitHub Actions workflows, selecting runners, configuring secrets, setting up deployment pipelines, or when the user asks about CI/CD configuration, self-hosted runners, or GitHub Actions setup.
---

# GitHub Actions Setup

Create workflow files under `.github/workflows/` in your repository.

## Runner selection

**Always specify `self-hosted`** to use our runners:

```yaml
jobs:
  build:
    runs-on: [self-hosted, linux, x64, v1]      # Linux 16 GB RAM
    # OR
    runs-on: [self-hosted, macOS, ARM64, v2]     # macOS
```

### Linux runners

- Based on Ubuntu 22.04
- Pre-installed: Docker, kubectl, helm, helmfile, GCloud CLI, Blackbox, yarn, pnpm, asdf
- For language SDKs (Go, Java), use official setup actions.
- Stable public IP: add `uk` or `tw` geo tag.

### macOS runners

- Versions: macOS 14.6.1 (tw) / macOS 26.2 (hk)
- Pre-installed: Homebrew, Xcode, Blackbox, cocoapods, carthage
- For language SDKs, use official setup actions.

## Available secrets

| Secret              | Available On  | How to access                           |
| ------------------- | ------------- | --------------------------------------- |
| GCP service account | Linux         | `$OURSKY_KUBE_SERVICE_ACCOUNT_KEY_FILE` |
| Faseng GPG key      | Linux + macOS | Auto-imported at job start              |
| Sentry auth token   | All           | `$SENTRY_AUTH_TOKEN`                    |
| App Store Connect   | macOS         | `$APP_STORE_CONNECT_API_KEY_FILE`       |
| Oursky certs        | macOS         | Auto-imported to keychain               |
| Google Play key     | macOS         | `$FASTLANE_GOOGLE_PLAY_KEY_FILE`        |

> Secrets are **not** available in Pull Request CI runs.

## Key best practices

- **Avoid artifacts** (costs money); use a short expiry (1 day) if artifacts are necessary.
- Do not use default artifact expiry — always set `retention-days`.

## Multi-line secrets

```yaml
- name: Decode secret
  run: echo -e "$MY_SECRET" | some-command
  env:
    MY_SECRET: ${{ secrets.MY_MULTILINE_SECRET }}
```
