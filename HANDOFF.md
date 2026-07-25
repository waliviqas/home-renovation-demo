# Handoff — "Architect in Your Pocket" pitch deck

Context dump for continuing this work in a fresh chat. Written 2026-07-25.

---

## 1. What the product is

An AI-native home renovation product for **Bay Area homeowners** (B2C SaaS).

The homeowner:
1. Enters their **home address**
2. Uploads a **floor plan**
3. Describes the renovation in **natural language** — e.g. *"I'm looking to renovate
   my house, add two bathrooms, and break down the master suite"*
4. Optionally sets a **budget** (its own numeric field, separate from the prompt)

The backend then:
1. **Searches** — a web-search agent queries **Zillow and Redfin only**, given the address
2. **Profiles** the house — ranch vs. two-story, slab vs. raised foundation,
   load-bearing walls, square footage, year built
3. **Checks feasibility** — a rules engine crossing the prompt against the house
   profile, the budget, and **county permit rules**
4. **Returns up to 3 renovation routes** — each a self-contained HTML card with an
   isometric visualization, a cost range, and the **value uplift**
   (pre-renovation valuation vs. projected post-renovation valuation)

**Target user:** Bay Area homeowners looking to renovate. Scoped to **one county at
launch**, not the whole Bay Area.

---

## 2. Current status — read this first

**Only the pitch deck exists. The product itself has NOT been built.**

There is no backend, no floor-plan upload, no Zillow/Redfin agent, and no
feasibility engine. The UI shown on the deck's product slide is a **static mockup
picture**, not a working form.

The user explicitly said: *"no dont build the product. i am only doing the slide
deck."* Do not start building the product unless they ask.

---

## 3. The deck

**Repo:** `github.com/waliviqas/home-renovation-demo`
**Local:** `~/home-renovation-deck`

The repo was empty when cloned (one commit, just a `.gitignore`). Everything here
is new and **nothing has been committed yet.**

### Design brief

Match **jobs.netflix.com** — the user asked for this twice, and the second time
was explicit that they want the *flow*, not just the look: *"i want it exactly
same flow as netflix."*

Visual language:
- Black canvas (`#000`), Netflix red (`#E50914`)
- Giant stacked uppercase headlines in **Archivo Black** (stand-in for Netflix Sans)
- Middle line often **outlined** (`-webkit-text-stroke`) instead of filled
- Dark cards (`#141414`) with red hover borders

### How Netflix's flow actually works (researched from the live site)

This was the key finding, and it's **not** what a normal slide deck does:

- The page is **~22,000px tall at a 900px viewport — about 25 viewports.**
- It is **pinned-scroll**, not scroll-snap slides. Each act is a tall wrapper
  (3–15 viewports) containing a `position: sticky` inner that holds the viewport
  while your scroll position drives the animation inside it.
- **Hero "peekaboo" component:** headline words are split into `word` spans inside
  overflow-hidden wrappers (mask reveal, words slide up into view), with an
  **image sitting inline between the words** that parallaxes horizontally as you
  scroll. Netflix's own class name for this is `peekaboo`.
- An `sr-only` element with `clip-path: inset(50%)` carries the accessible full
  sentence, since the visible text is chopped into per-word spans.
- Five acts total: **Hero (3vh pin) → "We call it the Dream team" (5vh pin) →
  Gallery (15vh pin, slides advance under a fixed frame) → Mission statement
  (word-by-word reveal + CTA) → Closing (reuses the exact hero component as a
  bookend).**

### What was built to match

`index.html` — single self-contained file, no build step, no dependencies
(Archivo loads from Google Fonts). **~28 viewports tall**, five pinned acts:

| Act | Height | Content |
|---|---|---|
| 1 · Hero | 300vh | "An architect [house] in your pocket" — peekaboo image inline between words |
| 2 · Problem | 500vh | "Renovation is a black box" + 3 cards fading up |
| 3 · Gallery | 1200vh | 6 slides advancing under the pin: product mock, then pipeline steps 1–4, then the 3 routes |
| 4 · Mission | 500vh | Word-by-word lit reveal of the mission line + 3 stats |
| 5 · Closing | 300vh | "Built to demo end to end" — same peekaboo component as hero, + contact |

**Implementation notes:**
- `.act` is a tall wrapper; `.pin` inside is `position: sticky; top: 0; height: 100vh`
- One `requestAnimationFrame`-throttled scroll handler computes progress
  `p = (scrollY - act.offsetTop) / (act.offsetHeight - vh)` clamped 0–1, then drives
  word reveals, image parallax, fades, gallery slide index, and mission word lighting
