# `<JoyRedeemCart/>` reference

For `@joy-loyalty/hydrogen` 0.3.x.

## Props

| Prop | Type | Required | Notes |
|---|---|---|---|
| `apiUrl` | `string` | yes | Headless API base (`JOY_API_URL`, incl. `/app/api/v1/headless`). |
| `shopDomain` | `string` | yes | The shop's `*.myshopify.com` domain. |
| `initialData` | `JoyInitialDataResult \| null` | yes | The root loader's `getJoyInitialData()` result — the same value `<JoyWidget/>` takes. |
| `cart` | cart \| `() => cart` | no | The storefront cart with the fragment fields below. Falls back to `<JoyProvider cart>`. No or empty cart → the block renders nothing. |
| `cartFetcher` | `useFetcher()` result | no | `useFetcher({key: 'joy-redeem-cart'})`. Required **with** `cartRevalidate` to apply codes. |
| `cartRevalidate` | `() => void` | no | `useRevalidator().revalidate`. Required **with** `cartFetcher`. |
| `cartRoute` | `string` | no | Cart action route. Default `/cart`. Locale-prefix it on multi-market stores. |
| `onCartUpdated` | `(cart) => void` | no | Called with the cart each of the block's cart mutations returns. Use it to sync a client cart store. |
| `tokenRoute` | `string` | no | Same-origin token route. Default `/api/joy-token`. |
| `loginUrl` | `string` | no | Overrides `initialData.login_url` for the guest **Sign in** link. |
| `onNavigate` | `(url) => void` | no | Client-side navigation (`useNavigate()`). Omitted → full page load. |
| `assetUrl` | `string` | no | Overrides the widget-core URL (`JOY_WIDGET_ASSET_URL`). Leave unset in production. |
| `className` | `string` | no | Class for the wrapper element. |

Without `cartFetcher` + `cartRevalidate` the block runs in **copy-code mode**: it still redeems, shows
the issued code with a **Copy** button and asks the customer to apply it at checkout. Issued codes are
remembered per customer for the browser session only.

## Cart actions the block submits to `cartRoute`

| `CartForm` action | When |
|---|---|
| `DiscountCodesUpdate` | Apply a redeemed code; remove a code on revoke or when it fails to apply. |

The block never adds lines: it lists only amount, percentage and free-shipping rewards.

The response must carry the updated `cart` (skeleton default `{cart, errors, warnings}`).

## Cart fragment

Merge into `CART_QUERY_FRAGMENT`. Pick one variant by Storefront API version.

**Default** — Storefront API 2025-10 to 2026-10:

```graphql
fragment JoyRedeemCartDefault on Cart {
  discountCodes { code applicable }
  cost { subtotalAmount { amount currencyCode } totalAmount { amount currencyCode } }
  discountAllocations { targetType discountedAmount { amount currencyCode } ... on CartCodeDiscountAllocation { code } }
  lines(first: $numCartLines) { nodes {
    id quantity
    cost { subtotalAmount { amount currencyCode } totalAmount { amount currencyCode } }
    discountAllocations { targetType discountedAmount { amount currencyCode } ... on CartCodeDiscountAllocation { code } }
    merchandise { ... on ProductVariant { id product { id collections(first: 20) { nodes { id } pageInfo { hasNextPage } } } } }
  } }
}
```

**Modern** — Storefront API 2026-07 and later only (fails on 2026-04 and earlier):

```graphql
fragment JoyRedeemCartModern on Cart {
  discountCodes { code applicable }
  cost { subtotalAmount { amount currencyCode } totalAmount { amount currencyCode } }
  lines(first: $numCartLines) { nodes {
    id quantity
    cost { subtotalAmount { amount currencyCode } totalAmount { amount currencyCode } }
    discountAllocations { targetType discountedAmount { amount currencyCode } ... on CartCodeDiscountAllocation { code } }
    joyAllDiscounts: discountAllocations(lineLevelOnly: false) { targetType discountedAmount { amount currencyCode } ... on CartCodeDiscountAllocation { code } }
    merchandise { ... on ProductVariant { id product { id collections(first: 20) { nodes { id } pageInfo { hasNextPage } } } } }
  } }
  deliveryGroups(first: 10) { nodes { discountAllocations { targetType discountedAmount { amount currencyCode } ... on CartCodeDiscountAllocation { code } } } }
}
```

- Merchandise totals are rebuilt from line costs and discount allocations (shipping excluded), never
  from `cost.totalAmount`. Missing allocation fields while a code is applied → redemption disabled.
- `collections(first: 20)` is optional. Absent or truncated (`hasNextPage: true`) → collection-scoped
  rewards are hidden.

## Joy admin settings the block honors

**On-site content → Cart drawer → Redeem in cart** (same settings as the Online Store theme block):

| Setting | Effect on Hydrogen |
|---|---|
| Design, texts | Applied as-is; no styling in code. |
| Hide block for guest users | Guests see nothing. |
| Enable customer self-revoke coupon | Shows the remove/revoke action on applied rewards; points are refunded. |
| Automatically remove failed discount codes | A code that fails to apply is removed and its points refunded. |
| Show points input for dynamic rewards | Typed amount in addition to the slider. |
| Replace applied reward when a new one saves more | Not supported yet — treated as off. |
