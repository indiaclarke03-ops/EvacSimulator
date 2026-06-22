# Extension prompt: Weighting profiles (perspective/context layer)

## Context

This extends the Destination Readiness Factor Web tool (see the original
build prompt, `destination-readiness-tool-prompt.md`, and the accompanying
methodology document, `destination-readiness-methodology.docx`).

The current v1 tool uses a single fixed weighting scheme: gatekeeper factors
(Security, Local authorities consent) are weighted 2x standard factors, and
the score is capped at 20% if either gatekeeper is Blocked.

This extension reframes that single scheme as **one profile among several**,
representing the idea that different actors, contexts, or evacuation types
may reasonably weight the same seven factors differently. The same factor
statuses run through different profiles should produce different readiness
scores — and the *difference* between scores is itself meaningful output.

This is a research framing, not just a UI feature: the tool becomes an
instrument for showing how the same underlying situation can look more or
less "ready" depending on whose priorities are encoded into the assessment.

---

## Part 1: Add weighting profiles to the tool

### Data structure

Define a set of named weighting profiles. Each profile specifies:
- A name and short description
- A weight for each of the 7 factors (not just gatekeeper vs standard —
  allow per-factor weights so profiles can genuinely differ in shape)
- Whether the "gatekeeper blocked → cap score" rule applies, and if so,
  which factors act as gatekeepers and what the cap value is

Suggested starter profiles (label these clearly as illustrative starting
points, not finalised research outputs):

**Profile A — "Security-led" (current default)**
- Security: weight 2, gatekeeper, cap 20% if blocked
- Local authorities consent: weight 2, gatekeeper, cap 20% if blocked
- Willingness, Capacity, Shelter, Food & water, Medical capacity: weight 1 each

**Profile B — "Humanitarian access"**
- Local authorities consent: weight 2, gatekeeper, cap 20% if blocked
- Security: weight 1.5 (still important but not sole gatekeeper)
- Shelter, Food & water, Medical capacity: weight 1.5 each (resource
  factors weighted more heavily than in Profile A)
- Willingness: weight 1
- Capacity: weight 1
- No additional cap beyond the consent gatekeeper

**Profile C — "Community-led"**
- Willingness: weight 2 (community acceptance is the primary gate)
- Shelter: weight 1.5
- Security: weight 1.5
- Local authorities consent: weight 1
- Capacity, Food & water, Medical capacity: weight 1 each
- No gatekeeper cap — reflects a perspective where no single factor
  automatically overrides all others

Implement these as data only (e.g. a `WEIGHTING_PROFILES` object or array),
so additional profiles can be added later without changing the scoring
logic.

### Scoring logic changes

- Generalise the score calculation to use per-factor weights from the
  active profile, rather than a hardcoded gatekeeper/standard split.
- Generalise the cap logic so it is driven by the active profile's
  gatekeeper list and cap value (profiles without a cap should simply not
  apply one).
- The formula itself (weighted average × 100, then cap if applicable)
  stays the same — only the weights, gatekeeper list, and cap value
  become profile-dependent.

### UI changes

- Add a profile selector (e.g. a dropdown or segmented control) above the
  readiness score, showing the active profile name and a one-line
  description.
- When the user switches profiles, recalculate and animate the readiness
  score and bar using the same factor statuses.
- Add a small "compare profiles" view: given the current factor statuses,
  show the readiness score under each profile side by side (e.g. a simple
  table or set of small bars), so the user can see at a glance how much
  the score varies depending on perspective.
- Keep this lightweight — this is not meant to become a complex dashboard.
  A compact table (Profile name | Score | Capped?) is sufficient.

### What NOT to change

- The seven factors, their IDs, and the dependency relationships (arrows
  in the network diagram) stay the same across all profiles. Only the
  *weights and cap rules* vary by profile — the structure of the factor
  web itself is profile-independent in this version.
- Keep the existing status values (Operational / Partial / Blocked /
  Unknown) and their underlying scores (1.0 / 0.5 / 0.0 / excluded)
  unchanged.

---

## Part 2: Update the methodology document

Add a new section to `destination-readiness-methodology.docx` (insert
after the existing "Section 6: Readiness score" and before "Section 7:
Visual encoding"). Renumber subsequent sections accordingly.

### New section: "Weighting as perspective: introducing profiles"

This section should:

1. **Reframe the original weighting** (Section 6.2 in the current
   document) as one profile — "Profile A: Security-led" — rather than
   the definitive scheme. Make explicit that this was the v1 default but
   is not the only reasonable way to weight these factors.

2. **Introduce the concept of weighting profiles**, explaining that
   different actors (e.g. military planners, humanitarian organisations,
   host communities, evacuees themselves), different contexts (e.g.
   countries with strong vs weak rule of law), or different evacuation
   types (e.g. mass civilian displacement vs targeted evacuation of a
   vulnerable group) may reasonably prioritise these seven factors
   differently.

3. **Present Profiles A, B, and C** (as defined in Part 1 above) using a
   table similar in style to the existing weighting table in Section 6.2
   — one table per profile, or one combined table with profiles as
   columns and factors as rows, whichever renders more clearly.

4. **Explicitly flag this as an open research question**, not a finished
   design. Include language along these lines (adapt tone to match the
   rest of the document):

   > "These profiles are illustrative starting points rather than
   > validated weighting schemes. A key open question is which axis of
   > variation is most relevant to this research: differences between
   > assessing actors, differences between geographic or political
   > contexts, or differences between types of evacuation. Each of these
   > would suggest a different research design — for example, eliciting
   > weights directly from practitioners representing different actor
   > types, or testing whether a single profile produces sensible results
   > across multiple real-world case studies."

5. **Add a worked comparison** showing one of the existing worked example's
   factor statuses (from the original Section 6.3) run through Profiles
   A, B, and C, with the resulting scores shown side by side. Briefly
   interpret the difference — e.g. note if the same underlying situation
   looks "ready" under one profile and "not ready" under another, and why.

6. **Add a short limitations note** acknowledging that:
   - The profiles are currently hypothetical and not grounded in cited
     literature or elicited from practitioners.
   - Per-factor weights within each profile are illustrative and have not
     been validated.
   - The dependency structure (Section 5) is held constant across
     profiles in this version; a more advanced version might allow the
     relationships themselves to vary by context, but this is out of
     scope for now.

### Style and formatting

- Match the existing document's style: same heading levels, same colour
  palette and table formatting conventions used elsewhere in the
  document (header rows with dark blue fill and white text, alternating
  row shading, colour-coded status cells where relevant).
- Keep the tone consistent with the rest of the document — direct,
  acknowledges limitations openly, frames open questions as invitations
  for discussion rather than gaps to be hidden.

---

## Deliverables

1. Updated tool (single `index.html` or equivalent) with the weighting
   profile selector, generalised scoring logic, and profile comparison
   view, built on top of the existing v1 tool.
2. Updated methodology document with the new "Weighting as perspective:
   introducing profiles" section inserted in the correct place, existing
   sections renumbered as needed.