- Word mask reveal: `.w { overflow:hidden }` wrapping `.w > i { transform: translateY(110%) }`
- The peekaboo images are **inline SVG isometric house drawings** (no external assets)
- `prefers-reduced-motion` fully supported — unpins everything and shows all content
- Keyboard nav jumps act-to-act; a red progress rail runs across the top

### Files

- `index.html` — the deck (pinned-scroll, current version)
- `slides-v1.html` — the earlier scroll-snap 7-slide version, kept as backup
- `README.md` — how to run and edit
- `.claude/launch.json` — dev server config for the preview tooling
- `HANDOFF.md` — this file

### Running it

```bash
cd ~/home-renovation-deck && python3 -m http.server 4173
```

Then open http://localhost:4173.

---

## 4. Decisions made (and why)

These came from the user's original voice brief plus discussion:

- **One county at launch, not the whole Bay Area.** Permit rules are the hard,
  defensible part and they're per-jurisdiction. Encoding one county deeply beats
  covering nine shallowly.
- **Zillow + Redfin only, no general web search.** Two structured, predictable
  sources keep the house profile consistent; every later stage depends on it.
- **Budget is optional and separate from the prompt.** Mixing money into the
  free-text box would force the model to parse it out of prose for no benefit.
- **Value uplift is part of the output, not just cost.** Showing only cost frames
  renovation as spending; showing pre- vs. post-renovation valuation frames it as
  the investment decision the homeowner is actually making.
- **Graceful no-API-key fallback is a definition-of-done item** — the MVP must
  always demo.
- **Deck flow matches Netflix's pinned-scroll**, not scroll-snap slides. The first
  version used scroll-snap and was rebuilt after researching the real mechanism.

---

## 5. Claude Design connection

The user asked how to connect Claude Design. Findings:

- **It's already connected.** A `DesignSync` tool is live in the session and
  `list_projects` succeeded through their claude.ai login with no auth prompt.
- **The account currently has zero design-system projects** — the list came back empty.
- A new one can be created with `DesignSync` `create_project`. Note the project
  **type is immutable at creation** — a regular claude.ai project can never become
  a design system, so it must be created as one.
- The `/design-sync` skill that normally pairs with the tool is **not installed
  locally**; only gstack's design skills are present (`design-consultation`,
  `design-html`, `design-review`, `design-shotgun`, `plan-design-review`,
  `ios-design-review`).
- **Caveat worth repeating to the user:** Claude Design is built for **design-system
  component libraries**, not single-page presentations. For a 7-slide pitch deck its
  value is marginal. It would make sense if they want to extract the deck's palette
  and type scale into a reusable system.

---

## 6. Tooling gotchas discovered (save yourself the time)

- **The in-app browser pane is unreliable for this.** Its viewport resets to ~527px
  after reloads (rendering the mobile layout), and its screenshots desynchronize
  from JS state — screenshots repeatedly showed a different scroll position than
  `window.pageYOffset` reported.
- **Headless Chrome `--screenshot` ignores JS-driven scrolling.** Scrolling via
  `window.scrollTo` before capture does not move what gets captured.
- **`will-change` breaks headless capture.** With `--virtual-time-budget`, elements
  carrying `will-change` render dimmed or blank because compositor layers aren't
  painted. Strip `will-change` in any capture copy.
- **Working approach for verification:** generate a throwaway `_cap.html` that takes
  `?act=N&p=0.0–1.0`, pins the target act with `position: fixed`, hides the others,
  overrides the scroll value the animation math reads, and calls the render function
  directly. Then screenshot with headless Chrome at `--window-size=1440,900`. This
  is deterministic and covers every animation state. Delete `_cap.html` afterward.

---

## 7. Wiki

Per the user's standing wiki rules, this work is captured at
`~/Documents/Wali's Neural Network/wiki/Home Renovation AI.md`, linked from
`wiki/index.md`, with a session entry appended to `log.md`.

**Note:** that wiki page was written for the *first* (scroll-snap) version of the
deck. If you continue this work, update its "Pitch deck" section to describe the
pinned-scroll rebuild.

---

## 8. Open items

- Nothing is committed to git yet — the deck, README, and this file are all untracked.
- The wiki page's deck section is stale (describes v1's scroll-snap flow).
- The gallery act's 6 slides at 1200vh means ~200vh of scrolling per slide; worth
  checking whether that feels too slow on a real trackpad.
- `slides-v1.html` can be deleted once the user confirms they prefer the new flow.
