# GitHub Actions setup

Two reusable workflows + a gitleaks config live in each of the 12 standardized public repos:

| File | Purpose | Trigger |
|---|---|---|
| `.github/workflows/validate-profile.yml` | Description ≥20 chars, capitalized, no internal-leak phrases; topics ≥3, no leaks | push to `main`/`master`/`finetuning` + manual |
| `.github/workflows/scan-secrets.yml` | gitleaks on push (gate); TruffleHog weekly (deep audit, opens issue on verified finding) | push + weekly Sun 03:00 UTC + manual |
| `.gitleaks.toml` | Local allowlist for known-noise paths (lockfiles, binaries, license files) | picked up automatically by `gitleaks-action` |

The `karmiphuc/karmiphuc` profile repo **does not** carry these workflows — it's a thin profile shell, not a code repo. The 12 repos that do:

`claude-code-mastery`, `openclaw-docs-skill`, `reddit-json`, `gdocs-cli`, `dotfiles`, `learning-journey`, `lifehack-scripts`, `karmiphuc.github.io`, `facebook-favorite-app`, `fb-friends-bookmarklet`, `gpt-2-simple`, `gpt-2`.

## Smoke test

Pick any of the 12 (e.g. `reddit-json`):

```bash
gh workflow run validate-profile.yml --repo karmiphuc/reddit-json
gh workflow run scan-secrets.yml   --repo karmiphuc/reddit-json
```

Both should turn green ✅ within ~30s. The validator proves correct by failing if you set the repo description to e.g. `"internal corp tool"` (lowercase + internal-leak phrase).

## What triggers what

- **Push** to `main` / `master` / `finetuning` runs both workflows. Fail = push is blocked.
- **Manual** (`workflow_dispatch`) lets you re-run from the Actions tab.
- **Weekly cron** runs only `trufflehog-audit` (the deep scanner). Findings open an issue, not a PR comment.

## Repo-local allowlists

`learning-journey/.gitleaks.toml` excludes `2018-06-02Amazing_History_Facts.htm` — an archived web scrape containing public recaptcha sitekeys (not credentials). If gitleaks flags a real-looking string that you know is benign, add it to that repo's `.gitleaks.toml` under `[allowlist].paths` (regex) or `[allowlist].regexes` (string pattern).

## Hybrid dispatch (private homelab)

To trigger any workflow from the private `homelab-agentic` repo, drop `homelab-receiver.yml` from this directory into `homelab-agentic/.github/workflows/`, set a `KARMIPHUC_PAT` secret (classic PAT with `workflow` scope), then:

```bash
gh api repos/karmiphuc/homelab-agentic/dispatches \
  -f event_type=profile_housekeeping_run
```

The receiver maps `event_type → gh workflow run` against the public repo.

## Cost

- **Public repos:** unlimited minutes, free. These workflows run <30s each.
- **Private homelab dispatch:** ~1–2 min/month, well inside the 2,000 min free tier.
- **Total so far (smoke tests + iterations across all 12 repos):** ~25 min over the setup session — well under any cap.