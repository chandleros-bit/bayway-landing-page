# Bayway Landing Page — Motion, Fork Revival & Lead Capture
## Design Spec
### Date: 2026-07-02 | For: Chandler Atkinson (Bayway Mortgage Group, Houston Team)

---

## 01. Background

The handoff bundle (`Mortgage loan officer landing page-handoff.zip`) contains two things that disagree with each other:

- **`Bayway_Houston_LandingPage_DesignSpec.md`** — the original design brief. Its signature idea is "The Fork": directly under the hero, the ribbon graphic (from the logo's waterway motif) splits into two paths — "I'm buying a home" and "I'm a Realtor partner" — which then filters the Programs and Value Tool sections to that reader's version, without navigating to a separate URL.
- **`Bayway Landing.dc.html`** — the actual built page. It never implements the fork. It's a static, linear page shown identically to every visitor, with almost no motion (a FAQ chevron rotation and a scroll-triggered sticky call/apply bar are the only dynamic touches). The pre-approval checklist is on-page content with no email capture. There are no interactive tools.

This spec closes that gap and adds the motion, tools, and lead-magnet elements requested, based on decisions made during brainstorming (see §08).

---

## 02. Goals

1. Revive the fork as the page's one differentiated interaction, doing real work (audience-appropriate Programs and Value Tool content) rather than being decorative.
2. Add scroll-triggered motion that reinforces the page's existing "data as trust signal" idea (Plex Mono stat callouts) instead of just decorating it.
3. Add one ungated tool (payment calculator) and one gated lead magnet (Realtor co-marketing kit) that each earn their section rather than padding it.
4. Add one new high-value content block (application timeline) that speaks directly to the anxious first-time-buyer persona named in the original brief.
5. Do all of this without breaking the original brand's "restraint" rule — quiet except for the fork.

**Out of scope for this pass:** the rate-range quiz, the DSCR/self-employed qualifier tool, and any changes to Why-a-Broker or Testimonials content (they read fine for both audiences already and the original layout table never marked them as fork-filtered).

---

## 03. The Fork (revived)

**Mechanism:** unchanged from the original spec — the ribbon SVG under the hero visually forks into two labeled paths. Clicking one sets an audience state (`buyer` | `realtor`) that persists for the session via a small "Switch view" pill that stays reachable (e.g., pinned near the nav) after the initial choice.

**What actually filters** (narrower than the original spec's suggestion, matching what its own layout table specified):
- **Programs section** — Buyer view leads with Conventional / FHA / VA / First-Time Buyer. Realtor view leads with Non-QM / DSCR / Bank-Statement and foregrounds the 10-day-close detail.
- **Value Tool section** — Buyer view shows the existing pre-approval checklist + credit-score-by-program table + the new payment calculator (§05). Realtor view shows the new gated co-marketing kit (§06).

**What stays universal:** Why-a-Broker's three proof points and the Testimonials section. Both already read fine for either audience, and narrowing the build surface here keeps the fork's implementation cost proportional to its actual payoff.

**Transition:** filtered sections cross-fade/slide when the fork state changes, rather than snapping instantly — ties into the motion system in §04.

**Mobile:** the ribbon-fork animation collapses to a simple two-tab toggle pinned directly under the hero (no scroll-jacking), consistent with the original spec.

---

## 04. Motion System — "Scrollytelling," restrained

Selected over a fully restrained option (which is the spec as literally written) and a bold/ambient option (continuous floating/pulsing motion, rejected as too "salesy" for a trust-driven finance page).

- **Count-up numbers**: stat callouts (10 days, NMLS #, states licensed, lenders shopped) animate from 0 to their value once, the first time they scroll into view. Numeric fields only — text fields (e.g., "TX · FL") just fade/rise as before.
- **Staggered card entry**: Program cards, stat cards, and testimonial cards pop in with a slight delay between each (not all at once), on first scroll into view only — no re-triggering on repeated scroll past the same section.
- **Ribbon redraw**: the brand ribbon SVG (hero fork, and the Why-a-Broker section divider if one is added) animates as a self-drawing stroke the first time it enters view, rather than rendering fully static.
- **Fork transition**: cross-fade/slide (≈300–400ms) when switching between Buyer/Realtor filtered content, so the swap reads as a deliberate transition, not a flicker.
- **Reduced motion**: `prefers-reduced-motion: reduce` disables all of the above — count-ups render at final value, staggers render simultaneously, ribbon renders in its fully-drawn state, fork transitions become instant. This is a hard requirement carried over from the original spec, not a nice-to-have.
- **Everything not listed here stays exactly as restrained as the original spec** — hover-only micro-interactions on interactive elements, no parallax, no scroll-jacking, no ambient/looping animation anywhere on the page.

---

## 05. Monthly Payment Calculator (ungated)

**Placement:** inside the Value Tool section, Buyer fork view, alongside the existing checklist and credit-score table.

**Inputs:** home price, down payment (% or $), loan term, and a user-adjustable rate assumption (slider or input, not a fixed/quoted number).

**Output:** estimated monthly principal & interest, styled in the page's existing Plex Mono "ledger" treatment to match the Meet Chandler stat row.

**Compliance constraint (important, not optional):** the rate input must be user-editable and clearly framed as an assumption the visitor is modeling — e.g., "See what a scenario could look like" — not a rate Chandler is quoting or advertising as available today. Advertising a specific rate as fact triggers Regulation Z "trigger term" disclosure requirements (APR, finance charges, etc.). Framed as a modeling tool, it stays a harmless estimate and directly sets up the existing FAQ answer that already says "ask me to model two scenarios before you commit — it's free."

**Disclaimer copy (required, adjacent to the tool):** "Estimate only, for illustration. Actual rate and payment depend on credit, program, and lock date — not a quote or commitment to lend."

---

## 06. Realtor Co-Marketing Kit (gated)

**Placement:** replaces/extends the Realtor fork view of the Value Tool section. The existing Realtor Partner Band section stays exactly as-is and ungated — it's the pitch. This is the deeper asset behind it.

**Flow:** a second CTA option next to "Partner with me →" — "Get the co-marketing kit" — opens a lightweight inline email-capture form (name + email + brokerage, optional). On submit:
- Unlocks/sends co-branded flyer template links.
- Enrolls the submitter in the Monday Market Brief list.
- Surfaces Bayway's actual Realtor Partner Program terms (not invented ones — see open items).

**Why gated, not the checklist too:** matches the "mix" gating strategy chosen during brainstorming — quick, low-effort tools (the calculator, the existing checklist) stay ungated to build trust fast; the deeper, more valuable asset (real templates + real program terms + a list enrollment) is what's worth an email exchange.

---

## 07. "What Happens After You Apply" Timeline

**Placement:** near the Value Tool section, universal (not fork-filtered — both audiences benefit from process transparency, Realtors especially since it's what they're promising their clients).

**Content:** a short step sequence — Application → Documents → Underwriting → Clear-to-close → Closing day — with realistic day-count ranges per step.

**Required disclaimer:** reuse the existing FAQ pattern — "Timelines vary with loan type and how quickly documents come in" — so the sequence reads as a realistic range, not a guarantee.

---

## 08. Decisions Made During Brainstorming

| Decision | Choice | Rationale |
|---|---|---|
| Page structure | Revive the fork | The one differentiated idea in the brief; gives motion a real job; lets tools/magnets be audience-targeted instead of generic |
| Motion intensity | Scrollytelling (B) | Adds energy without breaking the "quiet except the fork" rule; makes the "data as trust signal" idea actually felt |
| Lead-magnet gating | Mix | Quick tools ungated (build trust fast); deeper assets gated (actually grows the list) |
| Tools/magnets to build | Payment calculator (ungated) + Realtor co-marketing kit (gated) | Rate-quiz and DSCR-qualifier tools deferred as future scope, not needed for this pass |
| Extra content | "What happens after you apply" timeline | Directly answers the anxious first-time-buyer's biggest unspoken question |

---

## 09. Open Items — Confirm Before Build

(Mirroring the original design spec's own pattern — these block implementation, not this design.)

- [ ] **Form-handling backend.** The handoff bundle is a static Claude-Design prototype (`.dc.html`) with no server. A real target is needed before the gate can capture anything — options include Formspree, Netlify Forms, a direct webhook into Bayway's CRM, or a plain `mailto:` fallback. Whoever implements this needs a decision here first.
- [ ] **Realtor co-marketing kit assets.** Actual flyer templates and Bayway's actual current Partner Program terms — not invented placeholders. (Same open item as the original spec's §08.)
- [ ] **Application timeline day-counts.** Needs to be confirmed against Chandler's actual typical process, not assumed.
- [ ] **Target implementation stack.** This spec describes behavior, not framework. Whatever codebase ultimately builds this (plain HTML/CSS/JS, React, etc.) should follow its own conventions — the spec deliberately doesn't prescribe one.

---

## 10. Explicitly Out of Scope This Pass

- Rate-range quiz with gated PDF result
- DSCR / self-employed qualifier calculator
- Any change to Why-a-Broker or Testimonials content/filtering
- Any change to the footer/compliance block (already correct per the original spec's §07)
