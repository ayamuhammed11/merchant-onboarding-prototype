# Merchant Onboarding — UI prototype

UI prototype for the merchant onboarding flow ("New Application" in the main
Kashier back-office nav) and the document-rules sub-flow it depends on, built
as static HTML pages following the **Kashier Design System**.

Split out from [`Kashierprototypes`](https://github.com/Kashier-payments/Kashierprototypes)
so this flow can be iterated on independently.

## Pages

- `merchant-onboarding.html` — entry point / application list ("New Application")
- `business-info.html`, `account-details.html`, `merchant-legal-data.html` — onboarding steps
- `documents-upload.html` — document upload step
- `document-types.html`, `document-rules.html`, `document-rules-create.html` — document type/rule configuration
- `on-hold.html` — applications on hold
- `maker-checker-tasks.html`, `maker-checker-workflows.html`, `maker-checker-logs.html` — maker-checker review flow

## Style guide

`kashier-styleguide/SKILL.md` is the design-system rulebook (tokens, colors,
spacing, component patterns), copied from `Kashierprototypes` and installed
as a Claude Code skill:

```powershell
New-Item -ItemType Directory -Force ~\.claude\skills\kashier-styleguide
Copy-Item kashier-styleguide\SKILL.md ~\.claude\skills\kashier-styleguide\SKILL.md
```

## Status

Local-only for now — no GitHub remote has been created yet.
