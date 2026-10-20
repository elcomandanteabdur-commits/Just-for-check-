# The Chocolate Story – Admin Panel (standalone)

Static HTML/CSS/vanilla-JS (ES modules) admin panel for the existing Firebase project `the-chocolate-story`. It does **not** need the customer storefront ZIP or the old ORÈVA admin ZIP. No build step or npm.

**Status: Stage 2 of 5.** Foundation (Stage 1) plus products, categories, stock fields and Cloudinary image upload are written. Orders, shipments, payments, customers, content, reviews, requests, settings and the live dashboard are still placeholders (Stages 3–4). Final ZIP comes at Stage 5.

## Files
- `index.html` routes to login or dashboard; `login.html`; one HTML file per section (`dashboard`, `products`, `product-edit`, `categories`, `orders`, `order-detail`, `shipments`, `payments`, `customers`, `custom-orders`, `reviews`, `content`, `activity-logs`, `settings`); `offline.html`.
- `js/firebase.js` is the only place with the Firebase config and SDK version. `js/auth.js` guard, `js/common.js` shell and helpers, `js/pages.js` page list, `js/collections.js` collection names and order transitions, `js/config.js` business constants.
- `manifest.webmanifest`, `sw.js`, `assets/`, `css/`, `docs/`.

## Run / host
Serve the folder over **HTTPS** from any static host (GitHub Pages project URLs work; paths are relative). For local testing use `localhost`, e.g. `python3 -m http.server` in this folder. Opening files with `file://` will not work (ES modules, service worker).

## Google sign-in setup
1. Firebase Console → Authentication → Sign-in method: enable **Google**.
2. Authentication → Settings → Authorized domains: add your hosting domain (and `localhost` if testing).
3. Sign in with `thechocolatestory.ind@gmail.com`. Any other account is shown "Access denied".

## Security: UI gate vs real security
The email check only hides the interface. Firestore rules are what actually protect data; do not weaken them. Current rule limits this app respects: orders cannot be created/deleted and admin can change only `status`, `updatedAt`, `adminNote` along valid transitions; payment transactions and activity logs are read-only; Razorpay verification, audit logging and safe stock handling need a trusted backend that is not configured. No email service, courier API or Razorpay verification exists. Never put secret keys in this project.

## Cloudinary (from Stage 2)
Upload endpoint `https://api.cloudinary.com/v1_1/dyt6fwvw0/image/upload`, unsigned preset `Wb_mobile_products`. Restrict that preset in Cloudinary settings before production. No API secret is used.

## Install on Android
Open the hosted HTTPS URL in Chrome, then menu → Install app / Add to Home screen (wording varies by browser). Not yet verified on a real phone.

## Testing checklist (Stage 1)
- Signed out: open `dashboard.html` directly → redirected to `login.html?next=dashboard.html`, nothing private shown.
- Sign in with a non-owner Google account → Access denied, "Sign out and try another account" works.
- Sign in as owner → dashboard; every sidebar link opens; sign out works; mobile menu opens and closes (Escape, scrim).
- Turn off network → offline banner appears; reload a visited page → offline page or cached shell.
- DevTools → Application: manifest valid, service worker active, cache holds only static files.

## Later integration
If the storefront ZIP is supplied later, compare its collection paths, fields and image consumers with `docs/FIREBASE_DATA_CONTRACT.md`. Compatibility with an unseen storefront is not claimed.

## Stage 2: products, categories, images
- **Products** (`products.html`, `product-edit.html`): search by name/slug/keyword/tag; filters for category, type, visibility, stock and featured; activate/deactivate, mark available/unavailable, delete (with a usage warning). The editor covers every `products` field, including per-product delivery charge, options, images, stock and SEO.
- **Categories** (`categories.html`): add/edit, image, order (Up/Down), activate/deactivate. A category that products still use cannot be deleted; the app lists the products and suggests moving them or deactivating instead.
- **Options (new records):** `choice` options (list of choices with optional extra price) and `text` options (customer types text, max length, optional extra price). Existing options in another shape are kept untouched. See `docs/FIREBASE_DATA_CONTRACT.md`.
- **Images:** JPG/PNG/WebP up to 8 MB and 6000 px, uploaded to Cloudinary with preview, progress, error and Retry. Only the real `https` URL Cloudinary returns is saved, and only when you press Save. Removing an image here does not delete it from Cloudinary (that needs the API secret, which this app never holds).
- **Saving:** updates change only the fields in the form, so other fields on a product are left alone. If saving fails after an upload, the page says the image is on Cloudinary but the product was not saved, and keeps the images in the form for a retry.
- **Stock caution:** stock numbers are set by hand in the browser and are not safe against simultaneous checkouts. Safe stock holds need a trusted backend.

## Automated checks (no browser needed)
`node --import ./tests/register.mjs tests/run.mjs` runs 19 checks of validation, option handling, product/category services (against an in-memory fake Firestore) and upload-file rules. These are code-path tests, not tests against your live Firebase.
