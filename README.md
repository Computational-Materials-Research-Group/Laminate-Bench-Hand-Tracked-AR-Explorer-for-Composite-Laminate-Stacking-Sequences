# Laminate Bench: Hand-Tracked AR Explorer for Composite Laminate Stacking Sequences

<p align="center">
  <img src="https://img.shields.io/badge/WebXR-Immersive%20AR-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Three.js-r160-black?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Meta%20Quest-Passthrough%20AR-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Hand%20Tracking-Pinch%20%2B%20Fingertip%20Keyboard-purple?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Classical%20Lamination%20Theory-Live%20ABD-teal?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Web%20Speech%20API-Narrated-yellow?style=for-the-badge"/>
</p>

<p align="center">
  Put on a <b>Meta Quest</b>, type a stacking sequence such as <code>[0/±45/90]s</code> on a
  floating keyboard with your fingertip, and the laminate appears in your real room as an
  exploded stack of plies, each colored by its angle and drawn with its fiber direction. Below
  the stack, a <b>polar plot of Ex(θ)</b> shows how in-plane stiffness changes with loading
  angle, and a floating panel reports <b>Ex, Ey, Gxy, νxy</b>, thickness, and whether the
  laminate is symmetric and balanced. Pinch to move it, pinch with both hands to scale and turn
  it, and listen as each reference laminate is narrated with a short note on why that layup is
  used. Everything is computed live in the browser with <b>classical lamination theory (CLT)</b>,
  from a single self-contained HTML file with nothing else to install.
</p>

<img width="896" height="442" alt="image" src="https://github.com/user-attachments/assets/55b3d62b-27f4-49b6-9d2c-c28d3cbf5e55" />


---

## Read This First: What This Viewer Gives You, and What It Does Not

This is a **teaching and design-intuition tool** built on classical lamination theory, not a
certified structural analysis package. Every number is computed in the browser from the ply
properties and stacking sequence you enter; nothing is loaded from an external dataset.

| This viewer gives you | This viewer does NOT give you |
|---|---|
| Live CLT: transformed stiffness Q̄ for every ply, the full A, B and D matrices, and effective Ex, Ey, Gxy, νxy | Failure prediction (no Tsai–Wu, Tsai–Hill, Hashin or first-ply failure envelopes) |
| A polar plot of Ex(θ) at 3° steps, recomputed whenever the layup or material changes | Hygrothermal analysis. The warning that unsymmetric panels warp after cure is qualitative, not calculated |
| Symmetric and balanced checks, with plain-language explanations of B and A16/A26 coupling | Transverse shear or thick-plate effects (CLT assumes thin Kirchhoff plates) |
| Passthrough AR on Meta Quest with pinch-to-move, two-hand scale and rotate, and a fingertip keyboard | Controller support, or hand tracking on hardware without a WebXR hand-tracking implementation |
| Spoken notes for six reference laminates, plus a generated summary for custom layups (Web Speech API) | Certified material data. Ply properties are textbook values, fine for teaching, to be replaced with measured lamina data for research |
| One self-contained HTML file, no build step, no bundled data files | A hosted, persistent deployment. You still need to serve the file yourself |

---

## The Mechanics: Classical Lamination Theory in the Browser

Each ply's reduced stiffness is built from the four in-plane lamina constants, then rotated to
the ply angle θ with the standard transformation (c = cos θ, s = sin θ):

```
ν21 = ν12·E2/E1,   Δ = 1 − ν12·ν21
Q11 = E1/Δ,  Q22 = E2/Δ,  Q12 = ν12·E2/Δ,  Q66 = G12

Q̄11 = Q11c⁴ + 2(Q12 + 2Q66)s²c² + Q22s⁴
Q̄22 = Q11s⁴ + 2(Q12 + 2Q66)s²c² + Q22c⁴
Q̄12 = (Q11 + Q22 − 4Q66)s²c² + Q12(s⁴ + c⁴)
Q̄66 = (Q11 + Q22 − 2Q12 − 2Q66)s²c² + Q66(s⁴ + c⁴)
Q̄16 = (Q11 − Q12 − 2Q66)sc³ + (Q12 − Q22 + 2Q66)s³c
Q̄26 = (Q11 − Q12 − 2Q66)s³c + (Q12 − Q22 + 2Q66)sc³
```

