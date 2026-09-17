# NBQ Customer Form — Prototype

Bilingual (English / Arabic) **suggestion & complaint form** prototypes for
National Bank of Umm Al Qaiwain, built to run full-screen on a branch kiosk, tablet or
desktop browser tab **with no page scrollbar** — the whole form fits the viewport.
Phones are the deliberate exception and scroll normally.

> [!IMPORTANT]
> **This is an unofficial prototype for design review.** It is not operated by NBQ and
> is not a live banking service. Nothing you type is stored or transmitted anywhere —
> submissions are simulated in the browser and logged to the developer console. Do not
> enter real personal or account information.

**Live demo:** https://YOUR-USERNAME.github.io/nbq-customer-form-prototype/

---

## The two demos

Open [`index.html`](index.html) and pick one.

| | Demo 1 | Demo 2 |
|---|---|---|
| File | [`nbq_customer_form_prototype.html`](nbq_customer_form_prototype.html) | [`nbq_contact_form_variation.html`](nbq_contact_form_variation.html) |
| Purpose | The focused form, built to the approved requirements | Mirrors the live `nbq.ae/general/contact` layout, for consistency with the website |
| Required | Message | Message |
| Optional | Email, Mobile, plus a Suggestion / Complaint selector | Account, Title, Full Name, E-mail, Mobile, plus Subject / New Customer / Need Callback selectors |
| Layout | Single centred card | Info rail beside a two-column form |

Both are single self-contained HTML files — no build step, no dependencies, no backend.
Fonts load from Google Fonts with a system fallback, so they degrade gracefully offline.

---

## Running it

Any static host, or straight off disk:

```bash
# from the repo root
python -m http.server 8000
# then open http://localhost:8000
```

All files must stay in the same folder — `index.html` links to its siblings by
relative path.

---

## How it is built

Three conventions hold across all three pages, so the set behaves as one system.

### 1. Fixed viewport on kiosk, tablet and desktop

On a kiosk there is often no mouse wheel and no visible scrollbar, so anything below
the fold is unreachable. The pages are therefore built to *fit*, not to scroll:

| Mechanism | Detail |
|---|---|
| Viewport shell | `body` is `height: 100dvh; overflow: hidden`, a flex column of header / stage / footer |
| Fluid scale | One variable, `--fs: clamp(…, vmin, …)`, drives every font size, padding, radius and control height through `calc()` |
| Elastic field | The message `textarea` is `flex: 1` with `resize: none`, absorbing all leftover height |
| Reserved error lines | Each field reserves a fixed-height line for its error text (toggling `visibility`, not `display`), so validation never reflows the layout |
| Progressive trimming | `max-height` media queries drop non-essential chrome (footer, subtitles, helper copy) before fields are allowed to shrink |
| Scroll fallback | If a viewport genuinely cannot fit the content (tiny phone, or an on-screen keyboard halving the screen), a small JS guard restores a thin scrollbar **inside the form column only** — better a visible scrollbar than content stranded behind an invisible one |

#### Phones scroll instead

Forcing nine fields into a 375 px-wide viewport means type too small to read, and phone
users expect to scroll anyway. Below the breakpoint the fixed shell switches off
entirely: natural page scroll, fixed 16 px type instead of viewport-scaled, a sticky
header, a resizable message box, and no reserved error lines (they exist only to stop a
desktop error reflowing a fixed layout).

```css
@media (max-width: 640px), (max-height: 480px) and (pointer: coarse) { … }
```

The second clause catches phone landscape. Tablets and desktop windows match neither, so
they keep the fixed design untouched. Inputs are exactly 16 px on mobile — below that,
iOS Safari zooms the page when a field takes focus and leaves the layout shifted
sideways. The full-screen button is hidden on phones: mobile browsers manage their own
chrome, and iOS Safari will not fullscreen a non-video element.

#### Verified

| | Result |
|---|---|
| 320×568, 360×740, 375×844, 414×896, 430×932 | scrolls vertically, no horizontal overflow, 16 px inputs |
| 768×1024, 834×1112, 1024×768 (tablet) | fits, no scroll |
| 1280×800, 1366×768, 1600×900, 1920×1080 (desktop) | fits, no scroll |

In both languages, including with every validation error showing.

### 2. Brand palette derived from the logo

The logo is the real asset, extracted from the bank's own icon sprite
(`icons.svg#icon-logo`) and saved as [`nbq-logo.svg`](nbq-logo.svg). It is inlined in
each page so every file stays self-contained — relevant for a kiosk behind an internal
root CA, where an extra asset request is one more thing to go wrong.

The palette comes from that logo. Only two source colours; the darker stops are the
**same hues scaled down**, so gradients and hovers cannot drift off-brand.

| Token | Hex | Derivation |
|---|---|---|
| `--nbq-blue` | `#0072BC` | **logo blue** |
| `--nbq-blue-mid` | `#005B96` | blue × 0.80 |
| `--nbq-blue-dark` | `#004A7A` | blue × 0.65 |
| `--nbq-blue-deep` | `#003B62` | blue × 0.52 |
| `--nbq-red` | `#ED1C29` | **logo red** |
| `--nbq-red-deep` | `#C71520` | red, darkened |

