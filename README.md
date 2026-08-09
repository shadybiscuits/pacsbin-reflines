# PacsBin Reference Lines

A bookmarklet that adds **cross-plane reference lines** and **synchronised
scrolling** to [PacsBin](https://pacsbin.com) viewers, which do not ship with
them.

Built for reading the PAUSE study, where cases are reviewed across several
series side by side and there is no way to tell where an axial slice sits on
the sagittal — the thing every clinical PACS gives you for free.

![status](https://img.shields.io/badge/type-bookmarklet-blue)
![deps](https://img.shields.io/badge/dependencies-none-green)

---

## What it does

**Cross-plane reference lines.** With two or more series in different planes,
a dashed line is drawn on each viewport showing where the other viewports'
current slices intersect it. Each source viewport gets its own colour.

**Synchronised scrolling.** With two or more series in the *same* plane —
three axials side by side, say, or pre- and post-contrast — scrolling one
viewport moves the others to the anatomically matching slice, not the matching
slice *index*. Series with different slice counts or start positions stay
aligned.

## How it works

PacsBin's Cornerstone instance does not populate the `imagePlaneModule`
metadata that `cornerstoneTools.referenceLines` needs, so the reference-line
tool has nothing to compute from. The bookmarklet supplies it:

1. **Registers a metadata provider** that reads the DICOM tags straight off the
   cached image — `ImageOrientationPatient` (0020,0037),
   `ImagePositionPatient` (0020,0032) and `PixelSpacing` (0028,0030) — and
   returns a well-formed `imagePlaneModule`.
2. **Assigns a shared `frameOfReferenceUID`** so Cornerstone will relate series
   that it would otherwise treat as unconnected.
3. **Derives the slice normal** as the cross product of the row and column
   cosines, and projects `ImagePositionPatient` onto it to get a scalar slice
   position that is comparable across series.
4. **Draws** on each `CornerstoneImageRendered`, coalesced through
   `requestAnimationFrame` so a scroll does not queue a redraw per frame.
5. **Syncs** on `CornerstoneNewImage`, matching by nearest slice position and
   guarded by a re-entrancy flag so viewports do not drive each other in a loop.

Series are treated as parallel when both cosine dot products exceed 0.95, which
tolerates the small obliquity between series that are nominally the same plane.

## Install

Open [`instructions.html`](instructions.html) and drag the button to your
bookmarks bar, or copy [`bookmarklet.js`](bookmarklet.js) and save it as a
bookmark URL by hand.

## Use

1. Open a PacsBin case.
2. Choose a multi-panel layout and put a different series in each panel.
3. Click the bookmarklet.

Click it once per case — it re-binds on page navigation — and again if you
change the layout after activating.

## Limitations

- Needs series that carry orientation and position tags. Secondary captures and
  most ultrasound will not have them.
- The shared frame-of-reference is asserted, not read. Series acquired in
  genuinely different frames of reference will be related anyway, and lines
  will be wrong; in practice this only matters across separate studies.
- No build step, no dependencies, no configuration — it uses the Cornerstone,
  cornerstoneTools, cornerstoneMath and jQuery instances the page already has.
  A PacsBin upgrade that changes those could break it.

## Licence

MIT. Not affiliated with PacsBin.