The ABD matrices are then integrated through the thickness, with ply 1 at z = −h/2:

```
A = Σ Q̄k (zk − zk−1)          [MN/m]
B = ½ Σ Q̄k (zk² − zk−1²)       [kN]
D = ⅓ Σ Q̄k (zk³ − zk−1³)       [N·m]
```

Effective in-plane (membrane) properties come from the compliance a = A⁻¹:

```
Ex = 1/(h·a11),  Ey = 1/(h·a22),  Gxy = 1/(h·a66),  νxy = −a12/a11
```

### Verified Against Known Results

With carbon/epoxy T300/5208 plies, the implementation reproduces standard textbook values:

| Laminate | Result | Why it is correct |
|---|---|---|
| `[0]8` | Ex = 181.0, Ey = 10.3, Gxy = 7.17 GPa, νxy = 0.28 | Returns exactly the lamina constants E1, E2, G12, ν12 |
| `[0/±45/90]s` | Ex = Ey = 69.68 GPa, polar plot constant at every angle | Matches the quasi-isotropic worked example in Kaw, *Mechanics of Composite Materials* |
| `[0/90]2s` | Gxy = 7.17 GPa | Equals G12, since 0° and 90° plies add no in-plane shear stiffness |
| `[±45]2s` | Gxy = 46.59 GPa | Equals (Q11 + Q22 − 2Q12)/4, the closed-form result for ±45° plies |
| `[0/90]4` | B ≠ 0 | Unsymmetric, so bending–extension coupling appears; every symmetric layup gives B = 0 |
| `[30/0]2s` | A16 ≠ 0 | Unbalanced, so shear–extension coupling appears; every balanced layup gives A16 = A26 = 0 |

### Known Limitations of the Mechanics

- **Unsymmetric laminates.** Effective moduli use A⁻¹ only, which assumes bending is
  restrained. A free unsymmetric laminate is more compliant; the exact value uses the inverse of
  the full 6×6 ABD matrix. For symmetric laminates the result is exact.
- **Sign convention for B.** Ply 1 sits at z = −h/2, following Kaw. Texts that use the
  opposite convention get B with the opposite sign and the same magnitude.
- **Scope.** Linear-elastic CLT only: no failure, no thermal or moisture strains, no transverse
  shear.

## Why a Polar Plot of Ex(θ)

A table of four numbers hides the most important thing about a laminate: how its stiffness
depends on load direction. The polar plot makes that visible at a glance. It is computed by
rotating every ply by −φ, rebuilding A, and taking 1/(h·a11), for φ from 0° to 357° in 3° steps:

```
for φ in 0..357 step 3:
    A(φ) = abd(material, angles.map(θ => θ − φ))
    Ex(φ) = 1 / (h · inv(A(φ))[0][0])
```

The plot lies flat under the stack, aligned with the ply axes, so the 0° direction on the plot is
the 0° fiber direction in the plies above it. The outer white ring is drawn at E1 of the chosen
material, so every plot is on the same scale as a unidirectional ply of that material. A
quasi-isotropic layup gives a circle, a cross-ply pinches at 45°, and an unbalanced layup is
visibly tilted.

## Stacking Sequence Notation

The parser accepts standard laminate code as well as a plain list of angles:

| Input | Expands to |
|---|---|
| `[0/±45/90]s` | 0, 45, −45, 90, 90, −45, 45, 0 |
| `[0/90]2s` | 0, 90, 0, 90, 90, 0, 90, 0 |
| `[0_2/±30]s` | 0, 0, 30, −30, −30, 30, 0, 0 |
| `[0/90]4` | 0, 90, 0, 90, 0, 90, 0, 90 (unsymmetric) |
| `0,45,-45,90` | 0, 45, −45, 90 |

