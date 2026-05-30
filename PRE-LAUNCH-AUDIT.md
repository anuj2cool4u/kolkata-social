# Kolkata Social — Pre-Launch Audit
*Reviewed: May 2026 · Scope: index.html + all assets*

---

## How to read this

Issues are grouped into three tiers. Fix everything in **🔴 P0** before the site goes live — these are either broken links, broken pages, or performance problems so severe they will drive people away. **🟡 P1** items will hurt conversions and professionalism if left. **🟢 P2** are polish and future-proofing.

---

## 🔴 P0 — Launch Blockers

### 1. Every single "Reserve a Table" link is broken

**This is the most critical issue on the site.**

There are **five** reservation CTAs across the page — nav bar, hamburger drawer, hero section, mobile sticky bar, and the contact column. Every one of them has `href="#"` with a code comment saying *"link coming soon."* Clicking any of them just snaps the user back to the top of the page.

For a restaurant, the reservation button IS the product. Nothing else matters if this doesn't work.

**What to do:** Get the OpenTable (or Dimmi/SevenRooms/direct booking) URL from the owner and replace every `href="#"` that has a reservation TODO comment. There are exactly five of them — search for `href="#"` in `index.html` to find them all.

Example fix:
```html
<!-- BEFORE -->
<a href="#" class="opening__reserve" title="Reservations via OpenTable — link coming soon">
  Reserve a Table →
</a>

<!-- AFTER -->
<a href="https://www.opentable.com.au/r/kolkata-social-newtown" class="opening__reserve"
   target="_blank" rel="noopener">
  Reserve a Table →
</a>
```

---

### 2. The "Our Story" page is a 404

The story section has `<a href="about.html" class="story__link">Our story →</a>`. The file `about.html` does not exist in the project. Any user who clicks it hits a dead end.

**What to do:** Either create `about.html` before launch, or temporarily change the link to `href="#story"` so it scrolls to the story section on the homepage.

---

### 3. Images will bring mobile users to a halt

Three venue images in the `/assets/images/venue/` folder are **raw camera exports** that have never been compressed:

| File | Size | Dimensions |
|---|---|---|
| `wall art.JPG` | **37 MB** | 5464 × 8192 px |
| `outside entry shot.JPG` | **29 MB** | 5464 × 8192 px |
| `plant wall.JPG` | **24 MB** | 4692 × 7034 px |

These three files alone are **90 MB**. The food rail images add another ~30 MB. On a typical Sydney mobile connection (25 Mbps), just the venue images would take **30+ seconds to load**. Most users will abandon the site inside 3 seconds.

**What to do:** Convert and resize everything before deploying. A practical target:

- Food rail images: resize to max 800px wide, export as WebP at quality 80 → target ~60–100 KB each
- Venue/dining room image (used on the site): resize to max 1400px wide, WebP at 80 → target ~150–200 KB
- Logos: leave as-is (PNG with transparency, already small)

Quick command to batch-process from Terminal (requires `cwebp`):
```bash
# Install: brew install webp
for f in assets/images/food/*.jpg assets/images/drinks/*.jpg; do
  cwebp -q 80 -resize 800 0 "$f" -o "${f%.jpg}.webp"
done
```

The three giant venue JPEGs aren't used on the homepage — confirm whether they're needed before launch or simply exclude them from the deployed build.

---

### 4. The popup drives users to the menu — not to book a table

The welcome popup fires 1.8 seconds after page load and its primary CTA is **"Explore the menu →"** which just scrolls the user to the food rail. This is a wasted conversion touchpoint.

The popup is the first interactive moment a new visitor has. It should drive the #1 action: **making a reservation**.

**What to do:** Change the popup CTA to link directly to the reservation URL:
```html
<!-- BEFORE -->
<button class="popup__cta" id="popup-cta">Explore the menu →</button>

<!-- AFTER -->
<a href="https://www.opentable.com.au/..." class="popup__cta"
   target="_blank" rel="noopener">Reserve a Table →</a>
```

Also change the popup heading from *"Bengali food shaped by memory"* to something that creates urgency around booking, e.g. *"Book your table"* or *"We're open Tue–Sat"*.

---

## 🟡 P1 — Significant Issues

### 5. Unresolved TODOs in the code — confirm before launch

