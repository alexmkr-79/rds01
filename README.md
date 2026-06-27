# Rachnakar Design Studio — Standalone Production Build

Standalone Vite + React 19 site, extracted from the original Replit pnpm monorepo
(`Industrial-Precision`) and converted into a self-contained project for npm + Cloudflare Pages.

## What changed from the monorepo version

- Removed all `@replit/*` packages and plugins (cartographer, dev-banner, runtime-error-modal).
- Removed the required `PORT` / `BASE_PATH` / `REPL_ID` environment-variable checks in `vite.config.ts` — those only existed for Replit's multi-service runner.
- Removed the unused `@workspace/api-client-react` (`workspace:*`) dependency and the unused `zod` / `@hookform/resolvers` packages — none of them were actually imported anywhere in this frontend.
- Replaced every `catalog:` version with the exact resolved version pulled from the monorepo's `pnpm-lock.yaml`, so the dependency tree matches exactly what was already tested and working.
- The `api-server` artifact is not part of this project at all — this frontend never called it (all catalog data lives in `src/data/catalog.ts`, and the WhatsApp flow is a plain `wa.me` link), so there was nothing to rewire.
- Single self-contained `tsconfig.json` (no more `extends: "../../tsconfig.base.json"` or project references into sibling packages).
- `package.json` scripts are now plain `npm`/`vite` scripts: `dev`, `build`, `typecheck`, `preview`.

Everything else — every component, page, animation, and the visual design — is untouched.

## Before you deploy — replace these placeholders

1. **WhatsApp number** — currently a placeholder (`919876543210`) in:
   - `src/utils/whatsapp.ts`
   - `src/pages/contact.tsx`
2. **Domain name** — `https://www.rachnakardesign.com` is a placeholder used in:
   - `index.html` (canonical URL, Open Graph, JSON-LD)
   - `public/robots.txt` (sitemap URL)
   - `public/sitemap.xml`
3. **`public/admin/`** — a Decap/Netlify CMS admin panel using the `git-gateway` backend. That backend is Netlify-specific and **will not work on Cloudflare Pages as-is**. It isn't wired to the live site's content either way (the catalog is static data in `src/data/catalog.ts`), so it's safe to delete this folder if you don't plan to set up a CMS, or replace the backend config if you do.

## Local development

```bash
npm install
npm run dev
```

Opens at `http://localhost:5173`.

## Production build

```bash
npm run build
```

Outputs static files to `dist/`. Preview the production build locally with:

```bash
npm run preview
```

Optional type-check (not required for the build to succeed):

```bash
npm run typecheck
```

## Deploying to Cloudflare Pages

**Via Git integration (recommended):**
1. Push this repo to GitHub.
2. In the Cloudflare dashboard: Workers & Pages → Create → Pages → connect your repo.
3. Build settings:
   - **Build command:** `npm run build`
   - **Build output directory:** `dist`
   - **Node version:** 18 or later (set via an environment variable `NODE_VERSION=20` in the Pages project settings if needed)
4. Deploy.

`_headers` and `_redirects` in the project root are picked up automatically by Cloudflare Pages:
- `_redirects` sends all unmatched routes to `index.html` so client-side routing (wouter) works on hard refresh/direct links.
- `_headers` adds security headers (CSP, HSTS, X-Frame-Options, etc.) and long-cache rules for hashed build assets.

**Via Wrangler CLI (alternative):**
```bash
npm run build
npx wrangler pages deploy dist
```

## Notes on the security headers

The Content-Security-Policy in `_headers` allows:
- `style-src 'unsafe-inline'` — required because Framer Motion applies animation styles as inline `style` attributes.
- `fonts.googleapis.com` / `fonts.gstatic.com` — for the Google Fonts `<link>` tags in `index.html`.

Everything else defaults to `'self'`. If you add external APIs, images, or scripts later, you'll need to extend the relevant `*-src` directive in `_headers`.