`±` expands to a +/− pair and `∓` to a −/+ pair; `+-` and `+/-` are accepted as `±`. A trailing
number repeats the bracketed group, and a trailing `s` mirrors it about the mid-plane. `_n` (or
`_(n)`) repeats a single ply. Angles are normalized to the range −90° to +90°, decimals are
allowed, and the total is limited to 48 plies. Invalid input produces a specific error message,
for example `"3a" is not a ply angle`.

## Ply Materials

| Material | E1 (GPa) | E2 (GPa) | G12 (GPa) | ν12 | Ply thickness (mm) | Source |
|---|---|---|---|---|---|---|
| Carbon/epoxy (T300/5208) | 181 | 10.3 | 7.17 | 0.28 | 0.125 | Kaw, *Mechanics of Composite Materials* |
| E-glass/epoxy | 38.6 | 8.27 | 4.14 | 0.26 | 0.125 | Kaw, *Mechanics of Composite Materials* |
| Aramid/epoxy (Kevlar 49) | 76 | 5.5 | 2.3 | 0.34 | 0.125 | Typical handbook values |

## Reference Laminates

| # | Name | Code | What it shows |
|---|---|---|---|
| 1 | Unidirectional | `[0]8` | Very high stiffness along the fibers, low across them |
| 2 | Cross-ply | `[0/90]2s` | Equal Ex and Ey, but the polar plot pinches at 45° because no fibers carry shear |
| 3 | Angle-ply | `[±45]2s` | Shear carried by the fibers; high Gxy, low Ex |
| 4 | Quasi-isotropic | `[0/±45/90]s` | Circular polar plot; same in-plane stiffness in every direction |
| 5 | Unbalanced | `[30/0]2s` | Symmetric but A16 ≠ 0, so axial load produces shear; tilted polar plot |
| 6 | Unsymmetric | `[0/90]4` | B ≠ 0, so stretching couples to bending; such panels warp after cure |

## Why Ply Colors Are Tied to Angle

Each ply is colored by interpolating between five anchors, so the same angle always has the same
color in every laminate:

```
−90° blue   −45° green   0° red   +45° amber   +90° blue
```

Fiber lines are drawn on top of every ply as thin flat ribbons clipped to the ply square, not as
1-pixel lines, because single-pixel lines shimmer and nearly disappear in a headset.

## Keeping the Stack Readable at Any Ply Count

The plate is 16 cm square. Ply pitch depends on the spacing slider and is capped so that the
whole stack stays about 20 cm tall however many plies you enter:

```
pitch     = min(0.0045 + spacing × 0.018, max(0.2 / n, 0.0045))   // meters
thickness = min(0.0035, 0.8 × pitch)
```

Per-ply angle labels are shown only when the pitch is at least 9 mm, so they never overlap on
thin, densely stacked laminates.

## Reference Space and Placement in AR

The session requests `local-floor`, `hand-tracking` and `dom-overlay` as optional features, then
calls `renderer.xr.setReferenceSpaceType('local-floor')`, falling back to `local` if the headset
does not grant a floor-calibrated space.

Placement is anchored to your head pose, not to a fixed world coordinate. Three frames into the
session, `placeInFront()` reads the camera's position and horizontal facing direction and places:

```
Laminate:  0.50 m in front of you, 0.15 m below eye level, facing you
Keyboard:  0.34 m in front of you, 0.40 m below eye level, tilted back 0.6 rad (about 34°)
```

The keyboard is tilted like a lectern so you can press it with your fingertip without raising
your arm. Placement runs once per session; after that, the laminate and keyboard stay put in your
room so you can walk around them. Pinching both hands away from the laminate at the same time
runs `placeInFront()` again.

## The AR Keyboard

The 2D side panel is hidden in AR, and DOM overlays render unreliably in headsets, so text entry
uses a 3D keyboard built from textured planes (30 cm wide, 4.6 cm keys):

```
[0/±45/90]s|                   ← display, with error messages in red
  7   8   9   [   ]   Del
  4   5   6   /   ±   Clear
  1   2   3   -   _   s
  0   ,   .   [    Build    ]
  Material | Spacing − | Spacing + | Hide
```

Keys are pressed with the **index fingertip**, not by pinching. Each frame the fingertip is
transformed into the keyboard's local frame:

