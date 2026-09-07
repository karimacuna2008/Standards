# Responsive Audit

Applies to any project following the prompt-driven-development pattern that has (or plans to
have) more than a desktop-only surface — real users will open it from a phone, a small laptop,
or a very large/high-resolution display, and nobody has verified the CSS holds up outside
"whatever screen it was built on."

## When to run one

Once a module is built and validated (its functional checklist passes), and before/whenever
real usage on non-standard screen sizes matters. Not tied to a batch — a module can get its
checklist confirmed today and its responsive audit weeks later, or the other way around if the
module is audited before it's fully signed off functionally.

## File

`docs/Estatus Actual/<Modulo>/_<Modulo>-RESPONSIVE.md` — sibling of
`_<Modulo>-CHECKLIST.md`. One per module (or per transversal piece, e.g. a shared shell/layout
that isn't a "module" with its own screen).

Findings are centralized here, never duplicated into each component's own `.md` — organize by
width tested → component/screen affected. A component's own `.md` may link back here, never
repeat the finding.

## What widths to test

Two ends matter, and the number in between is whatever the project's own desktop baseline is:

- **Narrow end (phone):** only relevant if the project's real users will actually open it from a
  phone browser — confirm this with the project owner before auditing, don't assume. When it
  applies, test at least three widths spanning compact to large phones (e.g. 360 / 390 / 430 px)
  — a layout that survives the narrowest and the widest phone width usually survives everything
  in between, but single-point testing misses real breaks.
- **Wide/high-density end:** desktop screens, especially the highest-resolution ones the team
  actually uses. **Do not test at the display's reported physical/panel resolution** — on macOS
  Retina displays (2x scaling), the CSS viewport is half the panel resolution reported in System
  Settings (e.g. a panel reported as "2560×1664" renders a 1280×832 CSS viewport by default). On
  Windows, the CSS viewport depends on the display scale percentage set for that monitor (100%
  scale on a standard external monitor ≈ physical width in CSS px; laptops with dense panels are
  frequently scaled 125-150%, which shrinks the CSS viewport proportionally). Confirm the actual
  CSS viewport for the team's real hardware before picking a number — don't test at a resolution
  nobody's browser ever renders at.
- Whatever the project's normal desktop development width already is (often 1920px) as the
  middle baseline.

## Testing method

Browser window/viewport resized against the real deployed site (not local dev, not a mocked
build) — a reviewer driving Chrome (or equivalent) is enough to catch layout breaks (CSS media
queries, flex/grid behave the same regardless of touch input). Confirming on a real physical
device is optional per project — reserve it for judgment calls that depend on physical rendering
a screenshot can't fully capture (e.g., "does this font size feel legible" on a specific real
display), not for catching layout breaks.

## What each finding records

- Width tested.
- Screen/component affected.
- What breaks (with a screenshot as evidence).
- Severity: bloqueante (unusable) / molesto (usable but degraded) / cosmético.
- Whether the fix is a plain CSS change or needs its own design decision first (e.g. "this fixed
  sidebar needs to become a mobile nav pattern" is not a CSS tweak — flag it and stop, don't
  design the fix inside the audit).

## What this is not

Not a place to design or implement fixes — an audit only documents. Not a place to decide new
values for a design-token scale (font sizes, spacing) even if the audit surfaces evidence that
the current scale is too small somewhere — that's its own decision, documented as a finding here
and picked up as its own piece of work.
