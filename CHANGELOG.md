# Changelog

## joy-headless-install 1.1 — 2026-10-19

- Prerequisites hard-stop in §0: stop before touching code when the storefront, Joy app, Customer Account API or Headless secret is missing.
- Full frontmatter: `when-to-use`, `argument-hint`, `compatibility`, `metadata.version`.
- Never stub `customerAccount` (§5); session-cookie guard and `Vary: Accept-Encoding, Cookie` for full-page cache (§10); sign-up points verify step (§12) and symptom row (§13).

## joy-headless-install 1.0

- Initial skill.
