# CLAUDE.md — taskhive

This repo follows the portfolio cooperation rules in [`../platform/CLAUDE.md`](../platform/CLAUDE.md) (a private sibling repo): the PA ticket flow, red then green, ADRs in `docs/adr/`, Terraform via CI only, and the autonomy tiers. Read them before starting any ticket. The rules below only add to them.

## Jira

- Project **PA** (`Portfolio Apps`, https://lukaszkot03.atlassian.net), label **`taskhive`**.
- Epics PA-86 to PA-97. Ticket summaries carry no code prefix; dependencies are "Blocks" links.

## Repo-specific rules

- **This repo is public.** Never commit secrets, personal data, or tenant, subscription or other cloud identifiers. Those belong in Key Vault or GitHub Actions variables and secrets.
- **Requirements:** `docs/design-spec.md` is a snapshot of the TaskHive spec Claude Doc, which stays the source of truth. Change the doc first, then re-export it here in the same PR as the work that depends on it.
- `main` is protected: every change goes through a PR.
