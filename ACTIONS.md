# GitHub Actions setup

Two reusable workflows live in `.github/workflows/` of each standardized public repo:

| Workflow | Purpose | Trigger |
|---|---|---|
| `validate-profile.yml` | Enforces description length, ≥3 topics, no internal-leak phrases | push + manual |
| `scan-secrets.yml` | gitleaks on push (gate); TruffleHog weekly (deep audit) | push + weekly Sun 03:00 UTC + manual |

## Smoke test

After the workflows are merged, run both manually on `karmiphuc/karmiphuc` to confirm wiring:

1. `https://github.com/karmiphuc/karmiphuc/actions` → click **Validate profile metadata** → **Run workflow**.
2. Same for **Scan for leaked secrets**.
3. Both should turn green ✅ within ~30s.

The validator should pass on this repo (description "Personal README profile for karmiphuc" is short — see known issue below).

> ⚠️ **Known issue:** the profile repo's description "Personal README profile for karmiphuc" is 36 chars but only barely passes the no-leak-phrase rule. If you change it, keep it ≥20 chars.

## Hybrid dispatch (private homelab)

To trigger any workflow from the private `homelab-agentic` repo, drop `homelab-receiver.yml` into `.github/workflows/` there, set a `KARMIPHUC_PAT` secret (classic PAT with `workflow` scope), then:

```bash
gh api repos/karmiphuc/homelab-agentic/dispatches \
  -f event_type=profile_housekeeping_run
```

The receiver maps `event_type → gh workflow run` against the public repo.

## Cost

- **Public repos:** unlimited minutes, free. These workflows run <30s each.
- **Private homelab dispatch:** ~1–2 min/month, well inside the 2,000 min free tier.