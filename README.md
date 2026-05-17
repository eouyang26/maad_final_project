# Forth Graph ∞

A generative-art breeding tool where short programs written in the Forth
language act as the "DNA" of visualizations. Breed programs together the same
way you would breed animals — combining genetic material from two parents to
produce offspring that visually blend, mutate, and surprise.

Open `forth_graph_ai.html` directly in a browser. No server, no dependencies.
An Anthropic API key is only needed for the AI injection features.

For hands-on instructions, see [PLAYTEST.md](PLAYTEST.md).

---

## The Big Idea

Each node in the graph holds a short Forth program. Running that program draws
something on a canvas. When you breed two nodes, their programs are spliced
together at a random cut point — the child starts like one parent and ends like
the other. A small numeric tweak is applied afterward. Over many generations
of breeding, the visuals evolve.

---

## What Is Forth?

Forth is a stack-based language. Instead of writing `sin(x)`, you push `x`
onto a stack and call `sin`, which pops `x` and pushes the result.

A short example:
```
0 0 0 0.1 bg-a
12 times {
  i twopi * 12 / frame 0.03 * +
  dup sin 100 * cx +
  swap cos 100 * cy +
  8 circle
}
```

- `0 0 0 0.1 bg-a` — draw a near-transparent black rectangle each frame
  (creates a trail/fade effect)
- `12 times { ... }` — run the block 12 times; `i` is the loop index (0–11)
- The math computes an angle from `i` and `frame`, converts to x/y with
  `sin`/`cos`, then draws a circle of radius 8

The canvas is never fully cleared between frames, so shapes leave trails.

---

## Token Types

`tokenize(src)` breaks the source string into three kinds of tokens:

| Token | Shape | Example source |
|---|---|---|
| Number | `{t:'n', v:42}` | `42`, `0.03`, `-1` |
| Word | `{t:'w', v:'circle'}` | `circle`, `hsl`, `times` |
| Block | `{t:'block', v:[...tokens]}` | `{ i sin 50 * circle }` |

---

## Drawing Words

| Word | Stack args | Effect |
|---|---|---|
| `circle` | `x y r` | Filled circle |
| `ocircle` | `x y r` | Stroked circle (outline) |
| `rect` | `x y w h` | Filled rectangle |
| `line` | `x1 y1 x2 y2` | Line segment |
| `hsl` | `h s l` | Set color (hue 0–360, sat/light 0–100) |
| `rgb` | `r g b` | Set color (0–255 each) |
| `bg-a` | `r g b a` | Fill canvas with semi-transparent color (trail fade) |
| `lw` | `n` | Set line width |
| `alpha` | `a` | Set global alpha |
| `rotate` | `angle` | Rotate canvas |
| `translate` | `x y` | Translate canvas |
| `push` / `pop` | — | Save / restore canvas transform state |

---

## Stack Words

| Word | Effect |
|---|---|
| `dup` | Duplicate top value |
| `drop` | Discard top value |
| `swap` | Swap top two values |
| `over` | Copy second value to top |
| `2dup` | Duplicate top two values |

---

## Math Words

`+`, `-`, `*`, `/`, `mod`, `abs`, `neg`, `floor`, `ceil`, `round`, `sqrt`,
`sin`, `cos`, `tan`, `pow`, `min`, `max`, `clamp`

---

## Special Values

| Word | Pushes |
|---|---|
| `frame` | Current frame number (increases each tick) |
| `i` | Current loop index (innermost `times` loop) |
| `j` | Loop index of the next outer loop |
| `cx` / `cy` | Canvas center x / y |
| `width` / `height` | Canvas dimensions (240) |
| `pi` / `twopi` | π / 2π |
| `rand` | Random float 0–1 |

---

## Loops

```
n times { ... body ... }
```

Runs the body `n` times. Inside the body, `i` is the current iteration
(0 to n−1). Loops can be nested; the inner loop's index is `i`, the outer
loop's index is `j`.

---

## How Breeding Works

Breeding combines two programs into one child, then applies a small mutation.
Two separate steps:

### Step 1 — Crossover (combination)

Splices program A and program B at random cut points:

```
Program A:  [bg-a] [50 times] [{...}] [hsl] [circle]
Program B:  [bg-a] [20 times] [{...}] [rect]

Cut A after index 2:  [bg-a] [50 times] [{...}]
Cut B after index 1:  [20 times] [{...}] [rect]

Child:  [bg-a] [50 times] [{...}]  +  [20 times] [{...}] [rect]
        ←—— from A ———————————————→   ←—— from B ————————————→
```

**Why compress/decompress?** A raw splice could cut between `50 times` and
its `{ block }`, producing a broken program. Before splicing, each
`n times { block }` triplet is folded into a single atom. The splice only
happens at atom boundaries. Afterward the atoms are unfolded back.

### Step 2 — Mutate (small twist)

Applies one random numeric change to the child program. Only number tokens
are ever touched — words like `circle`, `times`, `hsl` are never modified
or removed, so the program always stays executable.

Five possible mutations (one chosen at random):

