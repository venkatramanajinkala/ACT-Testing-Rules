# a11y-rule-fixtures

Internal regression-test fixtures for 15 accessibility ACT rules. Each rule
has one intentional FAIL example and one corrected PASS example, using the
same headings so results are easy to diff side by side.

All files listed below — including `media/demo-video.mp4`,
`fixtures/untagged-document.pdf`, and `fixtures/tagged-document.pdf` — are
included in this package. Nothing needs to be created manually.

## Directory structure

```
a11y-rule-fixtures/
├── accessibility-rules-fail.html
├── accessibility-rules-pass.html
├── scrollable-region-focus-demo.html
├── sticky-header-focus-demo.html
├── media/
│   ├── demo-video.mp4
│   ├── demo-video-correct.vtt
│   └── demo-video-incomplete.vtt
├── fixtures/
│   ├── untagged-document.pdf
│   └── tagged-document.pdf
├── quarterly-service-report.html
├── index.html
└── README.md
```

## Rules covered

| Rule | Description | Detection |
|---|---|---|
| ACT-R6 | `lang` attribute values must be valid and mutually consistent | automatic |
| ACT-R20 | Invalid `aria-*` attributes must not be used | automatic |
| ACT-R21 | Only valid WAI-ARIA role values may be used | automatic |
| ACT-R33 | Media alternatives must accurately/completely represent content | manual/behavioral |
| ACT-R36 | ARIA attributes forbidden on a given role must not be used | automatic |
| ACT-R71 | No loss of content/functionality under WCAG 1.4.12 text-spacing overrides | manual/behavioral |
| ACT-R79 | `<pre>` must not be used for data that belongs in a semantic table | manual/behavioral |
| ACT-R80 | Fixed-pixel line-height must not prevent scaling/cause overlap | automatic |
| ACT-R83 | Fixed-height + `overflow:hidden` must not clip content on zoom/font-size increase | manual/behavioral |
| ACT-R84 | Scrollable content must be keyboard reachable | automatic |
| ACT-R86 | Purely decorative content must be `aria-hidden="true"` | manual/behavioral |
| ACT-R96 | Pages must not auto-refresh without a user control | automatic |
| ACT-R100 | Linked/embedded PDFs must be tagged or have an HTML equivalent | manual/behavioral |
| ACT-R116 | Every `<details>` needs a `<summary>` with a meaningful accessible name | automatic |
| ACT-R119 | Fixed/sticky elements must not fully obscure a focused control | manual/behavioral |

**Automatic** rules should be caught by a standard automated scanner (e.g.
axe-core, Pa11y, Lighthouse accessibility audit). **Manual/behavioral**
rules require a human to compare rendered/audio/visual output against the
markup — no automated scanner claim is made for these six.

## Running the automated checks

Example using axe-core's CLI against a local static server:

```bash
npx http-server ./a11y-rule-fixtures -p 8080
npx @axe-core/cli http://localhost:8080/accessibility-rules-fail.html
npx @axe-core/cli http://localhost:8080/accessibility-rules-pass.html
```

Expect the FAIL page to report violations for R6, R20, R21, R36, R80, R84,
R96, R116. Expect the PASS page to report zero violations for those same
eight rules. (Coverage of R6 and R36 depends on your scanner's specific
rule set — see the per-rule notes below.)

## What changed since the previous package

- **R84 was restructured on both pages** to fix a real cross-browser bug:
  Chromium and Firefox automatically include any scrollable element with
  genuine overflowing content in the sequential Tab order once it has
  overflow-y set, *even with zero tabindex anywhere in the DOM* — this is
  a deliberate browser mitigation for exactly this class of accessibility
  bug, and it was silently making the old single-`<div>` FAIL fixture
  reachable by Tab (and scrollable by arrow keys) in real browsers,
  defeating the point of the fixture. The fix: split the region into an
  **outer wrapper** (purely visual, never scrolls) and an **inner
  content element** (owns `overflow-y: auto` and the real overflow).
  - **FAIL page**: inner element carries `tabindex="-1"`, which is the
    documented way to opt an element back out of the browser's automatic
    Tab-inclusion. Neither the outer wrapper nor the inner element is
    ever in the Tab order, so a keyboard-only user cannot reach or
    scroll the region at all.
  - **PASS page**: outer wrapper carries `tabindex="0"`,
    `role="region"`, and `aria-label="Scrollable content"`, plus a
    `:focus-visible` outline. A `keydown` listener on the outer wrapper
    handles ArrowUp/ArrowDown/PageUp/PageDown/Home/End and scrolls the
    inner element's `scrollTop` programmatically, calling
    `preventDefault()` only for those specific keys.
- Reverted `accessibility-rules-fail.html` / `accessibility-rules-pass.html`
  back to the clean, minimal fixture markup — no injected debugging script
  in these two files. They're meant to be exactly what your scanner sees,
  nothing more.
- R84's sections now include one plain link each to a new standalone demo
  (see below) instead of any inline script.
- Added three new rules: **R6**, **R36**, **R83** — one FAIL section and
  one PASS section each, added inline to the existing two pages, following
  the same heading/comment pattern as every other rule. No existing rule
  sections were altered besides the R84 link addition and the wrapper-split
  restructuring described above.
- Added two standalone interactive demo pages (see below).

## Interactive demo pages

`sticky-header-focus-demo.html` (R119) and `scrollable-region-focus-demo.html`
(R84) are separate from the core fixture pages on purpose, so they never
affect what a scanner sees when it scans `accessibility-rules-fail.html` /
`accessibility-rules-pass.html`. Each demo has:

- A **toggle checkbox** that switches live, without reloading, between the
  broken behavior and the corrected (WCAG-compliant) behavior.
