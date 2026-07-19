# Bounded Ink

## A procedural geometric codec for manga pages

### Proposal

A manga page should not be stored primarily as a rectangular field of pixels. It should be stored as a compact, executable description of ink.

Traditional image compression asks:

> How can we encode these pixels using fewer bytes?

Bounded Ink asks a different question:

> What small collection of bounded mathematical instructions would cause a renderer to draw this page?

The system converts a completed black-and-white manga page into an ordered program of curves, regions, patterns, text, and reusable visual components. The page becomes something closer to source code than a conventional image.

This is not merely raster-to-SVG conversion. The proposed format understands the recurring visual grammar of manga:

* Panels
* Inked contours
* Variable-width strokes
* Solid-black regions
* Negative space
* Speech balloons
* Typeset or handwritten text
* Screentones
* Hatching
* Motion lines
* Impact effects
* Repeated character features
* Reading order

## Central representation

A page is represented as:

[
P = G + I + T + R
]

Where:

* (G) is geometric structure.
* (I) is the procedural ink program.
* (T) is text and reading information.
* (R) is an optional residual correction layer.

The fundamental drawing element is a bounded function:

[
E_i = (f_i,\ D_i,\ S_i,\ L_i)
]

Where:

* (f_i) is a curve, region, pattern, or reusable component.
* (D_i) is its bounded domain.
* (S_i) contains stroke and fill properties.
* (L_i) determines drawing order.

The complete page is:

[
P = \mathcal{R}(E_1,E_2,\ldots,E_n)
]

where (\mathcal{R}) is a deterministic renderer.

## The distinctive idea: reconstruct the artist’s decisions

Ordinary vectorization attempts to trace visible pixel boundaries. Bounded Ink instead attempts to recover a compact approximation of the decisions that produced them.

Rather than storing 8,000 individual screentone dots:

```text
tone pattern 14
inside region 38
scale 0.72
rotation 35°
density 0.31
```

Rather than storing 300 separate speed lines:

```text
radial field
origin (0.61, 0.43)
line count 300
angular interval [18°, 167°]
length distribution L₇
seed 42819
```

Rather than independently encoding two nearly identical eyes:

```text
component eye_17
transform(position, scale, rotation)
local corrections [...]
```

This converts repeated visual behavior into reusable mathematical vocabulary.

## The Ink Grammar

The proposal introduces an intermediate representation called **Ink Grammar**:

```text
PAGE
 ├── PANEL
 │    ├── FIGURE
 │    │    ├── SKELETON
 │    │    ├── SILHOUETTE
 │    │    ├── CONTOUR
 │    │    └── DETAIL
 │    ├── INK_REGION
 │    ├── TONE_FIELD
 │    ├── MOTION_FIELD
 │    ├── BALLOON
 │    └── TEXT
 └── READING_ORDER
```

Each component has a compact binary form and a readable textual form.

Example:

```text
panel p1 rect 40 50 1120 810

figure f1 in p1
    skeleton human_pose_73
    transform 0.62 0.48 1.10 -4deg
    silhouette path s291
    face component face_18
    hair component hair_42

tone t1
    region r17
    pattern dots
    spacing 6
    radius 1.4
    angle 35deg

balloon b1 ellipse 760 130 280 190
text b1 "Where are you going?"
```

This representation is:

* Renderable
* Editable
* Searchable
* Resolution-independent
* Deterministic
* Compressible
* Suitable for machine learning

## Three reconstruction modes

### Structural mode

Preserve panels, silhouettes, major contours, text, solid fills, and procedural tones.

The page remains recognizable but visibly simplified.

### Perceptual mode

A trained renderer receives the Ink Grammar and reconstructs aesthetically coherent detail:

[
\widehat{P} = Decoder(G,I,T,Z)
]

Here (Z) is a small page-specific style vector.

The output need not reproduce every original stroke. It must preserve composition, characters, text, action, and visual atmosphere.

### Archival mode

Store a compressed residual:

[
R = P-\widehat{P}
]

Exact reconstruction becomes:

[
P = \widehat{P}+R
]

The shared renderer predicts most of the page; the residual preserves information it could not predict.

## Reusable visual atoms

Across a complete manga work, many visual forms repeat:

* A character’s eyes
* Hairstyle
* Face shape
* Clothing
* Speech-balloon style
* Sound-effect lettering
* Panel borders
* Screentones
* Background objects

Bounded Ink constructs a per-work component dictionary:

```text
work-components/
├── character_1_eye
├── character_1_hair
├── character_1_profile
├── uniform_front
├── uniform_side
├── balloon_normal
├── balloon_shout
└── tone_shadow
```

Pages reference and transform these components instead of independently encoding every occurrence.

This creates a second level of compression:

[
Work = SharedVisualVocabulary + PagePrograms
]

The more consistent the artist, the cheaper later pages become.

## Mathematical page families

Pages can also be clustered by structural similarity:

```text
conversation
close-up reaction
establishing shot
action impact
vertical reveal
four-panel comedy
```

A family supplies a reusable layout basis. Individual pages store deviations from that basis:

[
Page_i = FamilyBasis_j + \Delta_i
]

This is not merely visual similarity. It is compression based on the repeated rhetoric of sequential art.

## Proposed file format

```text
work.bink/
├── manifest.json
├── dictionary.bin
├── characters.bin
├── patterns.bin
├── pages/
│   ├── 0001.ink
│   ├── 0002.ink
│   └── ...
├── latents/
│   └── pages.bin
└── residuals/
    └── optional/
```

Manifest:

```json
{
  "format": "bounded-ink",
  "version": 1,
  "work_id": "name-derived-parent-id",
  "content_sha256": "canonical-content-hash",
  "canvas": [2400, 2000],
  "pages": 200,
  "renderer": {
    "name": "bounded-ink-renderer",
    "version": "1.0",
    "model_sha256": "..."
  }
}
```

## Initial experiment

Use 100 clean black-and-white manga pages representing:

* Simple amateur linework
* Professional clean linework
* Heavy screentones
* Dense hatching
* Action scenes
* Dialogue-heavy pages
* Minimalist pages

Compare:

1. Original PNG/WebP
2. Conventional SVG tracing
3. Bounded Ink structural mode
4. Bounded Ink perceptual mode
5. Bounded Ink archival mode with residuals

Measure:

* Bytes per page
* Curve and component count
* Rendering time
* OCR preservation
* Panel-boundary accuracy
* Structural similarity
* Human preference
* Exact reconstruction cost
* Resolution independence

## First prototype

The first implementation should avoid training a large model. Establish whether the geometric premise works:

1. Detect panels.
2. Separate text and balloons.
3. Threshold line art.
4. Trace major connected contours.
5. Fit piecewise Bézier curves.
6. Detect solid-black regions.
7. identify repeating screentones.
8. Replace tones with procedural parameters.
9. Store OCR text separately.
10. Render the mathematical description.
11. Compare size and appearance with the source.

Only after that should a learned renderer fill missing detail.

## Research claim

The proposal’s central claim is:

> Manga compression should operate on the grammar of ink-making rather than treating a page as an undifferentiated photograph.

The ultimate object is not merely a smaller image file. It is a **portable mathematical manuscript**:

* A page can be rendered at any resolution.
* Text can be translated without repainting the artwork.
* Panels can be rearranged responsively.
* Characters and objects become addressable.
* Search can operate on page structure.
* Corruption can be localized to individual instructions.
* A simplified drawing can become a finished page.
* The same representation supports compression, editing, restoration, translation, and generation.
