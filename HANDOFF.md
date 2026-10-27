# Cadence UI: "Koi Pond" design handoff

Paste or attach this file to a new Claude chat to continue the design or build it. It is self-contained. As of **2026-10-27**.

- **Live canvas:** https://claude.ai/artifact/6YfnW56BrZAwfcMyhB8yPh (shared "anyone with the link")
- **Source of every artboard:** `design/*.dc.html` in this repo (same files as the canvas)
- **Tokens:** `tokens.css` in this repo

---

## 1. What Cadence is (context)

Cadence is a private music ledger and fetch tool that replaces Lidarr + Aurral for one person (the owner, "Salt Father"). Tidal's catalogue is the source of truth; a SQLite ledger records what is owned; a fetch planner downloads what is missing. Backend is Python (CLI today); the planned web UI is FastAPI + Jinja + HTMX 2.x + a little Alpine, SSE for live parts, plain CSS with tokens, no build step, installable PWA served over Tailscale HTTPS. Phone first, desktop second.

Core concepts the UI shows:
- **Release states:** Owned · Partial n/m · Missing · Fetching · Lossy (AAC, held) · On card.
- **Sources:** Tidal (primary), OrpheusDL (Qobuz/Deezer module) and Soulseek (via slskd, only behind a Mullvad VPN through gluetun) as lossless fallbacks; YouTube Music as a lossy gap-finder.
- **Tidal gate:** one token bucket for all Tidal calls (1 req / 2 s, burst 3); it pauses on 429.
- **Destination per request:** Server · FiiO · Both. The FiiO Echo Snowsky Mini is a portable player whose SD card Cadence syncs.
- **Hard rules shown in copy:** nothing is ever deleted or overwritten without the owner confirming. Lossy results are *held*, not placed.

Real numbers used in mocks: library 5,210 files; last full fetch run 7 placed / 89 held as AAC / 1 failed; ~38 path conflicts to review; Will Paquin = 18 releases, 29 recordings; FiiO card 119 GiB exFAT, ~88 GiB used, budget = capacity − 5 GB. Anything unknown is a bracketed placeholder like `[Album title]` or `[N]`: never invent stats.

## 2. The look in one paragraph

A quiet koi pond at night. Deep blue-teal water with a faint 4px dither, lily-pad greens, heavy teal highlights, laid-back tones. 16-bit / SNES-era pixel art: hard pixel corners (2-step notch), inset bevels, sprites with selective outlines, 3-4 tone shading and dithering. Audiophile details are spliced in: a turntable, an analog VU meter with a warm cream backlight, a cassette, a DAP. One gold koi color (`#F4A640`) is reserved for "this needs you".

## 3. Tokens

| Token | Hex | Use |
|---|---|---|
| pond-deep | `#0B1F26` | page ground, nav bars, inset wells, dark text on light fills |
| pond-base | `#0F2B33` | app background (+ dither) |
| pond-shallow | `#163C46` | cards |
| pond-reed | `#1F5560` | borders, dividers, selected chip fill |
| teal | `#4FE0C1` | primary buttons, OWNED, active nav, progress |
| teal-shade | `#2E9C88` | pixel shadow of teal |
| teal-light | `#9FF2DF` | teal text on dark, ripples |
| lily | `#8AD47A` (shade `#5FAF5A`, dark `#3F8A43`) | PARTIAL, lossless counts |
| koi | `#F4A640` (shade `#C47A22`) | NEEDS YOU cards, LOSSY badge |
| alert red | `#E5603C` / `#D9452B` | VU red zone, peak LED only |
| foam | `#DDF3EC` | primary text, ON CARD badge |
| mist | `#B9D8D1` | secondary text |
| reed text | `#8FB8B0` | labels; ≥4.5:1 on pond-base |
| lotus | `#F7B8CF` / `#FFE6EF` / `#D9779A`, stamen `#F4D35E` | lotus flower only |
| VU face | `#F4DFA8` / glow `#FFF0C8` / shade `#E2C784` | VU meter only |

**Type**
- Display: **Pixelify Sans** 600/700, for names, headings, numbers and buttons. Sizes 72 (tile), 52 (desktop artist), 36 (desktop page title), 28-34 (phone title), 20-26 (card titles), 15-18 (buttons).
- Labels: **Silkscreen** 10-14px, upper case, letter-spacing 1-2px, for badges, chips, section labels and status.
- Body: **Atkinson Hyperlegible** 13-20px, for anything you read.

**Shapes and effects** (all in `tokens.css`)
- `.notch` is a 2-step pixel corner via clip-path. Every card, button, chip and badge has it. Never use border-radius.
- `.bevel` is an inset 2px light top-left plus 2px dark bottom-right.
- `.press:active` flips the bevel and moves the button 2px.
- `.dither` is the 4px checker water texture on the background.
- `.glint` is a stepped light sweep on FETCHING badges.
- Outlines use `outline: 2px solid|dashed` with `outline-offset: -2px`, since clip-path would clip a normal border.
- Text shadow on big titles: `4px 4px 0 #0B1F26`.

