# Wave Scene — SVG Parallax Scroll

A scrolling ocean scene built with SVG, GSAP and ScrollTrigger. Fish jump, a boat bobs, a whale drifts below. The sea darkens as you scroll deeper — bright turquoise at the surface, deep navy at the ocean floor. Each layer moves at a different speed, creating the illusion of depth.

This is a **learning project**. The architecture is studied from [isladjan's SVG Parallax](https://codepen.io/isladjan/pen/abdyPBw) and rebuilt from scratch with original art and a simpler scene. One scene instead of his three, waves instead of mountains, but the same core mechanics.

---

## What it will look like

A continuous vertical scroll through an ocean environment:

- **Sky** at the top — simple, mostly background colour
- **Sea surface** — the waterline, with a boat sitting on it
- **Wave layers** — stacked front to back in progressively darker blues
- **Jumping fish** breaking the surface
- **Underwater section** — a whale, smaller fish, sea plants
- **Ocean floor** at the bottom — rocks, sand, seaweed

As you scroll, the water colour shifts from bright surface blue to deep dark navy — the same technique he uses for his sunset-to-night sky, but applied to ocean depth.

---

## How it actually works (from studying his code)

### The architecture — fixed SVG + invisible scroll div

This is the key insight, and it's different from what you might expect:

1. **The SVG is small and fixed.** His viewBox is `750 × 500` — roughly a widescreen rectangle. It's pinned to the viewport with `position: fixed` and `height: 100vh`. It never scrolls. It just sits there filling the screen.

2. **An invisible div creates the scroll distance.** A separate `<div class="scrollElement">` is set to `6000px` tall. This is what your scrollbar actually scrolls through. The div is transparent — you never see it. But it gives ScrollTrigger something to measure against.

3. **JavaScript moves groups inside the fixed SVG** based on how far you've scrolled through the invisible div. Scroll 50% of the way through the div → the animations are 50% done. The SVG doesn't move; the pieces inside it do.

This means: your SVG canvas is viewport-sized (e.g. `viewBox="0 0 1440 900"`), not thousands of pixels tall. The scroll distance is controlled by the invisible div's height — make it taller for a slower, longer scroll; shorter for a quicker one.

### The speed multiplier pattern

He defines one variable: `let speed = 100`. Then each layer gets a multiplier:

```javascript
// Far away — moves a little
gsap.to("#far-wave", { y: 1.5 * speed }, 0);

// Mid depth — moves more
gsap.to("#mid-wave", { y: 3 * speed }, 0);

// Near — moves a lot
gsap.to("#close-wave", { y: 5 * speed }, 0);
```

The `speed` variable lets you tune the whole scene's feel by changing one number. The multipliers set the depth ratios between layers. Bigger multiplier = faster movement = feels closer to you.

### The scrub setting

`scrub: 3` means the animation takes 3 seconds to "catch up" to your scroll position. This creates a smooth, slightly trailing feel — things drift into place rather than snapping. Lower numbers (1) feel more responsive. Higher numbers (4) feel more floaty. He uses different scrub values for different elements: hills get 3 (smooth), clouds get 1 (responsive), the bird gets 4 (very floaty).

### The gradient colour shift — and how you'll use it for depth

This is how his sky goes from sunset to night, and how your sea will go from bright to dark.

Inside the SVG, the background is filled with a gradient — a set of colour stops that blend into each other. Each stop has an `offset` (where it sits in the gradient, 0 to 1) and a `stop-color`. GSAP can animate both:

```javascript
// Shift where a colour band sits
gsap.to("#ocean-grad stop:nth-child(2)", { attr: { offset: "0.7" } });

// Change what colour a stop is
gsap.to("#ocean-grad stop:nth-child(3)", { attr: { "stop-color": "#0a1628" } });
```

For your ocean scene, the SVG background would be a vertical gradient — light blue at the top, medium blue at the bottom. As you scroll deeper, GSAP shifts the stops and changes the colours, making the water progressively darker. Same mechanic as his sunset, different palette.

Your gradient might start as:
```xml
<linearGradient id="ocean-grad" x1="0" y1="0" x2="0" y2="1">
  <stop offset="0" stop-color="#87CEEB" />    <!-- sky / surface -->
  <stop offset="0.3" stop-color="#4FA4C7" />  <!-- shallow water -->
  <stop offset="0.6" stop-color="#2B6E99" />  <!-- mid water -->
  <stop offset="1" stop-color="#0D2B45" />    <!-- deep water -->
</linearGradient>
```

And GSAP animates it toward darker values as you scroll:
```javascript
depth.to("#ocean-grad stop:nth-child(1)", { attr: { "stop-color": "#2B6E99" } }, 0);
depth.to("#ocean-grad stop:nth-child(2)", { attr: { "stop-color": "#1A4D6E" } }, 0);
depth.to("#ocean-grad stop:nth-child(3)", { attr: { "stop-color": "#0D2B45" } }, 0);
depth.to("#ocean-grad stop:nth-child(4)", { attr: { "stop-color": "#050E1A" } }, 0);
```

The sea darkens smoothly as you scroll, the same way his sky goes from warm sunset to deep purple night.

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
├── index.html          # page structure — SVG inline + invisible scroll div + scripts
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

1. **The HTML** — the SVG is pasted inline (not loaded as an `<img>`). Below it sits an empty `<div class="scrollElement">` that creates the scroll distance. Find the named groups inside the SVG (`<g id="h1-1">`, `<g id="clouds">`, etc.)
2. **The CSS** — the SVG is `position: fixed`, `height: 100vh` (pinned to the screen). The `.scrollElement` is `height: 6000px` (creates the scrollable distance). The SVG never moves; the invisible div is what you scroll through.
3. **The JS** — find the `speed` variable. Find the `gsap.to()` calls that multiply `speed` by different amounts per layer. Find `ScrollTrigger.create()` and notice `scrub: 3`. Notice how each scene is a separate `gsap.timeline()`.

Don't try to understand everything. Map the shape: fixed SVG + invisible scroll div → JS watches scroll position → JS moves named groups at different speeds.

**Step 1.2: Sketch the scene on paper**
Draw a rough vertical strip. Mark the layers from back to front:
- What's furthest away? (sky, distant clouds at the surface)
- What's in the middle? (wave layers, boat)
- What's nearest? (foreground wave, underwater close-ups, ocean floor)

Write a name next to each layer. These names become the group IDs in your SVG.

---

### Phase 2 — Build the SVG (Figma session)

**Step 2.1: Set up the canvas**
Create a new Figma file. Set the frame to **1440 × 900px** — this matches a widescreen viewport. The SVG will be this size, not taller. All your layers stack within this rectangle.

**Step 2.2: Set up the background gradient**
Create a rectangle the full size of the frame. Give it a vertical gradient fill — light blue at the top (sky/surface), progressively darker blue toward the bottom (deep water). Name this group `ocean-bg`. This gradient will be animated in JavaScript later, but it needs to exist in the SVG as a `<linearGradient>` with named stops.

**Step 2.3: Draw the wave layers**
Draw wavy lines across the full width using the pen tool. Close each shape at the bottom to make a filled region. Each wave is a different shade of blue — lighter waves near the top, darker toward the bottom. Make 4–6 wave layers. Name them by depth: `wave-far-1`, `wave-mid-2`, `wave-near-3`, etc.

Place them all within the frame, overlapping. They don't need to fill the whole frame — JavaScript will move them into view as you scroll.

**Step 2.4: Add imported elements**
Download SVG elements from OpenClipart or SVGRepo (fish, whale, boat, sea plants, seabed). Import into Figma. Place them at the depth where they belong — boat on the surface wave, fish near the middle, whale deeper, seabed at the bottom. Recolour them to match your palette.

**Step 2.5: Name every group**
Select each element or wave layer in the Figma layers panel and give it a clear name: `sky`, `wave-far-1`, `wave-mid-2`, `wave-near-3`, `boat`, `fish-jump`, `whale`, `seabed`. These names become the IDs that JavaScript will target.

**Step 2.6: Export as SVG**
Select the whole frame → Export → SVG. Open the exported file in a text editor and check:
- Your group names survived as `<g id="...">` tags
- The background gradient exported as a `<linearGradient>` with `<stop>` elements (you'll need to target these from JavaScript)

If the gradient didn't export cleanly, you may need to add it manually in the SVG code — we'll handle that in the HTML/CSS session.

---

### Phase 3 — Page structure (HTML/CSS session)

**Step 3.1: Create the HTML file**
Build the page skeleton. The structure matches his exactly:

```html
<body>
  <!-- The SVG — fixed to the viewport, never scrolls -->
  <svg class="parallax" viewBox="0 0 1440 900" preserveAspectRatio="xMidYMax slice">
    <!-- Your full SVG content goes here, inline -->
  </svg>

  <!-- The invisible scroll div — this creates the scroll distance -->
  <div class="scroll-container"></div>

  <!-- Scripts at the bottom -->
  <script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/gsap.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/ScrollTrigger.min.js"></script>
  <script src="js/main.js"></script>
</body>
```

**Step 3.2: CSS — pin the SVG, create scroll distance**

```css
body {
  margin: 0;
  padding: 0;
}

svg {
  display: block;
  width: 100%;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 3;
}

.scroll-container {
  position: absolute;
  height: 6000px;     /* controls how long the scroll journey feels */
  width: 100%;
  top: 0;
  z-index: 4;
}
```

**Step 3.3: Test — does the SVG show?**
Open in a browser. You should see your scene filling the viewport. The scrollbar should be visible (because of the 6000px div) but scrolling doesn't move anything yet. If you see the scene, this phase is done.

---

### Phase 4 — Parallax animation (JavaScript session)

**Step 4.1: Register ScrollTrigger**

```javascript
gsap.registerPlugin(ScrollTrigger);

let speed = 100;  // one variable to control the whole scene's feel
```

**Step 4.2: Animate one layer first**
Start with the furthest-back wave to prove the concept:

```javascript
let ocean = gsap.timeline();
ScrollTrigger.create({
  animation: ocean,
  trigger: ".scroll-container",
  start: "top top",
  end: "bottom 100%",
  scrub: 3
});

ocean.to("#wave-far-1", { y: 1.5 * speed }, 0);
```

Scroll and watch. The far wave should drift. If it moves, the concept is working.

**Step 4.3: Add more layers at different speeds**

```javascript
ocean.to("#wave-far-1",  { y: 1.5 * speed }, 0);  // far — moves least
ocean.to("#wave-mid-2",  { y: 3 * speed },   0);   // mid
ocean.to("#wave-near-3", { y: 5 * speed },   0);   // near — moves most
ocean.to("#boat",        { y: 2 * speed },   0);   // sits on the surface
ocean.to("#whale",       { y: 4 * speed },   0);   // deep, moves fast
ocean.to("#seabed",      { y: 6 * speed },   0);   // closest — fastest
```

The `0` at the end of each line means "start at the same time as the timeline begins." All layers animate together, just at different speeds.

**Step 4.4: Add the depth colour shift**
This is where the sea darkens as you scroll deeper:

```javascript
let depth = gsap.timeline();
ScrollTrigger.create({
  animation: depth,
  trigger: ".scroll-container",
  start: "top top",
  end: "bottom 100%",
  scrub: 2
});

// Animate each gradient stop toward darker colours
depth.to("#ocean-grad stop:nth-child(1)", {
  attr: { "stop-color": "#2B6E99" }   // sky blue → mid blue
}, 0);
depth.to("#ocean-grad stop:nth-child(2)", {
  attr: { "stop-color": "#1A4D6E" }   // shallow → darker
}, 0);
depth.to("#ocean-grad stop:nth-child(3)", {
  attr: { "stop-color": "#0D2B45" }   // mid → deep
}, 0);
depth.to("#ocean-grad stop:nth-child(4)", {
  attr: { "stop-color": "#050E1A" }   // deep → near black
}, 0);
```

You can also shift where the stops sit (their `offset` values) to control how quickly the darkness spreads. Tune by scrolling and watching.

**Step 4.5: Add character animations**
Make the fish jump, the whale drift, the boat bob:

```javascript
ocean.to("#fish-jump", {
  y: -200,
  x: 100,
  rotation: 15,
  ease: "none"
}, 0);
```

For a fish that flips direction mid-jump (like his bird), use ScrollTrigger callbacks:

```javascript
gsap.to("#fish-jump", {
  y: -200,
  x: 300,
  scrollTrigger: {
    trigger: ".scroll-container",
    start: "20% top",
    end: "50% 100%",
    scrub: 3,
    onEnter: function() {
      gsap.to("#fish-jump", { scaleX: 1 });      // facing right
    },
    onLeave: function() {
      gsap.to("#fish-jump", { scaleX: -1 });      // flip to face left
    }
  }
});
```

**Step 4.6: Tune by eye**
Adjust the `speed` variable, the multipliers per layer, the scrub values, and the gradient colours. This is where the feeling lives. Sit with it. Scroll slowly, scroll fast, scroll back up. Change one number at a time.

---

### Phase 5 — Polish (HTML/CSS + JS session)

**Step 5.1: Responsive check**
`preserveAspectRatio="xMidYMax slice"` on the SVG makes it fill the viewport and crop rather than letterbox. Test on different screen sizes. The scene should fill the screen without distortion.

**Step 5.2: Scroll distance tuning**
Change the `.scroll-container` height to make the journey longer or shorter. 6000px is his default. Try 4000px for a quicker dive, 8000px for a slow descent. This is independent of the SVG — you're just changing how much scrolling maps to the full animation.

**Step 5.3: Reset scroll on refresh**
His code resets the scroll position when you refresh the page, so the animation always starts from the top:

```javascript
window.onbeforeunload = function() {
  window.scrollTo(0, 0);
};
```

---

### Phase 6 (future) — Add the beach scene

Same pattern as his scene transitions. Add new groups to the SVG (sand, palms, crabs). Create a new `gsap.timeline()` with its own `ScrollTrigger` that activates later in the scroll (e.g. start at 70%). Increase the `.scroll-container` height to accommodate the extra scroll distance. The ocean scene doesn't change — you're adding a new act to the same film.

---

## Key concepts to understand

### What is a viewBox?
`viewBox="0 0 1440 900"` defines the SVG's internal coordinate system — 1440 units wide, 900 units tall. The browser scales this to fill whatever space you give the SVG on screen. Shapes are described in viewBox units, not pixels, which is why SVGs scale cleanly.

### What does `preserveAspectRatio="xMidYMax slice"` mean?
It tells the browser how to fit the SVG when the viewport's aspect ratio doesn't match the viewBox. `slice` means "fill the viewport completely, cropping if needed" (like `background-size: cover` in CSS). `xMidYMax` means "keep the bottom-centre visible when cropping." This ensures the ocean floor stays anchored to the bottom of the screen.

### Why inline SVG (not `<img>`)?
When an SVG is loaded as an `<img>`, it's a flat picture — JavaScript can't reach inside it. Inline SVG (pasted directly into the HTML) puts every group and shape into the DOM, where JavaScript can target them by ID. Parallax needs JavaScript to move the groups, so the SVG must be inline.

### What is a GSAP timeline?
A timeline groups multiple animations together so they can share one ScrollTrigger. Instead of writing a separate ScrollTrigger for every layer, you create one timeline, add all the layer animations to it, and attach one ScrollTrigger to the whole group. The `0` at the end of each `.to()` call means "start this animation at time 0 in the timeline" — i.e. all together.

### What does `scrub` do?
Without `scrub`, animations play once when triggered. With `scrub: true`, the animation tracks your scroll position directly. `scrub: 3` adds a 3-second smoothing delay — the animation drifts to catch up with your scroll rather than snapping. Higher = smoother and floatier.

### How does the gradient colour shift work?
SVG gradients are defined in `<defs>` with `<stop>` elements. Each stop has an `offset` (position, 0–1) and `stop-color`. GSAP can animate both using `attr: {}`. By targeting stops with CSS selectors like `#ocean-grad stop:nth-child(2)`, you change the gradient in real time. This is how the water darkens — the gradient stops shift toward darker colours as you scroll.

### Why separate files (HTML, CSS, JS)?
At this stage of learning, keeping each language in its own file makes it easier to find things and reduces the chance of breaking one thing while editing another. One concern per file. The SVG file is the exception — it goes inline into the HTML because JavaScript needs DOM access to the groups inside it.

---

## Differences from isladjan's version

| His version | Your version |
|-------------|-------------|
| 3 scenes (sunset → dusk → night) with transitions | 1 scene (ocean surface → deep sea) |
| Mountains and hills | Waves and underwater layers |
| Sky gradient shifts sunset → purple → night | Ocean gradient shifts bright blue → deep navy |
| Bird, bats, falling star, twinkling stars | Fish, whale, boat |
| viewBox 750 × 500 | viewBox 1440 × 900 (wider for modern screens) |
| Complex scene visibility toggling | No visibility toggling needed (one continuous scene) |

The core mechanics are identical: fixed SVG + invisible scroll div + speed multiplier per layer + scrub smoothing + gradient colour animation.

---

## Session plan

| Session | Discipline | What happens |
|---------|-----------|-------------|
| **Documentation** | This README | Project plan, architecture notes, decisions log. Update as the build progresses. |
| **Figma** | SVG design | Draw waves, import elements, name groups, set up gradient, export SVG. No code. |
| **HTML/CSS** | Page structure | Build index.html + style.css. Get the SVG visible and the scroll div working. No animation. |
| **JavaScript** | Animation | Write main.js. Wire up GSAP + ScrollTrigger. Parallax layers, depth colour shift, character animations. Tune the feel. |
| **Integration/QA** | Testing | Check everything works together. Responsive, smooth, no visual glitches. |

Each session stays in its lane. If a JS session reveals a layer needs splitting in the SVG, that's a note back to the Figma session — not a reason to start editing SVG code mid-animation-work.

---

## What you already know that applies here

From your freeCodeCamp progress (step ~56) and Novo work, you already understand:

- **Variables** — `let speed = 100` and the `y: 3 * speed` multipliers
- **Functions** — `gsap.to()` is a function call with an object as its argument
- **Template literals / strings** — the `"#wave-far-1"` selectors are strings targeting element IDs
- **Objects** — `{ y: 100, scrollTrigger: { scrub: 3 } }` is an object with a nested object

What's new:
- **The DOM** — JavaScript reaching into the HTML to find elements by ID
- **Libraries** — using someone else's code (GSAP) loaded from a CDN
- **CSS `position: fixed`** — pinning the SVG to the viewport
- **SVG structure** — groups, IDs, gradients, and stops inside an SVG file
- **Animating SVG attributes** — GSAP's `attr: {}` syntax for changing gradient colours

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
