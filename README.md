# Wave Scene — SVG Parallax Scroll

A scrolling ocean scene built with SVG, GSAP and ScrollTrigger. Fish jump, a boat bobs, a whale drifts below. Each layer moves at a different speed as you scroll, creating the illusion of depth.

This is a **learning project**. The architecture is borrowed from [isladjan's SVG Parallax](https://codepen.io/isladjan/pen/abdyPBw) — not copied, but studied and rebuilt from scratch with original art and a simpler scene. The goal is to understand how SVG parallax works by building one.

---

## What it will look like

A continuous vertical scroll through an ocean environment:

- **Sky and clouds** at the top (move slowest — far away)
- **Waves** layered front to back in different blues (each moves at a different speed)
- **A boat** sitting on the surface waves
- **Jumping fish** breaking the surface
- **Underwater section** — a whale, smaller fish, sea plants
- **Ocean floor** at the bottom (moves fastest — closest to you)

Later: a **beach scene** added below the ocean, as a second scroll section.

---

## How the parallax works (the concept)

The whole scene is one tall SVG file. Inside it, every element that needs to move independently is in its own **named group** (`<g id="far-clouds">`, `<g id="mid-waves">`, `<g id="fish-1">`, etc.).

When you scroll, JavaScript (GSAP + ScrollTrigger) moves each group at a different speed:

- Far-away things (sky, distant clouds) move **slowly** — barely shifting
- Mid-depth things (waves, boat) move at **medium** speed
- Near things (foreground wave, underwater plants) move **fast**

Your brain reads the different speeds as depth. That's the entire trick.

ScrollTrigger ties the animation to scroll position — scroll down and the layers shift; scroll back up and they reverse. The `scrub` setting makes this smooth rather than snappy.

---

## Tech stack

| Tool | Job |
|------|-----|
| **Figma** | Drawing the SVG — waves, arranging imported elements, naming groups, exporting |
| **SVGRepo / OpenClipart** | Free vector elements — fish, whale, boat, sea creatures |
| **GSAP 3** | Animation library — moves SVG elements smoothly |
| **ScrollTrigger** | GSAP plugin — ties animations to scroll position |
| **HTML/CSS** | Page structure and base styling |
| **JavaScript** | Wiring up ScrollTrigger to each SVG group |

No build tools, no frameworks, no npm. One HTML file, one CSS file, one JS file, one SVG file.

---

## Project structure

```
wave-scene/
├── index.html          # page structure — loads the SVG inline + scripts
├── css/
│   └── style.css       # layout, background, responsive rules
├── js/
│   └── main.js         # GSAP + ScrollTrigger setup for each layer
├── assets/
│   └── scene.svg       # the full scene — one file, named groups inside
├── reference/
│   └── (isladjan code notes, screenshots, your own sketches)
└── README.md           # this file
```

---

## Build order

### Phase 1 — Study (no code yet)

**Step 1.1: Read isladjan's code on CodePen**
Open [the pen](https://codepen.io/isladjan/pen/abdyPBw) and look at three things:

1. **The HTML** — find where the SVG sits in the page. Notice it's inline (pasted directly into the HTML, not loaded as an `<img>`). Find the named groups inside it (`<g id="...">`)
2. **The CSS** — see how the SVG is sized to fill the viewport and how the page height is set (the page needs to be taller than the screen to create scroll distance)
3. **The JS** — find the `gsap.to()` calls. Each one targets an SVG group by its ID and moves it a certain distance as you scroll. Notice the `scrollTrigger` object inside each one, especially `scrub: true`

Don't try to understand everything. Just map the shape: HTML loads the SVG → CSS sizes it → JS moves the named groups on scroll.

**Step 1.2: Sketch the scene on paper**
Draw a rough vertical strip. Mark the layers from back to front:
- What's furthest away? (sky, clouds)
- What's in the middle? (waves, boat)
- What's nearest? (foreground wave, underwater close-ups)

Write a name next to each layer. These names become the group IDs in your SVG.

---

### Phase 2 — Build the SVG (Figma session)

**Step 2.1: Set up the canvas**
Create a new Figma file. Set the frame to **1440px wide × 4000px tall** (you can adjust the height later — this gives roughly 3 screens of scroll to start with).

**Step 2.2: Draw the wave layers**
Start at the top of the canvas. Draw a wavy line across the full width using the pen tool. Close the shape at the bottom to make a filled region. Fill with a light blue. Duplicate, shift down, adjust the curve slightly, fill with a slightly darker blue. Repeat 4–6 times, getting darker toward the bottom. Each wave is its own layer.

**Step 2.3: Add imported elements**
Download SVG elements from OpenClipart or SVGRepo (fish, whale, boat, sea plants). Import them into Figma. Place them between your wave layers at the depth where they belong. Recolour them to match your palette.

**Step 2.4: Name every group**
This is critical. Select each element or wave layer in the Figma layers panel and give it a clear name: `sky`, `far-wave-1`, `mid-wave-2`, `near-wave-3`, `boat`, `fish-jump`, `whale`, `seabed`. These names become the IDs that JavaScript will target.

**Step 2.5: Export as SVG**
Select the whole frame → Export → SVG. Open the exported file in a text editor and check that your group names survived as `<g id="...">` tags. Figma usually preserves them.

---

### Phase 3 — Page structure (HTML/CSS session)

**Step 3.1: Create the HTML file**
Build the page skeleton: an HTML file that loads your CSS, pastes the SVG inline (the full SVG code goes directly into the HTML body, not as an image), and loads the GSAP + ScrollTrigger scripts from CDN at the bottom.

**Step 3.2: CSS — make the SVG fill the screen**
The SVG needs to be the full width of the viewport. The page height is determined by the SVG height — the taller the SVG, the more you scroll. No fixed `height` on the body — let the SVG's natural height create the scroll distance.

**Step 3.3: Test — does it scroll?**
Open in a browser. You should see your scene and be able to scroll through it top to bottom. Nothing moves in parallax yet — that's next. If you can see the scene and scroll through it, this phase is done.

---

### Phase 4 — Parallax animation (JavaScript session)

**Step 4.1: Load GSAP and ScrollTrigger**
Add these two CDN scripts to your HTML (before your own JS file):

```html
<script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/gsap.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/ScrollTrigger.min.js"></script>
```

Register the plugin at the top of your JS file:

```javascript
gsap.registerPlugin(ScrollTrigger);
```

**Step 4.2: Animate one layer first**
Start with a single layer to prove the concept. Pick your furthest-back element (e.g. `#sky`):

```javascript
gsap.to("#sky", {
  y: -100,              // moves up 100px over the scroll distance
  ease: "none",         // linear — no speeding up or slowing down
  scrollTrigger: {
    scrub: true          // ties the animation to scroll position
  }
});
```

Scroll and watch. The sky should drift upward slowly as you scroll down. If it moves, the concept is working.

**Step 4.3: Add more layers, each with a different speed**
Add `gsap.to()` for each named group. The key: far layers get small `y` values (move a little), near layers get large `y` values (move a lot). The difference in speed is what creates the depth.

```javascript
// Far — moves least
gsap.to("#far-wave-1", { y: -50,  ease: "none", scrollTrigger: { scrub: true } });

// Mid — moves more
gsap.to("#mid-wave-2", { y: -150, ease: "none", scrollTrigger: { scrub: true } });

// Near — moves most
gsap.to("#near-wave-3", { y: -300, ease: "none", scrollTrigger: { scrub: true } });
```

**Step 4.4: Tune by eye**
Adjust the `y` values until the depth feels right. This is taste, not science. Sit with it. Change the numbers. Scroll again. The tuning is where the feeling lives.

**Step 4.5: Add character animations**
Make the fish jump, the whale drift, the boat bob. These use the same `gsap.to()` pattern but may also move on the `x` axis or use `rotation`:

```javascript
gsap.to("#fish-jump", {
  y: -200,
  x: 100,
  rotation: 15,
  ease: "none",
  scrollTrigger: { scrub: true }
});
```

---

### Phase 5 — Polish (HTML/CSS + JS session)

**Step 5.1: Background colour**
Set a gradient background behind the SVG — light blue at the top (sky), darker blue at the bottom (deep ocean). This fills any gaps between your wave layers.

**Step 5.2: Responsive check**
Open on a smaller screen. Does the SVG scale? The `viewBox` attribute on the SVG handles this — it should scale proportionally. Test and adjust if needed.

**Step 5.3: Smooth scroll feel**
Adjust `scrub` values — `scrub: true` is instant tracking; `scrub: 0.5` adds a slight lag that feels smoother. Try both.

---

### Phase 6 (future) — Add the beach scene

This is a separate section below the ocean, added when the ocean feels complete. Same pattern: new SVG groups (sand dunes, palm trees, crabs), new ScrollTrigger rules. The ocean section doesn't change.

---

## Key concepts to understand

### What is a viewBox?
The `viewBox` attribute on an SVG defines its internal coordinate system. `viewBox="0 0 1440 4000"` means the SVG's internal world is 1440 units wide and 4000 units tall. The browser then scales that world to fit whatever space you give the SVG on screen. This is why SVGs scale cleanly — the shapes are described in their own coordinate system, and the browser maps it to pixels.

### Why inline SVG (not `<img>`)?
When an SVG is loaded as an `<img>`, the browser treats it as a flat picture — you can't reach inside it with JavaScript. When the SVG code is pasted directly into the HTML (inline), every group and shape becomes part of the DOM, and JavaScript can target them by ID. Parallax needs JavaScript to move the groups, so the SVG must be inline.

### What does `scrub: true` do?
Without `scrub`, a ScrollTrigger animation plays once when triggered (like pressing play). With `scrub: true`, the animation's progress is tied directly to your scroll position — scroll 50% of the way and the animation is 50% done. Scroll back up and it reverses. The scroll bar becomes the playhead.

### What does `ease: "none"` do?
Easing controls acceleration. `ease: "none"` means constant speed — no slow-start or slow-finish. For parallax this is usually what you want, because the movement should feel tied directly to your scrolling, not bouncing or easing in.

### Why separate files (HTML, CSS, JS)?
At this stage of learning, keeping each language in its own file makes it easier to find things and reduces the chance of breaking one thing while editing another. One concern per file. Later, when you're more comfortable, you can make informed choices about combining them.

---

## Session plan

| Session | Discipline | What happens |
|---------|-----------|-------------|
| **Documentation** | This README | Project plan, architecture notes, decisions log. Update as the build progresses. |
| **Figma** | SVG design | Draw waves, import elements, name groups, export SVG. No code. |
| **HTML/CSS** | Page structure | Build index.html, style.css. Get the SVG visible and scrollable. No animation. |
| **JavaScript** | Animation | Write main.js. Wire up GSAP + ScrollTrigger. Tune the parallax feel. |
| **Integration/QA** | Testing | Check everything works together. Responsive, smooth, no visual glitches. |

Each session stays in its lane. If a Figma session discovers the SVG export has wrong IDs, that's a note for the next Figma session — not a reason to start editing code. If a JS session reveals a layer needs splitting, that's a note back to the Figma session.

**You will likely loop between Figma and JavaScript** as you discover things: "this fish needs to be in its own group" or "these two waves should be one layer." That's normal. The loop is: design → export → test in browser → note what needs changing → go back to design.

---

## What you already know that applies here

From your freeCodeCamp progress (step ~56) and Novo work, you already understand:

- **Variables** — the `y: -100` values are just numbers you'll adjust
- **Functions** — `gsap.to()` is a function call with an object as its argument
- **Template literals / strings** — the `"#sky"` selectors are strings targeting element IDs
- **Objects** — the `{ y: -100, scrollTrigger: { scrub: true } }` syntax is an object with a nested object inside it

What's new:
- **The DOM** — JavaScript reaching into the HTML to find elements by ID (you'll meet this in freeCodeCamp soon)
- **Libraries** — using someone else's code (GSAP) by loading it from a CDN
- **CSS layout for full-viewport elements** — making the SVG fill the screen
- **SVG structure** — how groups and IDs work inside an SVG file

---

## Reference

- [isladjan's SVG Parallax (CodePen)](https://codepen.io/isladjan/pen/abdyPBw) — the project this learns from
- [isladjan's project page](https://isladjan.com/work/2/) — his own description
- [GSAP ScrollTrigger docs](https://gsap.com/docs/v3/Plugins/ScrollTrigger/) — the official reference
- [SVGRepo](https://svgrepo.com/) — free SVG elements
- [OpenClipart](https://openclipart.org/) — public domain vectors
- [Figma](https://www.figma.com/) — SVG design tool (free tier)

---

## Licence

Learning project. Original art and code by Jeanette. Architecture inspired by isladjan's work (credited above). Free vector elements sourced from OpenClipart and SVGRepo under their respective licences.
