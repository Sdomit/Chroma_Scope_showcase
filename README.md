# ChromaScope

A technical color-analysis tool for inspecting and comparing RGB color spaces, transfer curves, white points, chromatic adaptation, and pipeline concepts — without collapsing unrelated ideas into a single label.

> **Private source — this repository is a project showcase.**

---

## The Problem It Solves

Most color tools either oversimplify or conflate distinct concepts under one convenient label. ChromaScope is built around a stricter rule: **a gamut is not a transfer curve, a transfer curve is not a delivery system, and a pipeline diagram is not a rendered image.** Many practical color mistakes come from comparing unlike concept classes as though they were directly equivalent.

ChromaScope keeps those boundaries visible while still letting users compare everything in one workspace.

---

## What It Analyzes

- CIE gamut geometry and chromaticity relationships
- RGB primaries, white points, and gamut fills
- Transfer curves and log encodings — SDR, HDR, gamma, log, camera models
- Chromatic adaptation (Bradford and others)
- Pipeline roles and staged transform workflows
- Image-based diagnostic transform previews with pixel inspection

The tool keeps the following concept classes explicitly distinct:

| Class | Examples |
|---|---|
| Gamut / primaries | sRGB, Rec.2020, ACEScg, DCI-P3, camera gamuts |
| Transfer functions | sRGB, PQ/ST 2084, HLG, ACEScc, ACEScct, BT.1886 |
| Chromatic adaptation | Bradford, D65 ↔ D60, white-point shifts |
| Pipeline stages | source encoding → working space → output transform → delivery |
| Signal encodings | log, linear, scene-referred vs display-referred |

---

## Viewer Modes

### Gamut View
Chromaticity geometry on **CIE 1931 xy** and **CIE 1976 u'v'** diagrams. Overlays include spectral locus, black-body locus, gamut fills, area comparisons, and labels. Useful for comparing sRGB, Rec.709, Display P3, Rec.2020, ACEScg, DCI-P3, camera gamuts, and custom spaces.

### Curve View
Transfer behavior across curve families — forward and inverse, with selectable axis contracts (signal, code values, linear, stops, or nits where an absolute mapping exists). Strict compare logic prevents mixed-family chart semantics.

### Dual View
Gamut and curve analysis side by side — chromaticity and transfer behavior together, without pretending they are the same property.

### Pipeline View
Staged transform understanding: source encoding → working-space transform → adaptation → display transform → delivery context. Labeled as conceptual, not photometric.

### Image Pipeline View
A generated diagnostic chart previewed through a full source → target transform chain. Stages: Source, Linearized, Adapted, Target RGB, Encoded Preview. Overlays: Clean, Delta, Out of Gamut, Clipping. Built for inspecting transform stress, clipping pressure, and out-of-gamut behavior.

### CIE 3D Viewers
Standalone **CIE xyY** and **CIE XYZ** 3D viewers. Interactive, responsive (desktop + mobile), with drag-to-inspect, spectral locus, reference planes, and shared inspector panels.

### Transform Compare
Side-by-side source and target space picker with diagnostic output — for comparing how image data moves through a transform pair.

### Learning Mode
Guided explanations of color-science concepts tied to the active viewer context.

---

## Screenshots

**Workbench overview**
![Workbench overview](screenshots/01-workbench-overview.png)

**Transform Compare — diagnostic output**
![Transform Compare diagnostic](screenshots/02-transform-compare-diagnostic.png)

**Pipeline Lab — trace view**
![Pipeline Lab trace](screenshots/03-pipeline-lab-trace.png)

**Pipeline Lab — graph view**
![Pipeline Lab graph](screenshots/04-pipeline-lab-graph.png)

**Pipeline Lab — image probe mode**
![Pipeline Lab image probe](screenshots/05-pipeline-lab-image-probe-mode.png)

**Transform Compare — source picker**
![Transform Compare source picker](screenshots/06-transform-compare-source-picker.png)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | TypeScript |
| Frontend | React 18 + Vite (multi-page build) |
| Desktop shell | Tauri v2 (Rust) |
| Color math | Custom `src/lib/color/` — matrix derivation, XYZ ↔ xy ↔ u'v', adaptation, curve modeling |
| OCIO / ACES | OpenColorIO config integration, ACES pipeline definitions |
| UI components | shadcn/ui + custom design system with full component library |
| Testing | Playwright end-to-end tests |
| Build | Vite multi-page (`index.html`, `transform-compare.html`, `pipeline-lab.html`, `cie-3d-viewer.html`, `cie-xyy-viewer.html`, `learning.html`) |

---

## Architecture

```
src/
  features/               Feature-based modules (gamut, curves, adaptation,
                          pipeline, image-pipeline, learning, catalog, export)
  lib/color/              Math library — matrix derivation, XYZ/xy/u'v'
                          conversions, chromatic adaptation, curve modeling
  components/             Shared UI (design system, charts, panels, inspector)
  pages/                  Entry points per Vite page

src-tauri/src/            Rust desktop shell (Tauri v2)

e2e/                      Playwright end-to-end test suite

OCIO/                     OpenColorIO + ACES config files
```

**Multi-page build:** each major viewer mode is its own Vite entry point and standalone HTML page — isolation prevents cross-mode state leakage and keeps bundle sizes small per page.

**Feature-based structure:** each analysis domain (gamut, curves, pipeline, etc.) owns its logic, state, and rendering. The shared math library is the only coupling between features.

**Design system:** a full component library with documented previews used across all pages — consistent visual language without a third-party component framework.

---

## Key Engineering Decisions

**Why separate concept classes?**
Color tools that flatten gamut, transfer, and pipeline into one list are easier to browse but harder to reason about. ChromaScope treats each class as a distinct technical object. This forces correctness — you can't accidentally compare a gamut width to a transfer knee if they are never on the same axis.

**Why an accuracy disclosure system?**
Some views are mathematically exact; others are normalized previews, heuristics, or conceptual-only explanations. ChromaScope labels each one. *Strict Technical Mode* hides preview-only and heuristic views by default. The goal: prevent a diagnostic chart from being mistaken for authoritative final appearance.

**Why a multi-page Vite build?**
Each viewer mode has meaningfully different state and rendering requirements. Separate entry points give each page its own bundle — no shared global state, smaller initial loads, and clean isolation for E2E testing.

**Why a custom color math library?**
Off-the-shelf color libraries often abstract away the intermediate values (XYZ matrices, adaptation matrices, per-channel curve tables) that ChromaScope needs to expose in inspector panels. Building the math layer in-house means every intermediate result is traceable to the chart that shows it.

**Why Tauri for the desktop shell?**
Same reasoning as the offline nature of the tool — Tauri produces a small binary (~5 MB) using the OS WebView rather than bundling Chromium. The multi-page web app runs identically in both browser and desktop contexts.

---

## Who It's For

- Colorists and finishing artists comparing delivery targets
- VFX and CG artists working in ACES or ACEScg pipelines
- Imaging engineers validating transform behavior
- Technical directors and pipeline developers
- Students learning color science with a technically honest reference

---

## Status

Active development. Private source — this repository is a project showcase.