**Rules**
- Tap targets ≥44px, primary buttons 48px.
- Every state badge has a word, never colour alone.
- Gold appears only for things that need the owner.
- All animations step (`steps(n)`) for a sprite feel and are disabled under `prefers-reduced-motion`.
- Real `<button>`, `<a>`, `<input>` + `<label>`. Icon-only buttons get `aria-label`, decorative SVG gets `aria-hidden`, progress bars get `role="progressbar"`, the card meter gets `role="meter"`.

## 4. Sprites (each is its own component file in `design/`)

All sprites are inline SVG built from `<rect>`s on an integer grid with `shape-rendering: crispEdges`. Each takes a `scale` prop (an integer: pixel size = grid × scale) and renders at an exact multiple. Animations are CSS keyframes in the component's `<helmet><style>`, with stepped timing.

| File | Grid | Props | Animation |
|---|---|---|---|
| `Koi.dc.html` | 48×24 sprite in a 52×28 box (outline + drop shadow) | `scale` (1-10, default 4), `variant` kohaku/yamabuki/showa/kumonryu, `swim` bool, `wag` bool (default true) | Tail rotates ±10° around the peduncle, `steps(5)` 1.1s alternate. Pectoral fins flutter out of phase, `steps(3)` .9s. `swim` adds a 7s drift (translate + slight yaw). |
| `Vinyl.dc.html` | 40×34 | `scale`, `playing` bool, `labelColor` (teal/lily/koi) | The label (with a tiny koi emblem) spins `steps(8)` 1.6s while the groove sheen stays fixed, so it reads as a real light source. Tonearm on the record when playing, parked when not. |
| `VU.dc.html` | 40×28 | `scale`, `active` bool | The needle bounces through an irregular 2.4s keyframe "song", `steps(3)` per segment, and peaks into the red at 50%. The peak LED blinks at the same moment. At rest the needle sits at −50°. |
| `Tape.dc.html` | 40×26 | `scale`, `playing` bool | The two reel hubs spin `steps(6)` at different speeds (full vs nearly empty spool). The label is a mixtape with a koi doodle. |
| `Ripple.dc.html` | 24×12 | `scale`, `color`, `offset` (seconds, to desync several), `animate` | 3-frame sprite animation (small → medium → large ring with fading opacity), toggled by `step-end` opacity keyframes over 2.4s. |
| `Lily.dc.html` | 24×25 | `scale`, `bloom` bool, `animate` | The pad bobs 3px with ±2° rock (`steps(4)`). The lotus sways ±6° (`steps(3)`). |

**Koi palettes** (keys: w1 body, w2 body shadow, hl highlight, o1/o2/o3 pattern base/shadow/highlight, f1/f2 fin/fin ray, k eye, ol outline):
- kohaku (white + red-orange): `#FFF6E8 #D9CDBA #FFFFFF #F06A2A #B8401F #FF9A55 #F4D9BC #D9A57A #14232A #3A2018`
- yamabuki (gold): `#F7C948 #C9952A #FFF0A8 #E89A2A #B4701C #FFE08A #F7DFA0 #D9AE5A #14232A #4A3010`
- showa (white/red/black): `#FFF6E8 #CFC3B0 #FFFFFF #E0502A #1C2226 #FF8450 #E9DCCB #2A3338 #0B1418 #0B1418`
- kumonryu (black/white): `#E9EEF0 #B5C0C4 #FFFFFF #24323A #0E181D #4A5E68 #C8D2D6 #6F7F86 #0B1418 #121C21`

**Koi detail:**
- top-down body
- 4-direction 1px outline drawn as offset copies
- drop shadow on the water at (+2, +3), 40% opacity
- dithered belly via a 2×2 checker pattern
- scale texture via a 4×2 white-pixel lattice at 16% opacity, clipped to the body
- barbels and eye glints

**Variant meaning** (used in the UI):
- kohaku = Tidal / default
- yamabuki = OrpheusDL
- kumonryu = Soulseek
- showa = alert card

## 5. Screens (artboards)

**Style tile** `Main.dc.html` (1280×1040, static board):
- palette, type
- the four koi variants
- the hi-fi sprites, animated
- state badges, controls
- the "koi swims across" progress bar, with a ripple trailing behind
- a header pond with ripples, a bobbing lotus and two drifting koi