| Mutation | Probability | What it does |
|---|---|---|
| Small tweak | 30% | Scale a number ±20% and add a small jitter |
| Negate | 20% | Flip the sign (mirrors, inverts) |
| Swap | 15% | Exchange two numbers (shifts proportions) |
| Wild replace | 15% | Replace a number with a random value ±360 |
| Large jitter | 20% | Add a big random offset ±60 |

---

## Seeds

Nine starting programs are built into the file, chosen for visual variety so
crossover has interesting raw material:

| # | Style | Color scheme | Trail |
|---|---|---|---|
| 0 | Orbiting dots | Warm orange (fixed hue) | Medium |
| 1 | Wave lines | Cyan (fixed hue) | Very long |
| 2 | Expanding rings | Grayscale | Fast fade |
| 3 | Scattered sparks | Two-color (orange / cyan alternating) | Very long |
| 4 | Spiral | Full rainbow (hue cycles) | Medium |
| 5 | Two-loop composition | Dim rings + blue dots | Medium |
| 6 | Rect equalizer bars | Two-tone (bouncing) | Fast |
| 7 | Dense dot grid | Hue ripple | Long |
| 8 | Sweeping lines | Monochrome white | Fast |

Six seeds are picked at random on each page load.

---

## Node Types

| Border color | Type | Description |
|---|---|---|
| Dim (default) | Seed | One of the 9 starting programs |
| Orange | P1 | Currently selected as Parent 1 |
| Blue | P2 | Currently selected as Parent 2 |
| Teal | Generated | Random node from GENERATE |
| Green pulse | Anim | Live animation via requestAnimationFrame |
| Purple | AI | Program written by Claude via AI INJECT |

Each node shows:
- **Header** — node ID and lineage (parent IDs, `seed`, or `✦` for generated)
- **Canvas** — 144×144 px display (rendered internally at 240×240)
- **Code panel** — the Forth source that produced this visual

---

## Static vs. Animated Rendering

**Static nodes** render once by running the program 100 consecutive frames
from a random `frameOffset`. Because programs use `bg-a` each frame, the
frames accumulate a trail effect — the result is a rich still image rather
than a single bare frame.

**Anim nodes** (created via ANIMATE) run one frame per `requestAnimationFrame`
tick, accumulating in real time. The canvas is pre-rolled 100 frames first so
the animation starts from the same visual state as the static snapshot —
it looks like the static image coming alive. If the program turns out to be
frame-independent, a gentle breathe-and-sway fallback keeps it feeling live.

---

## AI Features

**AI INJECT** — Type a theme or vibe into the text field (e.g. *ocean*,
*collapse*, *warm static*) and click AI INJECT. Claude writes a brand-new
Forth program evoking that theme and drops it as a purple node.

**AUTO** — Automatically injects a new AI node on a timer. Cycles through
30s / 60s / 120s / off. Countdown appears in the status bar.

**API key** — Required for AI INJECT and AUTO. Set via the KEY button or the
**?** help panel. Stored in browser `localStorage`. Get a key at
`console.anthropic.com`.

---

## Lineage Edges

Edges are SVG S-curve bezier paths connecting parent canvas centers to child
canvas centers. P1's edge is dim orange, P2's edge is dim blue. Anim node
edges are dashed. A small dot at the child end shows direction.

---

## Controls

| Action | How |
|---|---|
| Select P1 | Click any node |
| Select P2 | Click a second node (P1 stays) |
| Deselect | Click the selected node again |
| Move a node | Click and drag |
| Breed | BREED button (needs P1 + P2); child auto-becomes P1 |
| Animate | ANIMATE button (any node selected) |
| Export PNG / video | EXPORT button (any node selected) |
| Generate 1 random node | GENERATE button |
| AI-written node | Type theme → AI INJECT (needs API key) |
| Auto AI injection | AUTO button (cycles 30s / 60s / 120s / off) |
| Set API key | KEY button or ? panel |
| Delete selected node | Delete or Backspace key |
| Open instructions | ? button (also auto-shown on first visit) |
| Reset to 6 random seeds | RESET button |
| Scroll / pan workspace | Mouse wheel or drag background |

---

## Code Structure

Everything lives in one `<script>` block inside `forth_graph_ai.html`:

```
tokenize(src)               — string → token array
run(toks, ctx, W, H, fr)    — execute tokens on a canvas context

toksToStr(toks)             — token array → display string

compress(toks)              — fold loops into atoms (pre-crossover)
decompress(atoms)           — unfold atoms back to tokens (post-crossover)
crossover(a, b)             — splice two programs

mutate(toks)                — apply one random numeric change

SEEDS[]                     — 9 starting programs

addNode(...)                — create a static node (DOM + render)
createAnimNode(parentId)    — create a live animation node

selectNode(id)              — update P1/P2 selection
updateUI()                  — sync button states and status text

breed()                     — crossover + mutate → new child node
deleteNode(id)              — remove node, cancel RAF if anim
generate()                  — add 1 random node
aiInject()                  — call Claude API, add AI-written node
cycleAuto()                 — toggle auto-inject timer
exportNode(id)              — download node as PNG or WebM
reset()                     — clear everything, re-run init()
init()                      — place 6 random seed nodes in a grid
```