The HTML has **9 TODO comments** flagging unconfirmed content:

- Opening hours (hero and contact section) — *confirm with owner*
- Phone number `0448 773 256` — *confirm with owner*
- Email `hello@kolkatasocial.com.au` — *confirm with owner*
- Chef name / prior kitchens (Firedoor, Raja) — *confirm with owner*
- Events enquiry email — *confirm with owner*
- Popup body text — *"drop PDF or email hello@..."*

None of these should be live with unverified content. The hours discrepancy is worth noting: the hero says **"Dinner · Tue – Sat"** but the schema markup says `closes: "22:30"` — that part is fine, but cross-check with the owner that Tue–Sat dinner is correct (the contact section correctly lists Tue–Fri + Saturday as separate rows).

---

### 6. Popup fires every session — it will annoy repeat visitors

The popup uses `sessionStorage` to remember dismissal. This means it fires again every time the user opens a new tab, restarts their browser, or visits from a different device. For a restaurant with regulars, this becomes very irritating quickly.

**What to do:** Switch from `sessionStorage` to `localStorage` so the popup is suppressed for at least 7 days after first dismissal:

```js
// BEFORE
if (!sessionStorage.getItem('ks-popup-dismissed')) {
  setTimeout(() => popupEl.classList.add('visible'), 1800);
}
function closePopup() {
  sessionStorage.setItem('ks-popup-dismissed', '1');
  ...
}

// AFTER
const dismissed = localStorage.getItem('ks-popup-dismissed');
const sevenDays = 7 * 24 * 60 * 60 * 1000;
if (!dismissed || Date.now() - parseInt(dismissed) > sevenDays) {
  setTimeout(() => popupEl.classList.add('visible'), 3500);
}
function closePopup() {
  localStorage.setItem('ks-popup-dismissed', Date.now().toString());
  ...
}
```

The delay has also been changed to **3.5 seconds** above — 1.8s is not enough time for a user to read the hero before being interrupted.

---

### 7. No favicon

The browser tab currently shows a blank page icon. On a phone's home screen, a saved shortcut would have no icon. This looks unfinished.

**What to do:** Export the fish logo at 32×32, 180×180 (Apple touch), and 192×192 (Android), then add to `<head>`:

```html
<link rel="icon" href="assets/images/logos/favicon.ico" sizes="any">
<link rel="icon" href="assets/images/logos/favicon.svg" type="image/svg+xml">
<link rel="apple-touch-icon" href="assets/images/logos/apple-touch-icon.png">
```

---

### 8. Missing Open Graph image — social shares will show nothing

The `<head>` has `og:title` and `og:description` but no `og:image`. When someone shares the site on WhatsApp, iMessage, Facebook, or LinkedIn, the preview will be a blank card with just the title.

**What to do:** Pick one strong food photo, resize it to 1200×630px, and add:

```html
<meta property="og:image" content="https://www.kolkatasocial.com.au/assets/images/og-image.jpg">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:url" content="https://www.kolkatasocial.com.au">
```

---

### 9. Muted text contrast may fail accessibility standards

The muted colour (`#7A746E`) on the cream background (`#F4F1EB`) produces a contrast ratio of approximately **3.5:1**. WCAG AA requires **4.5:1** for normal text and **3:1** for large text. Body copy like the story paragraphs (`font-size: 1.05rem, color: var(--muted)`) will fail AA for normal text.

This is used extensively — story body, events body, hours labels, opening details.

**What to do:** Darken the muted token slightly. `#5C5650` gives a ratio of ~5.8:1 on the cream background and preserves the visual softness:

```css
--muted: #5C5650;  /* was #7A746E */
```

---

### 10. Nav link font size is 11.2px — too small

`.nav__links a` is set to `font-size: .7rem` which is 11.2px at default browser settings. This is below the generally accepted minimum of 14px for navigation text and will be unreadable for users with any vision impairment, or anyone on a non-retina screen.

**What to do:** Increase to `.8rem` (12.8px) at minimum, ideally `.875rem` (14px):

```css
.nav__links a {
  font-size: .875rem;  /* was .7rem */
  letter-spacing: .08em;  /* reduce slightly at larger size */
}
```

---

### 11. Image filenames have spaces and special characters

