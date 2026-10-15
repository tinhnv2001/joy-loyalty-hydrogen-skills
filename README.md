# joy-loyalty-skills

Agent skills for integrating [Joy Loyalty](https://joy.so) into headless Shopify storefronts.

## Install

Run in your Hydrogen project root:

```bash
npx skills add tinhnv2001/joy-loyalty-skills
```

The command detects which coding agents you use (Claude Code, Cursor, Codex, …) and installs the skills for each of them.

## Skills

| Skill | What it does |
|---|---|
| [`joy-headless-install`](skills/joy-headless-install/SKILL.md) | Installs Joy into a Shopify Hydrogen storefront (optionally with Weaverse): env vars, root loader, CSP, `/api/joy-token`, `<JoyWidget/>`, cart wiring, Loyalty Page, Weaverse blocks, Oxygen deploy and verification. |

Then ask your agent: *"Install Joy Loyalty on this Hydrogen storefront."*

## Docs

- Merchant guide: https://help.joy.so/settings/developers/headless/
- Developer docs: https://devdocs.joy.so/joy-headless-api/overview/
- Packages: [`@joy-loyalty/hydrogen`](https://www.npmjs.com/package/@joy-loyalty/hydrogen), [`@joy-loyalty/sdk`](https://www.npmjs.com/package/@joy-loyalty/sdk), [`@joy-loyalty/weaverse-hydrogen`](https://www.npmjs.com/package/@joy-loyalty/weaverse-hydrogen)
