# Architect in Your Pocket — pitch deck

A pinned-scroll deck for the AI-native home renovation MVP. Six acts across
~27 viewports, in a white-and-orange liquid-glass treatment.

Single self-contained `index.html`. No build step and no dependencies — Outfit
and Inter load from Google Fonts.

## Viewing

Open `index.html` directly, or serve it:

```bash
python3 -m http.server 4173
```

Then visit http://localhost:4173. Scroll, or use arrow keys / space to jump
act to act. Best at 1280px wide or more.

## The flow

Structure follows [jobs.netflix.com](https://jobs.netflix.com), which is *not*
a slide deck — each act is a tall wrapper holding a `position: sticky` inner
that pins the viewport while scroll progress drives the animation inside it.

| Act | Height | What happens |
|---|---|---|
| 0 · Title | 250vh | RENOVATE assembles letter by letter, a scan crosses it, rings expand, it dissolves into the hero |
| 1 · Hero | 300vh | "An architect in your pocket" — the house image opens from zero width and pushes the words apart |
| 2 · Problem | 500vh | Three glass cards on feasibility, quotes, permits |
| 3 · Gallery | 900vh | Six cards that scale toward you and dissolve into each other |
| 4 · Mission | 500vh | The mission line lights word by word, plus three stats |
| 5 · Closing | 300vh | Bookend that reuses the hero component, plus contact |

Everything is scroll-driven. Nothing is on a timer, so the deck plays forward
and backward exactly as you scroll.

## The light

Two **curtains** of vertical striations sit against the left and right edges,
centre kept clear so type stays readable. Each is three
`repeating-linear-gradient` layers scrolling sideways at different speeds — the
interference between layers is what makes it read as liquid rather than as
moving stripes. A **glint** sweeps each curtain every 6.5s, offset so the two
sides never fire together.

Behind that, four blurred orange blobs drift on a fixed layer. That layer is
what the glass panels refract; without it `backdrop-filter` has nothing to show
and every panel renders white-on-white.

## Editing

Everything is in `index.html`:

- `:root` — the orange spectrum, ink colours, and the glass tokens
- `.curtain` / `.c1`–`.c3` — light curtain speed (`--sp`) and density (`--tile`)
- `.glass` — the shared panel treatment
- One `render()` function drives every act from its scroll progress `p` (0→1)

Act length is set inline as `style="height:Nvh"` on each `<section class="act">`.
Making an act longer slows everything inside it down; the animation math is
normalised to the act, so nothing else needs changing.

### A note on verifying changes

Scroll-driven state can be screenshotted headlessly by faking `pageYOffset` and
pinning one act with `position: fixed`. Anything time-based cannot — headless
Chrome with `--virtual-time-budget` does not advance `requestAnimationFrame`,
so timed animation has to be checked in a real browser.
