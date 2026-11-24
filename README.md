# WA Express COD

Zero-WhatsApp-API Cash-on-Delivery checkout SaaS for Pakistani & GCC Shopify sellers, built on **100% free-tier Cloudflare Workers + KV + Static Pages**.

## 1. KV Namespace & Wrangler Setup
1. Install Wrangler: `npm i -g wrangler` and run `wrangler login`.
2. Create the single KV namespace:
   ```bash
   wrangler kv namespace create SAAS_KV
   wrangler kv namespace create SAAS_KV --preview
   ```
3. Paste the returned `id` and `preview_id` into `wrangler.toml` under `[[kv_namespaces]]`.
4. Generate and set the Worker secrets:
   ```bash
   openssl rand -base64 32
   wrangler secret put ENCRYPTION_KEY
   wrangler secret put SESSION_SECRET
   wrangler secret put PIN_PEPPER
   wrangler secret put SUPER_ADMIN_KEY
   wrangler secret put TURNSTILE_SECRET
   ```
5. Update `[vars]` in `wrangler.toml`:
   - `ALLOWED_ORIGINS`: comma-separated exact frontend origins
   - `SUPPORT_WA`: your support WhatsApp digits with country code (no `+`)
   - `APP_VERSION`: `1.0.0`

## 2. Deploy Worker & Static Frontend
1. Deploy the Worker: `wrangler deploy`.
2. Create a Cloudflare Turnstile widget (optional) and copy its Site Key.
3. Update `CONFIG` in `index.html`, `owner.html`, and `super-admin.html`.
4. Deploy the 3 HTML files to Cloudflare Pages or GitHub Pages.
5. Put **Cloudflare Access** in front of `super-admin.html`.
6. Cross-check `PK_CITIES` in `worker.js` and `index.html` periodically against your courier's (Leopards, PostEx, TCS) current serviceable city list.

## 3. Free-Tier KV Write Budget (1,000 Writes / Day)
Daily limits reset at **00:00 UTC (5:00 AM PKT)**.

| Event | KV Writes | Notes |
|---|---|---|
| Public catalog load (`GET /api/catalog`) | **0** | 90 s isolate memory cache + `Cache-Control: max-age=60` |
| Successful Shopify order (`POST /api/order`) | **1** | Only `usage:{billingSlug}:{YYYY-MM}` via `ctx.waitUntil` |
| WhatsApp-only / fallback / duplicate / bot reject | **0** | No KV write |
| `devdash` token refresh | **~1 / store / 23h** | Cached in `tok:{slug}` + isolate memory |
| Owner login failure | **0** (until 10 fails) | In-memory up to 8; 1 write to `lock:{slug}` after 10 |
| Store register / settings / admin approve | **1–2** | Infrequent control-plane actions |

**Upgrade trigger**: Upgrade to **Workers Paid ($5/month)** once you exceed ~**500–800 successful orders/day** or observe KV write-limit errors.

## 4. 10-Step QA Checklist (Using `?store=demo`)
1. `GET <API_BASE>/api/health` -> `{ "ok": true, "shopifyApi": "2026-07" }`.
2. Open `index.html?store=demo` on 360 px mobile -> verify skeleton and 3 demo products.
3. Toggle **اردو / EN** -> verify RTL flip and translated labels.
4. Check sold-out variant (`XL / Charcoal`) is disabled.
5. Toggle upsell -> verify live discount and free delivery hint.
6. Test `03000000000` (rejected) and `923001234567` (normalized).
7. Search `"Lahore"` -> verify `Rs 180` delivery fee.
8. Submit valid order after >3.5 s -> verify `#DEMO-1042` + `Open WhatsApp`.
9. Open `super-admin.html`, log in, edit `cfg:payment`, export CSV.
10. Verify `owner.html` loads cleanly and `ALLOWED_ORIGINS` matches.

## 5. Onboard Your First Real Store
1. Register in `owner.html` (slug, brand, `*.myshopify.com`, WhatsApp, PIN).
2. **Mode A (Dev Dashboard)**: Create app at `dev.shopify.com/dashboard`, App URL `https://shopify.dev/apps/default-app-home`, uncheck embedded & legacy install flow, add `read_products` & `write_orders`, Release, install on store, copy Client ID & Secret.
3. **Mode B (Legacy)**: Paste `shpat_...` Admin API token.
4. Click **Test Connection** in `owner.html` -> all 4 checks pass.
5. Copy WhatsApp Quick Reply from **Links** tab.