**Phone** (390×844):
- `Inbox.dc.html`:
  - status strip (Tidal OK · VPN UP · CARD AWAY)
  - gold "89 tracks held as AAC → Try fallbacks" card with a showa koi
  - new-release card with a stopped record, Grab plus destination chip
  - "38 path conflicts" review card
  - fetching-now bar with a swimming koi
  - bottom tab bar: Inbox · Library · Add · Activity · FiiO
- `Artist.dc.html`:
  - pond header with koi, ripple and lotus
  - Will Paquin, Following
  - owned / missing / lossy counts
  - filter chips (Albums, EPs, Singles, Variants)
  - 2-column release grid with state badges (pixel dither "covers" as placeholders)
  - sticky "Get all missing" bar
- `Activity.dc.html`:
  - Pause all
  - three source koi (Tidal gate OK, Orpheus · Qobuz, Soulseek · VPN Mullvad)
  - Tidal gate panel with the live VU meter
  - current job card showing its stage ("3 of 5 · verify"), progress koi and a plain-language why
  - queue
  - last-run tiles (7 / 89 / 1)
  - history
- `FiiO.dc.html`:
  - detailed DAP sprite with a koi on its screen
  - card capacity meter (used vs queued vs budget)
  - cassette sprite that spins while syncing
  - drag-to-order "set aside" list (On card / Queued / Won't fit)
  - gold "Proposed removals · 2, nothing leaves the card until you confirm"

**Desktop / web** (fluid pages, `expand: fill`, designed at 1440):
- Every page has a left rail (logo koi, nav, status block: Tidal gate, Mullvad VPN, FiiO card) that stacks above the content at phone width (flex-wrap; rail `flex: 1 1 220px; max-width: 260px`, content `flex: 999 1 560px`).
- `DeskHome.dc.html`:
  - pond banner with three koi, ripples and a lotus
  - search / paste-a-link + Add
  - Inbox card grid (`auto-fill minmax(260px,1fr)`) beside an Activity aside: fetching now, VU gate, last run
- `DeskArtist.dc.html`:
  - big pond hero
  - filter chips
  - release grid (`auto-fill minmax(170px,1fr)`)
  - release detail panel: tracklist with per-track quality/source (FLAC 24/48, FLAC · SOULSEEK, FLAC · ORPHEUS), "why this edition?", destination chips, history
- `DeskLibrary.dc.html`:
  - library health tiles (lossless, lossy/upgradable, unlinked legacy, 38 conflicts)
  - filter chips
  - sortable artist table inside an `overflow-x: auto` box

## 6. Component file format (`.dc.html`)

These files come from Claude's Design canvas, which is an HTML component format. Each file is one component: markup inside `<x-dc>`, page CSS in `<helmet><style>`, and a `<script type="text/x-dc" data-dc-script data-props='…'>` holding `class Component extends DCLogic { renderVals() {…} }`.
- `{{name}}` holes are plain lookups into `renderVals()`, never expressions.
- `<dc-import name="Koi" scale="{{k2}}" variant="kohaku" hint-size="104px,56px">` mounts a sibling component. Sizes are always `52×28 × scale` for the koi.
- `<sc-if value="{{cond}}">` is a conditional.

**Porting to the real app (Jinja + HTMX):**
- Each sprite becomes a Jinja macro `sprite_koi(scale=2, variant='kohaku', swim=False)` emitting the same SVG.
- The keyframes go once into the global stylesheet.
- `tokens.css` is the stylesheet base.
- Nothing needs JavaScript except the drag-to-order FiiO list (SortableJS island) and SSE.

## 7. Decisions made

- **Theme:** koi pond. This settles DESIGN.md §12.4 "visual direction".
- **No design system reused:** it is not Ender Tracker or Home Assistant. Cadence has its own look.
- **Pixel level:** 16-bit, 2px notch corners, bevels.
- **Koi variety per source:** described in §4.
- **Gold is reserved** for needs-you.
- **Placeholders** in brackets, never invented numbers.

## 8. Open / next ideas

- **Screens not drawn yet:**
  - Add/search with results and the request sheet
  - Settings: source health, Mullvad status, gate, quality policy
  - conflict review
  - empty and error states: no Tidal auth, gate cooling down, VPN down, card read-only, unknown card
- **Possible extra motion:**
  - a koi "turn-around" idle
  - koi following the cursor on desktop
  - tap-to-feed: tapping the water drops a pellet and a koi swims to it
  - a tonearm drop when a fetch starts
  - a VU meter driven by real gate load via SSE
  - a fireflies/lantern night mode
- **Light theme:** not designed. A "morning pond" with pale jade water would be the direction.
- **PWA icon:** the kohaku koi on a pond-deep square.

## 9. Repo layout

```
HANDOFF.md      this file
README.md       short pointer
tokens.css      colors, fonts, .notch/.bevel/.press/.dither/.glint
design/         canvas.json (artboard layout) + every .dc.html artboard and sprite
```
