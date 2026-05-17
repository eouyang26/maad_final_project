# Playtest Instructions — Forth Graph ∞

You're looking at a canvas of 6 static visualizations. Each one is a small
program that draws something. Your job is to breed them together, evolve new
visuals, and see how far the imagery drifts from where you started.

When you first open the file, a help popup will appear with a quick reference.
You can reopen it anytime with the **?** button in the top bar.

---

## The basics

**Click a node** to select it — it glows orange (Parent 1).
**Click a second node** — it glows blue (Parent 2).
**Click BREED.** A child node appears, mixing code from both parents.
The child automatically becomes your new selection. Keep going.

---

## Things to try

1. **Breed two visually opposite nodes** — sparse vs. dense, one color vs.
   rainbow. See what comes out in the middle.

2. **Breed a child back with one of its parents.** Results lean toward whichever
   side you keep selecting. After a few generations you can steer a direction.

3. **Click ANIMATE** (with any node selected) to bring it to life as a
   real-time animation. The canvas picks up exactly where the static image left
   off.

4. **Try the AI features** — type a word or phrase into the text field
   (e.g. *ocean*, *collapse*, *warm static*) and click **AI INJECT**.
   Claude will write a brand-new Forth program based on that vibe and drop it
   into the graph. You can then breed the AI node with existing ones.
   *(Requires an API key — see below.)*

5. **Click GENERATE** to add one fresh random node — good when you want new
   starting material without breeding.

6. **Export something you like** — select a node and click **EXPORT**.
   Static nodes download as PNG. Animated nodes record 3 seconds as a video.

7. **Delete nodes you don't like** — select one, press the Delete key.
   Clean up as you go so the graph stays readable.

---

## On breeding and randomness

Breeding is random by design. Sometimes the child looks like a direct blend of
its parents. Sometimes it looks almost identical to one parent. Sometimes it
comes out blank or strange.

**That's fine — treat it as a happy accident.** Don't delete it immediately.
Try breeding it with something completely different, or set it aside and work
from other nodes. Unexpected results are part of the process.

---

## API key (for AI features)

Click the **KEY** button in the top bar, or use the field in the **?** help
panel. Paste your Anthropic API key (`sk-ant-...`). It's saved in your browser
and only used to call Claude — nothing else.

Without a key, all the non-AI features (breed, animate, generate, export) work
fine.

---

## Quick reference

| Action | How |
|---|---|
| Select P1 | Click any node |
| Select P2 | Click a second node |
| Deselect | Click the selected node again |
| Move a node | Click and drag |
| Breed | BREED button (needs P1 + P2) |
| Animate | ANIMATE button (needs any node selected) |
| Export PNG / video | EXPORT button (needs any node selected) |
| Generate 1 random node | GENERATE button |
| AI-written node | Type theme → AI INJECT (needs API key) |
| Auto AI injection | AUTO button (cycles 30s / 60s / 120s / off) |
| Set API key | KEY button or ? panel |
| Delete selected node | Delete or Backspace key |
| Open instructions | ? button |
| Reset to 6 seeds | RESET button |

---

## Notes

- The small text under each canvas is the Forth code that produced it.
  You don't need to read it — use it as a fingerprint to tell nodes apart.
- Drag nodes freely to rearrange the graph.
- There's no score, no goal. Just explore.