```
press    when local z < 0.012 m (at the key face) and the tip is inside a key's rectangle
re-arm   when the tip moves back past z > 0.028 m or leaves the key
debounce 150 ms between any two presses
```

Each press gives a 1.4 kHz click and a small key dip. **Build** runs the same parser and CLT
update as the desktop Build button. **Hide** folds the keyboard away, leaving a **Keyboard**
button to bring it back. Pinches that start near the keyboard are ignored by the laminate
gestures, so typing never switches laminates by accident.

## Audio and Speech

Narration uses the browser's built-in **Web Speech API** (`speechSynthesis`), so no audio assets
are downloaded. Each reference laminate has a short note, followed by the computed Ex and Ey;
custom layups get a generated summary of ply count, symmetry and balance.

On Meta Quest, entering an immersive session can leave `speechSynthesis` paused, after which
`speak()` calls queue silently. This file uses the same fix as the TPMS gallery:

```js
synth.cancel(); synth.resume();              // before every utterance
setInterval(() => synth.resume(), 4000);     // background guard for the life of the page
// plus resume() on sessionstart
```

Speech only plays after a real user gesture (a tap, key press, or entering AR), and the
**Read a short note aloud** checkbox turns it off.

---

## Interaction Model

| Context | Gesture / Action | Effect |
|---|---|---|
| Desktop | Type a layup and press **Build** or Enter | Rebuilds the stack, polar plot, properties and ABD matrices |
| Desktop | Click a reference laminate | Loads it and reads its note aloud |
| Desktop | Ply material / ply spacing controls | Switches material or spreads the plies apart |
| Desktop | Drag / scroll on the stage | Orbits / zooms the camera (`OrbitControls`) |
| AR (Quest) | Press keys with your index fingertip | Types on the floating keyboard; **Build** applies the layup |
| AR (Quest) | Pinch on or near the laminate and move | Moves it (amber dot) |
| AR (Quest) | Pinch with both hands on the laminate | Scales (0.3× to 4×) and turns it |
| AR (Quest) | Short pinch away from the laminate, right hand | Next reference laminate (blue dot) |
| AR (Quest) | Short pinch away from the laminate, left hand | Previous reference laminate |
| AR (Quest) | Both hands pinch away at the same time | Brings the laminate and keyboard back in front of you |

A pinch starts when the thumb and index tips come within 1.8 cm and ends when they separate past
3.5 cm; the gap between the two thresholds stops the pinch from flickering on and off.

---

## Repository Structure

```
laminate-bench/
|
|-- index.html   # The entire app: markup, styles, CLT solver, three.js scene, hand tracking and AR keyboard
|-- README.md
```

`three.js` r160 and `OrbitControls` are loaded from `cdn.jsdelivr.net` through an import map, and
the Barlow fonts from Google Fonts, so there is nothing to build and nothing else to host.

---

## How to Run

### Requirements

- A modern desktop browser with WebGL, or
- A Meta Quest headset with Hand Tracking enabled (Settings &gt; Movement Tracking)
- Any static file host reachable over HTTPS from the headset (Netlify Drop, GitHub Pages)

### Step 1: Open it on desktop

Open the HTML file directly or load the hosted URL. The quasi-isotropic laminate loads first.
Type a layup, try the reference laminates, and read the ABD matrices in the side panel.

### Step 2: Host it for the headset

