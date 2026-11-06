# Changelog

## joy-headless-redeem-cart 1.0 — 2026-11-06

- New skill: add the Redeem in Cart block (`<JoyRedeemCart/>`, `@joy-loyalty/hydrogen` 0.3.0+) to a Hydrogen cart drawer or cart page — prerequisites gate, mount with a dedicated cart fetcher, cart action that returns the cart, Default/Modern cart fragment by Storefront API version, client cart store sync, CSP, verify checklist and troubleshooting.

## joy-headless-install 1.2 — 2026-11-06

- Points to `joy-headless-redeem-cart` for the cart drawer block; notes the 0.3.0 requirement.

## joy-headless-install 1.1 — 2026-10-19

- Prerequisites hard-stop in §0: stop before touching code when the storefront, Joy app, Customer Account API or Headless secret is missing.
- Full frontmatter: `when-to-use`, `argument-hint`, `compatibility`, `metadata.version`.
- Never stub `customerAccount` (§5); session-cookie guard and `Vary: Accept-Encoding, Cookie` for full-page cache (§10); sign-up points verify step (§12) and symptom row (§13).

## joy-headless-install 1.0

- Initial skill.