All four blues sit on hue 203.6°, both reds on 356.3°.

**Why there are two reds.** The logo red scores 4.38:1 on white — fine for borders and
rules, which need 3:1, but under the 4.5:1 that WCAG AA requires for small text. Error
messages and required asterisks use `#C71520` instead: same hue, 5.90:1. Large fills
and rules keep the true brand red. Every text pair in both forms was measured and
passes AA.

### 3. Bilingual, RTL-aware

Every string comes from an `i18n` dictionary keyed by `data-i18n`. Switching language
flips `dir` on `<html>`, swaps the font stack, and mirrors the layout via logical CSS
properties (`margin-inline-start`, `inset-inline-start`). The logo is *not* mirrored —
correct for a wordmark — only its position in the header flips.

Phone numbers carry `dir="ltr"` so the bidi algorithm does not reorder the digit
groups in Arabic.

---

## Field rules

**Message is the only required field. Both forms accept anonymous submissions.**

Everything else is validated *only if the customer filled it in*, which is what makes
anonymous submission work:

- **Message** — required. Non-empty after trimming, so whitespace alone is rejected.
  Max 2000 characters, with a live counter.
- **Email** — optional. If present, must match `name@domain.tld`.
- **Mobile** — optional. If present, prefers UAE format (`+9715XXXXXXXX` / `05XXXXXXXX`);
  general international numbers also accepted.
- **Everything else** — optional, no format rule.

The first field that fails gets focus. The submit button is disabled while the request
is in flight.

Covered by 7 scripted cases per form (message-only submits; empty and whitespace-only
messages blocked; valid email/mobile submits; malformed email/mobile rejected).

---

## Payload

Submission is simulated. The payload that *would* be posted is logged to the console —
open DevTools to inspect it. Every blank optional field is sent as `null`, and
`anonymous` is `true` when nothing identifying was supplied.

```json
{
  "type": "Complaint",
  "email": null,
  "mobile": null,
  "message": "The queue at the branch was very slow.",
  "anonymous": true,
  "language": "en",
  "submittedAt": "2026-09-17T15:30:00.000Z",
  "source": "Customer Form Prototype (kiosk)"
}
```

To wire up a real endpoint, replace the simulated delay in the `submit` handler:

```js
// currently:
console.log('Payload to be sent to CRM/ESB API:', payload);
await new Promise(r => setTimeout(r, 900));

// production:
const res = await fetch(ENDPOINT, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' /* + auth */ },
  body: JSON.stringify(payload)
});
if (!res.ok) { /* show a retry-able error */ }
```

Two things to confirm with the middleware team first: whether their schema accepts
`null` for email and mobile, and whether the `anonymous` flag can be added. If those
fields are mandatory server-side, anonymous submissions will be rejected even though
the form permits them.

---

## Kiosk behaviour

- **Full-screen toggle** (⛶) in the header, via the Fullscreen API.
- **Auto-reset** — the confirmation screen returns to a blank form after 15 s.
- **Idle wipe** — a part-filled form is cleared after 120 s of inactivity, so one
  customer's details are not left on screen for the next person.
- Pinch-zoom is disabled; touch targets are at least `--fs × 2.4` tall.

Both timings are constants at the top of the kiosk-hygiene block in each file:

```js
const AUTO_RESET_SECONDS = 15;
const IDLE_RESET_MS = 120000;
```

Nothing is persisted — no cookies, no `localStorage`, no backend.

---

## Files

```
index.html                        Demo launcher
nbq_customer_form_prototype.html  Demo 1 — Suggestions & Complaints
nbq_contact_form_variation.html   Demo 2 — Contact Us
nbq-logo.svg                      Official logo, extracted from the bank's icon sprite
.nojekyll                         Serve files as-is on GitHub Pages
```

If you add pages, copy the logo `<path>` data from `nbq-logo.svg` rather than retyping
it — it is ~8.5 KB of path data and a truncated copy renders a mangled wordmark.

---

## Browser support

Chrome, Edge, Safari and Firefox, current versions, desktop and mobile. Uses `100dvh`,
`clamp()`, CSS custom properties, logical properties, `matchMedia` and `ResizeObserver` —
all baseline in evergreen browsers. No transpilation, no polyfills.

Pinch-zoom is allowed (WCAG 1.4.4). Lock it at the browser or OS level if a kiosk needs
it disabled — a `user-scalable=no` meta tag is the wrong tool, since iOS has ignored it
since iOS 10.

---

## Deploying to GitHub Pages

1. Push to GitHub.
2. **Settings → Pages → Source:** *Deploy from a branch*, branch `main`, folder `/ (root)`.
3. Wait for the build, then open `https://YOUR-USERNAME.github.io/REPO-NAME/`.

`index.html` at the repo root is served as the landing page. `.nojekyll` tells Pages to
skip Jekyll processing and serve the files exactly as committed.

---

## Notes on ownership

The NBQ name, logo and brand colours are the property of National Bank of Umm Al
Qaiwain and are used here only to prototype work commissioned by the bank. This
repository is a design prototype and carries no affiliation with, or endorsement by,
NBQ. The pages are marked `noindex` so they do not appear in search results alongside
the bank's real website.
