---
name: joy-headless-redeem-cart
description: >
  Add the Joy Loyalty "Redeem in Cart" block (<JoyRedeemCart/> from
  @joy-loyalty/hydrogen 0.3.0+) to a Shopify Hydrogen cart drawer or cart page:
  package upgrade, mount with a dedicated cart fetcher, cart action that returns
  the cart, cart fragment fields for the Storefront API version, client cart
  store sync, verification. Also use when debugging it (every apply rolls back,
  Copy button instead of apply, block not showing). Requires Joy headless to be
  installed already (joy-headless-install).
when-to-use: add Joy redeem in cart to Hydrogen, Joy cart drawer block, JoyRedeemCart, redeem points in cart drawer headless, spend points in Hydrogen cart, Joy redeem cart not applying, /joy-headless-redeem-cart
argument-hint: "[path/to/cart-drawer component]"
compatibility: Requires a Shopify Hydrogen storefront (React Router 7, @shopify/hydrogen >= 2025.5) where Joy headless is already installed (root loader returns getJoyInitialData(), /api/joy-token route), and @joy-loyalty/hydrogen >= 0.3.0
metadata:
  author: tinhnv2001
  version: "1.0"
  short-description: "Add Joy Redeem in Cart to a Hydrogen cart drawer"
---

# Add Joy Redeem in Cart to a Hydrogen cart drawer

> **Goal:** mount `<JoyRedeemCart/>` in the storefront's cart drawer (or cart page) so signed-in
> customers see their points, redeem a reward and get the discount code applied to the cart in one
> click, and can remove an applied reward to get the points back. Design, texts and behavior come
> from Joy admin → **On-site content → Cart drawer → Redeem in cart**. Nothing is styled in code.

Merchant guide: https://help.joy.so/on-site-content/cart-drawer/redeem-in-cart-headless/ ·
Developer docs: https://devdocs.joy.so/joy-headless-api/redeem-in-cart/ ·
Reference storefront: https://github.com/tinhnv2001/weaverse-pilot-joy-loyalty
(`app/components/cart/joy-redeem-cart.tsx`, mounted in `cart-main.tsx`).

## 0. Prerequisites gate (check before touching code)

Stop and tell the user what is missing if any check fails.

- **Joy headless is installed.** The root loader returns `joy: await getJoyInitialData(...)` plus
  `joyApiUrl` and `joyShopDomain`, and `app/routes/api.joy-token.ts` (or `($locale).api.joy-token.ts`)
  exists. If not, run the `joy-headless-install` skill first. Do not half-install the auth here.
- **Package version.** `@joy-loyalty/hydrogen` must be **0.3.0 or later** (it pulls `@joy-loyalty/sdk`
  0.3.x). Older → upgrade: `npm install @joy-loyalty/hydrogen@^0.3.0` (pnpm/yarn equivalent), then
  `rm -rf node_modules/.vite` before restarting dev.
- **A cart drawer or cart page exists** and you can find the component that renders the cart lines
  and summary (§1).
- Never ask for, print or write `JOY_API_SECRET`. This skill does not need it.

## 1. Find where the cart renders

Look for the component that renders the line items and the summary of a non-empty cart:

- Hydrogen skeleton: `app/components/CartMain.tsx` (`CartMain` → `CartSummary`), drawer via `Aside type="cart"`.
- Weaverse Pilot: `app/components/cart/cart-main.tsx` (`layout === "drawer"` vs `"page"`).
- Otherwise: `grep -rn "CartForm\|CartSummary\|cart-drawer\|CartDrawer" app/`.

Mount the block **inside the cart, above the summary/checkout button, only when the cart has lines**.
The block already renders nothing on an empty cart, but wrapping it keeps layout stable. Ask the user
whether they want it in the drawer only, the `/cart` page only, or both. Default: drawer.

## 2. Create the component

