# joy-loyalty-hydrogen-skills

Agent skills for integrating [Joy Loyalty](https://joy.so) into headless Shopify storefronts (Hydrogen, Weaverse).

Works with **Claude Code** (default), **OpenAI Codex**, **Cursor**, **Grok** and any agent supported by the [Skills CLI](https://github.com/vercel-labs/skills). Agent procedures live in each skill's `SKILL.md`. This file is for humans.

## Skills

| Skill | What it does |
|---|---|
| [`joy-headless-install`](skills/joy-headless-install/SKILL.md) | Installs Joy into a Shopify Hydrogen storefront (optionally with Weaverse): env vars, root loader, CSP, personalized cache, `/api/joy-token`, `<JoyWidget/>`, cart wiring, Loyalty Page, Weaverse blocks, Oxygen deploy and verification. Also troubleshoots an existing integration. |
| [`joy-headless-redeem-cart`](skills/joy-headless-redeem-cart/SKILL.md) | Adds the **Redeem in Cart** block (`<JoyRedeemCart/>`, `@joy-loyalty/hydrogen` 0.3.0+) to your cart drawer or cart page: dedicated cart fetcher, cart action that returns the cart, cart fragment fields for your Storefront API version, client cart store sync and verification. Run it after `joy-headless-install`. |

## Prerequisites

- A **Shopify Hydrogen** storefront (React Router 7, `@shopify/hydrogen` ≥ 2025.5), optionally built on [Weaverse](https://weaverse.io) (e.g. the [Pilot](https://github.com/Weaverse/pilot) theme).
- **Node.js** ≥ 18 and npm / pnpm / yarn.
- The **Joy Loyalty** app installed on the store ([Shopify App Store](https://apps.shopify.com/joyio)).
- **Customer Account API** set up in Hydrogen, so signed-in customers are recognised.
- From Joy admin → **Settings → Manage keys → Headless secret**: the `hs_…` secret for `JOY_API_SECRET`. Not the REST "Secret key".

Liquid (Online Store theme) stores do not need these skills: Joy works there through the app embed.

## Install

Run in your Hydrogen project root. The CLI detects your agents and installs for each:

```bash
npx skills add tinhnv2001/joy-loyalty-hydrogen-skills
```

Preview without installing:

```bash
npx skills add tinhnv2001/joy-loyalty-hydrogen-skills --list
```

| Target | Command |
|---|---|
| This project, Claude Code **(default)** | `npx skills add tinhnv2001/joy-loyalty-hydrogen-skills -a claude-code -y` |
| All projects, Claude Code | `npx skills add tinhnv2001/joy-loyalty-hydrogen-skills -g -a claude-code -y` |
| Codex | `npx skills add tinhnv2001/joy-loyalty-hydrogen-skills -a codex -y` |
| Cursor | `npx skills add tinhnv2001/joy-loyalty-hydrogen-skills -a cursor -y` |
| Several agents | `npx skills add tinhnv2001/joy-loyalty-hydrogen-skills -a claude-code -a codex -a cursor -y` |
| One skill only | `npx skills add tinhnv2001/joy-loyalty-hydrogen-skills --skill joy-headless-redeem-cart` |
| Interactive | `npx skills add tinhnv2001/joy-loyalty-hydrogen-skills` |

`-g` installs user-wide, `-y` skips prompts. Add `--copy` if the machine cannot symlink.

### Update / remove / list

```bash
npx skills update -y
npx skills remove joy-headless-install joy-headless-redeem-cart -a claude-code -y
npx skills ls -a claude-code
```

### git clone (no Skills CLI)

```bash
git clone https://github.com/tinhnv2001/joy-loyalty-hydrogen-skills.git /tmp/joy-loyalty-hydrogen-skills
cp -R /tmp/joy-loyalty-hydrogen-skills/skills/joy-headless-install /tmp/joy-loyalty-hydrogen-skills/skills/joy-headless-redeem-cart .claude/skills/
```

Each `SKILL.md` must end up at `.claude/skills/<skill>/SKILL.md` (or the equivalent folder for your agent).

## Run

```
/joy-headless-install your-store.myshopify.com
```

Or just ask: *"Install Joy Loyalty on this Hydrogen storefront."* The agent picks the skill from its description.

Add Redeem in Cart to the cart drawer (after Joy is installed):

```
/joy-headless-redeem-cart
```

Or: *"Add the Joy Redeem in Cart block to my cart drawer."*

The skill checks prerequisites first and stops if something is missing. It asks you to put the Headless secret in `.env` / Oxygen environment variables yourself and never writes it into code or commits.

## Common pitfalls the skill prevents

- Using the REST "Secret key" instead of the Headless secret → `/api/joy-token` 401 loops.
- Prefixing env vars with `PUBLIC_` → the secret ships to the browser.
- Stubbing `customerAccount` to keep pages cacheable → new members never get sign-up points.
- Full-page caching a signed-in document → one customer's points shown to another.
- Missing CSP entries → widget stuck on the skeleton.

See §13 of [`SKILL.md`](skills/joy-headless-install/SKILL.md) for the full symptom → fix table.

## Docs

- Merchant guide: https://help.joy.so/settings/developers/headless/
- Redeem in Cart (Hydrogen): https://help.joy.so/on-site-content/cart-drawer/redeem-in-cart-headless/
- Developer docs: https://devdocs.joy.so/joy-headless-api/overview/ · https://devdocs.joy.so/joy-headless-api/redeem-in-cart/
- Packages: [`@joy-loyalty/hydrogen`](https://www.npmjs.com/package/@joy-loyalty/hydrogen), [`@joy-loyalty/sdk`](https://www.npmjs.com/package/@joy-loyalty/sdk), [`@joy-loyalty/weaverse-hydrogen`](https://www.npmjs.com/package/@joy-loyalty/weaverse-hydrogen)

## Changelog

See [CHANGELOG.md](CHANGELOG.md).