Files like `"Plantain kofta .jpg"` (note trailing space), `"Logo both scripts zoom - Edited (1) (2).png"`, `"Kolkata_Social_Sep25_Credit_Jorge_Santos-92 (1).jpg"` will cause issues on Linux web servers (AWS, Vercel, Netlify) and can break when referenced in CSS `url()` values if not properly URL-encoded.

**What to do:** Rename all asset files to lowercase, hyphenated slugs before deploying:
- `Goat Kosha.jpg` → `goat-kosha.jpg`
- `Logo both scripts zoom - Edited (1) (2).png` → `logo-main.png`
- `Plantain kofta .jpg` → `plantain-kofta.jpg`

Then update the references in `index.html`.

---

## 🟢 P2 — Polish & Future-Proofing

### 12. Add a canonical URL tag

```html
<link rel="canonical" href="https://www.kolkatasocial.com.au/">
```

Without it, if the site is ever accessible via both `http://`, `https://`, with and without `www.`, search engines may index multiple versions and split ranking authority.

---

### 13. Preload the hero image for faster LCP

The hero section uses a background watermark image. While it's decorative, the visible above-the-fold content should load as fast as possible. Add a preload hint for the most important visual image (e.g. the first food rail photo or the dining room image if used above fold):

```html
<link rel="preload" as="image" href="assets/images/food/goat-kosha.webp">
```

---

### 14. Add `loading="eager"` and `fetchpriority="high"` to the first rail slide image

All rail images currently have `loading="lazy"`, including the first one which is visible on page load. The first image should be eager:

```html
<!-- First slide only -->
<img src="assets/images/food/goat-kosha.webp" alt="Goat Kosha"
     loading="eager" fetchpriority="high">
```

---

### 15. The food rail hint text isn't visible enough on mobile

`← drag to explore` is `font-size: .68rem, color: var(--muted)`. On mobile this is barely visible. A subtle animated swipe indicator (a small arrow that bounces once) communicates the interaction far more effectively than text.

---

### 16. No "Reservations" column CTA visible without scrolling on desktop

On a 1280px screen, the contact section's Reservations column (with the booking CTA) sits below the fold. The user has to scroll to find it. The hero CTA and nav CTA cover this well — but only once the reservation links are fixed (see P0 #1).

---

### 17. Consider adding a `robots.txt` and `sitemap.xml`

Basic SEO hygiene. The `sitemap.xml` should list `index.html` and `about.html` (once created). The `robots.txt` should at minimum have:
```
User-agent: *
Allow: /
Sitemap: https://www.kolkatasocial.com.au/sitemap.xml
```

---

## Summary Checklist

| # | Issue | Priority | Effort |
|---|---|---|---|
| 1 | All 5 reservation links broken | 🔴 P0 | 15 min |
| 2 | `about.html` missing (404) | 🔴 P0 | 30 min |
| 3 | 90MB of uncompressed images | 🔴 P0 | 1–2 hrs |
| 4 | Popup CTA drives to menu, not booking | 🔴 P0 | 10 min |
| 5 | Unverified TODO content (hours, phone, email) | 🟡 P1 | Owner sign-off |
| 6 | Popup re-fires every session | 🟡 P1 | 15 min |
| 7 | No favicon | 🟡 P1 | 30 min |
| 8 | Missing og:image | 🟡 P1 | 15 min |
| 9 | Muted text fails WCAG AA contrast | 🟡 P1 | 5 min |
| 10 | Nav links too small (11.2px) | 🟡 P1 | 5 min |
| 11 | Image filenames with spaces | 🟡 P1 | 30 min |
| 12 | Missing canonical URL | 🟢 P2 | 5 min |
| 13 | No preload for hero/LCP image | 🟢 P2 | 10 min |
| 14 | First rail image shouldn't be lazy | 🟢 P2 | 5 min |
| 15 | Rail drag hint invisible on mobile | 🟢 P2 | 30 min |
| 16 | Reservations col below fold | 🟢 P2 | noted |
| 17 | No robots.txt / sitemap | 🟢 P2 | 20 min |

**The single most important thing:** fix the reservation links. The site looks beautiful and the brand voice is strong — but right now a customer who wants to book a table has no way to do so.
