# Viola — fashion landing page

React 19 + TypeScript + Vite. Plain CSS (one stylesheet per component), design tokens in
`src/styles/tokens.css`. No animation library is needed: every motion is CSS transitions/keyframes
plus small hooks for IntersectionObserver, parallax and swipe.

```bash
npm install
npm run dev      # http://localhost:5173  (dev-only swap test bench: /#__swap-test)
npm run build    # static output in dist/ (relative paths, deployable to any sub-path)
```

## Structure

```
src/
  data/assets.ts      Central asset registry (size, provenance, Figma node) + hero media modes
  data/products.ts    Products (modelImage / productOnlyImage / alt / pairVerified), section content
  hooks/motion.ts     Reduced motion, page visibility, in-view, one-shot section reveal, preload
  components/         ProductCard (+ swap), ProductPreview dialog, shared UI, icons, mask shapes
  sections/           Hero, SignatureStyle, Editorial, ShopTheLook, StyleAccordion,
                      FeatureCards, CircularGallery, Accessories, Testimonials, Footer
  dev/SwapTestBench   Dev-only harness for the hover swap (never in production builds)
```

## Motion system

| Use | Duration | Token |
| --- | --- | --- |
| Micro-interactions | 220ms | `--t-micro` |
| Image crossfade | 400ms | `--t-image` |
| Accordion | 360ms | `--t-accordion` |
| Gallery / carousel | 580ms | `--t-gallery` |
| Section entrance | 720ms | `--t-enter` |
| Stagger | 80ms | `stagger()` |
| Entrance easing | `cubic-bezier(0.22, 1, 0.36, 1)` | `--ease-out` |

Sections reveal once at ~18% visibility. Hidden states only apply after JS opts in
(`html.js-motion`), with a 2.5s fail-safe, so content is readable if scripts or animation fail.
`prefers-reduced-motion` disables reveals, parallax, hotspot pulse and the hero loop (poster shown).
Hero motion pauses when offscreen or when the tab is hidden.

## Hero breeze — how to switch it on

`heroMedia` in `src/data/assets.ts` picks the richest source available:

1. **Cinemagraph** — set `video: { webm, mp4 }`. Rendered muted, looping, `playsInline`, with the poster.
2. **Layers** — set `layers: [...]` with `role` (`background`, `flowers-back`, `model`, `flowers-front`),
   `anchor` (stem base, e.g. `"40% 100%"`), `sway` 1–3°, `period` 5–9s, `delay`, `depth` (parallax 0–1).
   Flower layers sway around their anchors; the model layer only gets depth parallax.
3. **Poster** (current) — still image with ≤10px pointer parallax on desktop.

## Smooth scrolling

Lenis (`hooks/smoothScroll.ts`) gives the whole page inertial scrolling; it is skipped for reduced motion
and paused while the product preview dialog is open.

## "Your Style, Naturally" slider

Scroll-driven: the section is a tall runway with a sticky stage; scroll progress picks the slide
(1.6s crossfade, slow zoom settle, small drift inside the mask). Arrows, dots and ←/→ scroll the page
to the matching slide. Slides live in `editorialSlides` (`products.ts`). The mask is the Figma shape with ~30px rounded corners (`components/shapes.ts`).

## Product image swap — how to switch it on

For each product set both `modelImage` and `productOnlyImage` (same SKU, colour, material, details),
write both alt texts, then set `pairVerified: true`. Until then the card shows one image and no swap.

## Asset replacement checklist

| # | Needed | Where | Status |
| - | --- | --- | --- |
| 1 | Hero cinemagraph `public/video/hero-loop.mp4`: 1920×1080, 24fps, 8s loop (Kling 4.0, H.264 CRF 21) | `heroMedia.video` | Generated |
| 2 | Product-only photos for Olive Linen Blazer, Cream Tailored Suit, Classic Blazer were AI-generated (ghost mannequin) from the model photos; replace with studio photography of the real SKUs before launch | `assets.ts` (status `generated`) | Generated — temporary |
| 3 | On-model photos for Ivory Blazer and Ivory Wide-Leg Trousers (cards currently show product-only shots) | `modelImage` | Missing |
| 4 | Confirm the Shop-the-Look tiles (blazer, trousers, slingbacks) are the exact SKUs worn in the look photo; re-check hotspot positions | `lookHotspots` | Needs confirmation |
| 5 | Separate floral layers for the footer if it should echo the breeze | `Footer` | Missing — static |
| 6 | Real, consented testimonials and portraits (the concept's quote, names and the "Famous Celebrities" claim are placeholders; the off-canvas celebrity photo was intentionally not used) | `testimonials` | Placeholder |
| 7 | Real prices, SKUs, product URLs, nav/footer links | `products.ts`, `navLinks` | Sample data |
| 8 | Higher-resolution masters (Figma sources max out at 1254–1672px wide) | `public/img` | Usable, could be sharper |

## The Finishing Touch

Every accessory starts as a white packshot; hover/focus crossfades to the same product in the garden
scene (on touch, the card nearest the centre of the track shows it). Scene images and the white hat
packshot were generated from the Figma product images (status `generated` in `assets.ts`).

## Single-file build

`npx vite build -c vite.single.config.ts` produces `dist-single/index.html` with JS/CSS/fonts inlined;
the distributed `Viola-landing.html` additionally inlines images and the hero video (window.__VIOLA_ASSETS),
so it opens by double-click with no server.