WebXR's `immersive-ar` mode requires a secure context. Name the file `index.html` and drag its
folder onto [app.netlify.com/drop](https://app.netlify.com/drop), or enable GitHub Pages on the
repository.

### Step 3: Open it on the Meta Quest

Open the hosted URL in the Quest browser and tap **Enter AR**. Accept the passthrough and hand
tracking permission prompts.

### Step 4: Find the laminate

The laminate appears half a metre in front of you, slightly below eye level, with its
properties panel to the right and the keyboard below.

### Step 5: Interact

Type a layup with your fingertip and press **Build**, or pinch away from the laminate to step
through the reference laminates. Pinch the laminate to move it, and use both hands to scale and
turn it.

---

## Common Errors and Fixes

| Symptom | Cause | Fix |
|---|---|---|
| Blank stage on desktop | WebGL unavailable, or three.js failed to load from the CDN | Check the browser console; confirm access to `cdn.jsdelivr.net` |
| Button reads "AR is not available in this browser" | The browser or device does not support `immersive-ar`, or the page is not served over HTTPS | Serve over HTTPS and open it in the Quest browser |
| Enter AR does nothing when opened from a claude.ai link | The embedding page may not permit immersive sessions | Host the file yourself on Netlify or GitHub Pages |
| Pinching does nothing | Hand Tracking is off, or you are holding controllers | Enable Settings &gt; Movement Tracking &gt; Hand Tracking and put the controllers down |
| Keys do not register | The fingertip is not reaching the key face, or it has not been pulled back to re-arm | Press through the key, then withdraw about 3 cm before the next press |
| Keys press twice | The finger is hovering at the press depth | Withdraw fully between presses; raise the 150 ms debounce if needed |
| Pinching away jumps to a reference laminate while typing | The pinch started outside the keyboard's ignore zone | Keep typing hands within a few centimetres of the keyboard |
| Laminate or keyboard is out of reach | You moved after the session started | Pinch both hands away from the laminate at the same time |
| No narration | The checkbox is off, no user gesture yet, or the Quest voice pack has not loaded | Tick the checkbox, tap once on the page before entering AR, and wait a moment after a system update |
| Red error on the keyboard display | The layup could not be parsed | Read the message, fix the token with **Del**, then press **Build** again |

---

## Extending the Viewer

| Extension | What to change |
|---|---|
| Add a ply material | Add an entry to `MATERIALS` with `label`, `E1`, `E2`, `G12`, `v12` and `t` (GPa and mm) |
| Add a reference laminate | Add an entry to `PRESETS` with `name`, `code` and `note` |
| Exact moduli for unsymmetric laminates | In `effective()`, invert the full 6×6 ABD matrix instead of A alone |
| Add failure envelopes | Compute first-ply failure (for example Tsai–Wu) per ply at each φ and draw a second polar curve |
| Change polar resolution | The `step` argument of `polarEx()` (default 3°) |
| Change where things appear in AR | The distances and offsets in `placeInFront()` |
| Change pinch sensitivity | The `0.018` / `0.035` thresholds in `updateHands()` |
| Change keyboard press depth | The `0.012` / `0.028` thresholds in `updateKeyboard()` |
| Change ply colors | The anchors in `ANCH` |

---

## Citation

If you use or adapt this viewer, a citation template is provided below:

```bibtex
@software{mishra_2026_laminatebench,
  author    = {Mishra, Akshansh},
  title     = {Laminate Bench: Hand-Tracked AR Explorer for Composite Laminate Stacking Sequences},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.23004229},
  url       = {https://doi.org/10.5281/zenodo.23004229}
}
```

> Mishra, A. (2026). *Laminate Bench: Hand-Tracked AR Explorer for Composite Laminate Stacking Sequences* [Computer software]. Zenodo. https://doi.org/10.5281/zenodo.23004229

---

## Author

**Akshansh Mishra**
GitHub: [akshansh11](https://github.com/akshansh11)

---

## License

<p align="center">
  <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">
    <img alt="Creative Commons Licence" style="border-width:0; margin: 12px 0;"
      src="https://licensebuttons.net/l/by-nc/4.0/88x31.png"/>
  </a>
  <br/>
  <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">
    Creative Commons Attribution-NonCommercial 4.0 International License
  </a>
</p>

This work is licensed under a **Creative Commons Attribution-NonCommercial 4.0 International License**.

You are free to:
- **Share**: copy and redistribute the material in any medium or format
- **Adapt**: remix, transform, and build upon the material

Under the following terms:
- **Attribution**: You must give appropriate credit to Akshansh Mishra ([akshansh11](https://github.com/akshansh11)) and link back to this repository
- **NonCommercial**: You may not use the material for commercial purposes

Copyright (c) 2026 Akshansh Mishra. All rights reserved for commercial use.