Create `app/components/cart/joy-redeem-cart.tsx` (or match the project's naming):

```tsx
import {JoyRedeemCart} from '@joy-loyalty/hydrogen';
import {useFetcher, useRevalidator, useRouteLoaderData} from 'react-router';

export function JoyCartRedeem({cart}: {cart: any}) {
  const data = useRouteLoaderData('root') as any; // use the project's RootLoader type
  // Dedicated key: never share the fetcher the drawer's own CartForms use.
  const cartFetcher = useFetcher({key: 'joy-redeem-cart'});
  const {revalidate} = useRevalidator();

  if (!data?.joyApiUrl || !data.joyShopDomain) return null;

  return (
    <JoyRedeemCart
      apiUrl={data.joyApiUrl}
      shopDomain={data.joyShopDomain}
      initialData={data.joy ?? null}
      cart={cart}
      cartFetcher={cartFetcher}
      cartRevalidate={revalidate}
      cartRoute="/cart"
      {...(data.joyAssetUrl ? {assetUrl: data.joyAssetUrl} : {})}
    />
  );
}
```

Then render `<JoyCartRedeem cart={cart} />` at the spot chosen in §1.

- **`cartFetcher` + `cartRevalidate` together** turn on one-click apply. Missing either → copy-code
  mode (the code is shown with a **Copy** button). Only use that if the user asks for it.
- **Markets / locales:** `cartRoute` must be locale-prefixed (`/fr-ca/cart`) and `loginUrl` too
  (`/fr-ca/account/login`). Reuse the project's helper (Pilot: `usePrefixPathWithLocale('/cart')`).
- **Client-side navigation:** pass `onNavigate={useNavigate()}` if the project wants SPA links.
- Do not pass `assetUrl` in production unless the root loader already exposes an override.

Full prop list: `references/redeem-cart.md` §Props.

## 3. The cart action must return the cart

Open the cart route (`app/routes/($locale).cart.tsx` or `cart.tsx`). The block submits standard
`CartForm` actions to `cartRoute`: **`DiscountCodesUpdate`** (apply/remove a code) and **`LinesAdd`**
(free product rewards). The action must:

- handle both actions (the skeleton's `switch (action)` already does), and
- return the updated cart in the response — the skeleton's `data({cart: cartResult, errors, warnings}, {status, headers})`.

The block trusts only the cart returned by its own request. If the action returns a redirect, an
empty body, or a cart without the fragment fields (§5), the block cannot confirm the discount and
**rolls back every apply**. Fix the action; do not work around it in the component.

## 4. Client-side cart stores

If the drawer reads the cart from loader data, `revalidate()` is enough. If it reads from a client
store (zustand, context, `useOptimisticCart` over a stored cart), the store must pick up the cart the
block's fetcher returns, or the drawer shows the old total until the next navigation:

```tsx
<JoyRedeemCart /* … */ onCartUpdated={(updated) => setCartInStore(updated)} />
```

Pilot does the same with `useCartFetcherSync(cartFetcher)` in `joy-redeem-cart.tsx`.

## 5. Merge the cart fragment fields

1. Find the Storefront API version. Hydrogen's calendar version maps to it: `@shopify/hydrogen`
   `2025.10.x` → API `2025-10`, `2026.7.x` → `2026-07`. An explicit `storefrontApiVersion` passed to
   `createHydrogenContext`/`createStorefrontClient` wins over that.
2. Merge the matching variant from `references/redeem-cart.md` §Cart fragment into the project's
   `CART_QUERY_FRAGMENT` (skeleton: `app/lib/fragments.ts`). **Merge** fields — do not replace the
   fragment, and do not add a second `lines` selection with different arguments.
   - **Default** variant: Storefront API 2025-10 → 2026-10.
   - **Modern** variant: 2026-07 and later only. It fails on 2026-04 and earlier.
3. Validate the merged GraphQL against the project's API version (Shopify dev tools / `shopify-hydrogen`
   skill, or run the storefront and check the cart query has no errors), then run the project's
   `codegen` and `typecheck` scripts.

`collections(first: 20)` is optional: without it, rewards limited to collections are hidden. If the
discount allocation fields are missing while a code is applied, the block disables redemption.

## 6. CSP

Nothing new if the storefront already merges `joyCspDirectives()` (or `JOY_CSP_DIRECTIVES`) into
`createContentSecurityPolicy`: the block loads its code from the same Joy origin as `<JoyWidget/>`.
If the console shows a CSP error mentioning `joy.so`, merge every Joy directive (script, connect,
img, style, font), not only `connectSrc`.

## 7. Verify (dev server, then the Oxygen preview)

- [ ] Add a product, open the drawer → the block shows. Empty cart → no block.
- [ ] Guest (incognito) → earn-back line + **Sign in** link (or nothing if **Hide block for guest users** is on).
- [ ] Signed in with enough points → redeem a reward → the discount shows in the cart total, no rollback.
- [ ] Remove the applied reward (needs **Enable customer self-revoke coupon**) → the discount leaves the cart, points come back.
- [ ] Dynamic reward: slider (and the points input when **Show points input for dynamic rewards** is on).
- [ ] Locale-prefixed store: apply works on `/fr-ca/...` pages (cartRoute prefixed).
- [ ] Change a color in Joy admin Redeem in Cart settings → the drawer reflects it after reload.
- [ ] No `[joy] JoyRedeemCart: …` warning and no CSP error in the console.

Do not deploy. Hand the user the diff and the checklist; they deploy to an Oxygen preview first.

## 8. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Every apply rolls back right away | Cart action doesn't return `cart`, or the fragment lacks the allocation fields | §3 + §5 |
| Code shown with a **Copy** button instead of applied | `cartFetcher` or `cartRevalidate` not passed | Pass both (§2) |
| Apply works, but the drawer total updates only after navigating | Drawer reads a client cart store | `onCartUpdated` → store (§4) |
| Block never shows | Empty cart; cart currency ≠ shop base currency (v1 limit); hidden for guests; `joyApiUrl`/`joyShopDomain` missing from the root loader | Add a product, test in base currency, check settings and root loader |
| `[joy] JoyRedeemCart: …` warning | Widget core too old or blocked | Leave `JOY_WIDGET_ASSET_URL` unset in production; check CSP (§6) |
| Collection-limited rewards missing | No `collections` in the fragment, or product in > 20 collections | Add `collections(first: 20)` |
| Cart query errors after merging | Modern variant on API ≤ 2026-04, or duplicated `lines` selection | Use the Default variant; merge into the existing `lines` |
| Signed-in customer sees the guest state / `/api/joy-token` 401 | Headless auth problem, not the block | Use `joy-headless-install` §13 |
| Wrong cart after apply on a locale page | `cartRoute` not locale-prefixed | §2 |

## Known limits (tell the user, do not work around)

- **Base currency only:** when the cart currency differs from the shop's base currency the block renders nothing.
- **Earn-back is an estimate** on rule-based earning shops (collection/tag/vendor place-order rules aren't matched in the cart). Points are awarded correctly on the order.
- **Replace applied reward when a new one saves more** isn't supported yet: a non-combinable new reward is rejected.