- A **Focus Inspector** panel (bottom-right corner) that reports in real
  time which element currently has keyboard focus, and whether it was
  successfully reached / is visibly obscured.

**How to test either demo:** click elsewhere on the page first (not into
the element under test), then press `Tab` repeatedly and watch the panel.
Clicking directly into a scrollable region first can let some browsers
scroll it with arrow keys regardless of `tabindex` — that's a separate,
mouse-initiated interaction and isn't what R84 measures. Always start the
Tab sequence without clicking into the test element itself.

## Manual review checklist (per rule)

- **R6** — Confirm the FAIL page's second `lang` value
  (`fr-QQ-typo`) is not a real, valid BCP-47 region subtag, and that the
  first (`engrish`) isn't a valid language subtag at all. Confirm the PASS
  page uses only real, valid codes (`en`, `fr`) that match the actual
  language of their text.
- **R36** — Confirm the FAIL page's `aria-selected` sits on `role="link"`,
  a role that doesn't support a selected state. Confirm the PASS page's
  `aria-current="page"` is a state that's actually valid on `role="link"`.
- **R33** — Play the video with captions on for both pages. On the FAIL
  page, confirm the captions omit the service-hours and phone-number
  sentences that are still audible. On the PASS page, confirm captions and
  the visible transcript both contain all three narration sentences.
- **R71** — Click "Toggle WCAG 1.4.12 text-spacing override" on the FAIL
  page's R71 section and confirm text is clipped/cut off. On the PASS
  page, confirm the equivalent paragraph wraps and the container grows
  with no content lost, override toggled or not.
- **R79** — Confirm the FAIL page's `<pre>` block holds genuinely tabular
  business data (not code, not an ASCII figure). Confirm the PASS page's
  `<pre><code>` holds only source code, and the same data appears in a
  real `<table>` with `<th scope>` headers.
- **R83** — Click "Simulate increased text size / zoom" on the FAIL page's
  R83 section and confirm the enlarged text is clipped by the fixed-height
  box. Confirm the PASS page's equivalent box grows to fit the enlarged
  text with nothing clipped.
- **R86** — Confirm the star icon on the FAIL page is exposed to a screen
  reader (e.g. announced as "star" or similar) even though it's purely
  decorative. Confirm the PASS page's icon is skipped by the screen reader
  because of `aria-hidden="true"`.
- **R100** — Confirm `fixtures/untagged-document.pdf` has no reading order
  / tag structure and no nearby HTML alternative on the FAIL page. Confirm
  `fixtures/tagged-document.pdf` is properly tagged and that
  `quarterly-service-report.html` is linked as its equivalent on the PASS
  page.
- **R119** — On each page, reload and press `Tab` once from the top at
  desktop width and 100% zoom. On the FAIL page the focus outline is
  completely hidden under the fixed header. On the PASS page the focused
  link remains fully visible below the header.

## Video narration

Both VTT files and the transcript on the PASS page use this exact
narration:

> Welcome to our service portal.
> The service desk is open Monday through Friday, 9 AM to 5 PM.
> For urgent help, call 555-0100.

- `media/demo-video-correct.vtt` includes all three sentences, timed to
  match `demo-video.mp4` (0:00–0:04, 0:04–0:08.2, 0:08.2–0:11.8).
- `media/demo-video-incomplete.vtt` includes only the first sentence.

### How `media/demo-video.mp4` was made

It's a short (~11.8s) generated clip: a solid-color background with the
label "Service Portal" burned in as a visual cue, and an audio track of
the exact narration above synthesized with `espeak-ng` and muxed in with
`ffmpeg`. The audio is synthetic text-to-speech, not a human recording —
it exists so the file has a genuine audio track whose spoken content can
be checked against the caption files for R33. If you want a more
realistic fixture for a live demo, swap in a real recording of the same
narration and keep the VTT timings roughly the same; the caption-omission
behavior being tested doesn't depend on who's speaking.

## PDF fixtures

Both PDFs are included and were verified programmatically (via `pikepdf`,
checking `/MarkInfo`, `/StructTreeRoot`, and `/Lang` on the document
catalog):

- **`fixtures/untagged-document.pdf`** — built by drawing text directly
  onto a PDF canvas (Python `reportlab`, `canvas.drawString`), which
  produces no structure tree at all. Verified: no `/MarkInfo`, no
  `/StructTreeRoot`.
- **`fixtures/tagged-document.pdf`** — built from a real structured source
  document (Word-style headings + a genuine table, via `python-docx`) and
  exported to PDF with LibreOffice Writer's headless PDF export, which
  writes a tagged PDF for structured content by default. Verified:
  `/MarkInfo` → `/Marked true`, `/StructTreeRoot` present,
  `/Lang` → `en-US`.

### Re-verifying the PDF tags yourself

- **Adobe Acrobat Pro**: Open the PDF → Tools → Accessibility →
  Accessibility Check. Review the report for "Tagged PDF", reading order,
  and table structure results.
- **PAC (PDF Accessibility Checker)**: A free Windows tool from the
  Access for All Foundation; open the PDF and review its tag tree and
  conformance report.
- **Scriptable check** (what was used here):
  ```python
  import pikepdf
  pdf = pikepdf.open("fixtures/tagged-document.pdf")
  root = pdf.Root
  print("/MarkInfo" in root, "/StructTreeRoot" in root, root.get("/Lang"))
  ```
  `untagged-document.pdf` should report `False False None`;
  `tagged-document.pdf` should report `True True en-US`.

## Notes

- All fixtures are self-contained: no CDN, framework, or external network
  dependency is used anywhere in the HTML/CSS/JS.
- Asset paths are local and relative throughout (`media/…`, `fixtures/…`).
