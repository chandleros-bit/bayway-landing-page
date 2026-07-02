# Bayway Landing Page Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the Bayway Mortgage Group landing page for Chandler Atkinson as a single static HTML/CSS/JS site — reviving the buyer/Realtor "fork" interaction, adding scrollytelling motion, a payment calculator, a gated Realtor co-marketing kit, and an application timeline — deployable to Netlify with zero build step.

**Architecture:** One `index.html` with all page content, one `styles.css` for design tokens/layout/motion, and small ES modules (`calculator.js`, `fork-state.js`) with pure, unit-tested logic imported by a thin DOM-wiring `script.js`. Netlify Forms handles the gated Realtor kit's email capture with no custom backend.

**Tech Stack:** Plain HTML5, CSS3 (custom properties), vanilla JS (ES modules, `IntersectionObserver`), Node's built-in test runner (`node --test`) for the pure-logic modules, Netlify for hosting + form handling.

Reference documents:
- Design spec: `docs/superpowers/specs/2026-07-02-bayway-landing-page-improvements-design.md`
- Original brief: extracted from `Mortgage loan officer landing page-handoff.zip` (`Bayway_Houston_LandingPage_DesignSpec.md` and `Bayway Landing.dc.html`) — source of the real copy (programs, testimonials, FAQs, docs checklist, credit ranges, footer compliance text) carried into this build.

---

## Testing approach

- **Pure logic** (payment math, audience-preference storage) gets real unit tests via `node --test` — no DOM, no browser, no mocking framework needed.
- **Markup, layout, motion, and gated-form wiring** are verified by manually opening the page in a browser and checking behavior — there's no meaningful way to unit-test CSS animations or `IntersectionObserver` timing without adding a browser-automation dependency, which would contradict the "no build step, no framework" goal for a one-page site. Each such task has an explicit "Verify in browser" step describing exactly what to check.
- Netlify Forms only actually submits once deployed (it works by scanning the built HTML at deploy time) — form submission itself is verified after Task 12's deploy, not locally.

---

### Task 1: Project scaffold, tooling, and real assets

**Files:**
- Create: `package.json`
- Create: `netlify.toml`
- Create: `.gitignore` (append to existing)
- Create: `assets/bayway-logo.png`
- Create: `assets/chandler.jpg`

- [ ] **Step 1: Create `package.json`**

```json
{
  "name": "bayway-landing-page",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "test": "node --test tests/"
  }
}
```

- [ ] **Step 2: Create `netlify.toml`**

```toml
[build]
  publish = "."

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
```

- [ ] **Step 3: Copy the real logo and headshot from the handoff bundle**

The handoff zip (`C:\Users\Chandler\Downloads\Mortgage loan officer landing page-handoff.zip`) contains the real brand assets. Extract just the two image files needed:

Run:
```bash
mkdir -p assets
cd /tmp && unzip -o "C:/Users/Chandler/Downloads/Mortgage loan officer landing page-handoff.zip" -d bayway-extract
cp "bayway-extract/mortgage-loan-officer-landing-page/project/assets/bayway-logo.png" "<repo-root>/assets/bayway-logo.png"
cp "bayway-extract/mortgage-loan-officer-landing-page/project/assets/chandler.jpg" "<repo-root>/assets/chandler.jpg"
```
(Replace `<repo-root>` with the actual repo path.)

Expected: `assets/bayway-logo.png` and `assets/chandler.jpg` exist and open as valid images.

- [ ] **Step 4: Update `.gitignore`**

Append to the existing `.gitignore`:
```
node_modules/
```

- [ ] **Step 5: Commit**

```bash
git add package.json netlify.toml .gitignore assets/bayway-logo.png assets/chandler.jpg
git commit -m "chore: scaffold static site project and add real brand assets"
```

---

### Task 2: Design tokens and base CSS

**Files:**
- Create: `styles.css`

- [ ] **Step 1: Write the design-token and reset foundation**

```css
:root {
  --forest: #0B5E42;
  --meadow: #7CAD44;
  --wheat: #E0A730;
  --cream: #F6F7F2;
  --ink: #17231D;
  --mist: #DCE5DA;

  --font-display: 'Fraunces', serif;
  --font-body: 'Public Sans', system-ui, sans-serif;
  --font-mono: 'IBM Plex Mono', monospace;
}

* { box-sizing: border-box; }

body {
  margin: 0;
  font-family: var(--font-body);
  color: var(--ink);
  background: #ffffff;
  line-height: 1.6;
  -webkit-font-smoothing: antialiased;
}

h1, h2, h3 { margin: 0; }

a { color: inherit; }

.section {
  padding: clamp(56px, 7vw, 92px) clamp(20px, 4vw, 44px);
}

.section-inner {
  max-width: 1120px;
  margin: 0 auto;
}

.eyebrow {
  font-family: var(--font-mono);
  font-size: 12px;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  margin-bottom: 14px;
}

.btn {
  display: inline-block;
  text-decoration: none;
  font-weight: 700;
  font-size: 16px;
  padding: 14px 28px;
  border-radius: 999px;
}

.btn-primary {
  background: var(--wheat);
  color: var(--ink);
  box-shadow: 0 10px 28px -14px rgba(224, 167, 48, 0.9);
}

.btn-secondary {
  border: 1.5px solid var(--forest);
  color: var(--forest);
  background: transparent;
}
```

- [ ] **Step 2: Verify in browser**

Create a temporary blank `index.html` with `<link rel="stylesheet" href="styles.css">` and a `<h1>Test</h1>` and a `<a class="btn btn-primary">Test</a>`. Open it in a browser. Confirm the heading uses no custom font yet (fonts come in Task 3), the button is wheat-colored with rounded pill shape, and there are no console errors. Delete the temporary file after checking.

- [ ] **Step 3: Commit**

```bash
git add styles.css
git commit -m "feat: add design tokens and base CSS reset"
```

---

### Task 3: Nav, hero, and static ribbon markup

**Files:**
- Create: `index.html`
- Modify: `styles.css` (append hero/nav rules)

- [ ] **Step 1: Write `index.html` head and nav**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Chandler Atkinson | Houston Mortgage Broker &amp; Loan Officer — Bayway Mortgage Group</title>
<meta name="description" content="Chandler Atkinson is a Houston mortgage broker and loan officer with Bayway Mortgage Group. Conventional, FHA, VA, first-time buyer, self-employed (Non-QM), and investor (DSCR) loans shopped across many wholesale lenders. Licensed in TX & FL. Free consultation — call 832-474-7707.">
<link rel="canonical" href="https://www.baywayhtx.com/chandler-atkinson">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,400;9..144,500;9..144,600&family=Public+Sans:wght@400;500;600;700&family=IBM+Plex+Mono:wght@400;500;600&display=swap" rel="stylesheet">
<link rel="stylesheet" href="styles.css">
</head>
<body>

<nav class="nav">
  <a href="#top" class="nav-logo" aria-label="Bayway Mortgage Group home">
    <img src="assets/bayway-logo.png" alt="Bayway Mortgage Group logo" height="46">
  </a>
  <div class="nav-links">
    <a href="#programs">Loan Programs</a>
    <a href="#chandler">About</a>
    <a href="#faq">FAQ</a>
    <a href="tel:8324747707" class="nav-phone">832-474-7707</a>
    <a href="#apply" class="btn btn-primary" style="padding:10px 20px;font-size:15px;">Apply&nbsp;→</a>
  </div>
</nav>

<div id="top"></div>
```

- [ ] **Step 2: Write the hero section with the (currently inert) fork buttons**

```html
<header class="hero">
  <div class="hero-inner">
    <div class="eyebrow" style="color:var(--forest);opacity:0.85;">Chandler Atkinson · Houston Mortgage Broker · NMLS #1851744</div>
    <h1 class="hero-h1">A Houston mortgage broker who shops the whole market for you.</h1>
    <p class="hero-sub">I'm Chandler Atkinson with Bayway Mortgage Group. As a <em>broker</em> — not a bank — I compare loans across many wholesale lenders to find your best rate and program, whether you're a first-time buyer, self-employed, an investor, or a Realtor's client on a deadline.</p>

    <div class="fork-buttons" data-fork-buttons>
      <button type="button" class="btn btn-primary" data-audience-choice="buyer">I'm buying a home</button>
      <button type="button" class="btn btn-secondary" data-audience-choice="realtor">I'm a Realtor partner</button>
    </div>

    <div class="hero-ctas">
      <a href="#apply" class="btn btn-primary">Apply Now</a>
      <a href="#cta" class="btn btn-secondary">Book a free consultation</a>
      <a href="tel:8324747707" class="hero-phone">☏ 832-474-7707</a>
    </div>

    <div class="hero-tags">
      <span>Conventional</span><span>·</span><span>FHA</span><span>·</span><span>VA</span><span>·</span><span>First-Time Buyer</span><span>·</span><span>Non-QM</span><span>·</span><span>DSCR</span><span>·</span><span>Licensed TX &amp; FL</span>
    </div>
  </div>

  <svg class="ribbon" data-ribbon viewBox="0 0 1000 120" preserveAspectRatio="none" aria-hidden="true">
    <path class="ribbon-path ribbon-path-main" d="M0,72 C 220,42 340,92 520,72 C 700,52 820,86 1000,60" fill="none" stroke="var(--meadow)" stroke-width="13" stroke-linecap="round"></path>
    <path class="ribbon-path ribbon-path-shadow" d="M0,86 C 220,56 340,106 520,86 C 700,66 820,100 1000,74" fill="none" stroke="var(--forest)" stroke-width="9" stroke-linecap="round" opacity="0.32"></path>
  </svg>
</header>
```

- [ ] **Step 3: Append nav/hero CSS to `styles.css`**

```css
.nav {
  position: sticky; top: 0; z-index: 40;
  display: flex; align-items: center; justify-content: space-between; gap: 24px;
  padding: 12px clamp(20px, 4vw, 44px);
  background: rgba(255, 255, 255, 0.92);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid var(--mist);
}
.nav-logo { display: flex; align-items: center; text-decoration: none; }
.nav-links { display: flex; align-items: center; gap: clamp(14px, 2.4vw, 30px); }
.nav-links a { text-decoration: none; color: var(--ink); font-weight: 500; font-size: 15px; }
.nav-phone { color: var(--forest) !important; font-weight: 600; font-family: var(--font-mono); }

.hero {
  position: relative; overflow: hidden;
  background: linear-gradient(180deg, #ffffff 0%, var(--cream) 100%);
  padding: clamp(48px, 7vw, 96px) clamp(20px, 4vw, 44px) 0;
}
.hero-inner { max-width: 960px; margin: 0 auto; text-align: center; }
.hero-h1 {
  font-family: var(--font-display); font-weight: 400;
  font-size: clamp(36px, 5.8vw, 60px); line-height: 1.05; letter-spacing: -0.01em;
  margin: 0 0 22px; color: var(--forest);
}
.hero-sub { max-width: 660px; margin: 0 auto; font-size: clamp(16px, 1.5vw, 18px); opacity: 0.84; }
.fork-buttons { display: flex; flex-wrap: wrap; gap: 14px; justify-content: center; margin-top: 30px; }
.hero-ctas { display: flex; flex-wrap: wrap; gap: 14px; justify-content: center; margin-top: 18px; }
.hero-phone { text-decoration: none; color: var(--forest); font-family: var(--font-mono); font-weight: 500; font-size: 16px; padding: 14px 8px; }
.hero-tags {
  display: flex; flex-wrap: wrap; gap: 10px 20px; justify-content: center; margin-top: 30px;
  font-family: var(--font-mono); font-size: 12.5px; color: var(--forest); opacity: 0.72;
  text-transform: uppercase; letter-spacing: 0.04em;
}
.ribbon { display: block; width: 100%; height: clamp(70px, 9vw, 108px); margin-top: 26px; }
```

- [ ] **Step 4: Verify in browser**

Open `index.html` directly. Confirm: logo displays, nav links scroll to anchors, headline renders in Fraunces serif, both fork buttons are visible and centered, ribbon SVG renders below the hero content.

- [ ] **Step 5: Commit**

```bash
git add index.html styles.css
git commit -m "feat: add nav, hero, and static ribbon markup"
```

---

### Task 4: Why-a-Broker, Programs (baseline), and Meet Chandler sections

**Files:**
- Modify: `index.html` (append after hero)
- Modify: `styles.css` (append section rules)

- [ ] **Step 1: Append the Why-a-Broker section to `index.html`**

```html
<section class="section why-broker">
  <div class="section-inner">
    <div class="why-broker-head">
      <div class="eyebrow" style="color:var(--wheat);">Why a broker, not a bank</div>
      <h2 class="section-h2">A bank sells you the one loan it has. A broker goes and finds yours.</h2>
    </div>
    <div class="why-broker-grid">
      <div class="why-broker-card">
        <div class="why-broker-num">01</div>
        <h3>Wholesale rates, not retail</h3>
        <p>I access the same wholesale lenders banks buy from — and pass the pricing to you instead of marking it up on a branded rate sheet.</p>
      </div>
      <div class="why-broker-card">
        <div class="why-broker-num">02</div>
        <h3>Many lenders shopped per file</h3>
        <p>One application, and I put your file in front of multiple lenders — then bring back the one that actually fits your income, credit and timeline.</p>
      </div>
      <div class="why-broker-card">
        <div class="why-broker-num">03</div>
        <h3>You reach me, not a call center</h3>
        <p>Direct answers from the person underwriting your options — a real phone number, not a ticket queue and a 1-800 line.</p>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Step 2: Append the Programs section (unfiltered baseline — fork filtering comes in Task 8)**

```html
<section id="programs" class="section programs">
  <div class="section-inner">
    <div class="programs-head">
      <div class="eyebrow" style="color:var(--forest);opacity:0.7;">Loan programs</div>
      <h2 class="section-h2">Home loans for every kind of Houston buyer</h2>
      <p class="section-lede">One application; I shop it across many lenders and bring back the fit. Every card lists a real qualifying detail — not a slogan.</p>
    </div>
    <div class="programs-grid" data-programs-grid>
      <div class="program-card" data-audience-tag="buyer">
        <h3>Conventional</h3>
        <p>The workhorse loan for buyers with steady income and solid credit.</p>
        <div class="program-detail">3% down · 620+ score typical</div>
      </div>
      <div class="program-card" data-audience-tag="buyer">
        <h3>FHA</h3>
        <p>First-time and credit-building buyers get in with less down and flexible credit.</p>
        <div class="program-detail">3.5% down at 580 · 500 with 10% down</div>
      </div>
      <div class="program-card" data-audience-tag="buyer">
        <h3>VA</h3>
        <p>Veterans, active-duty service members, and eligible surviving spouses.</p>
        <div class="program-detail">$0 down · no PMI · no VA score minimum</div>
      </div>
      <div class="program-card" data-audience-tag="buyer">
        <h3>First-Time Buyer</h3>
        <p>Down-payment assistance stacked onto your loan to keep cash-to-close low.</p>
        <div class="program-detail">Up to 5% assistance in Texas</div>
      </div>
      <div class="program-card" data-audience-tag="realtor">
        <span class="program-tag">Specialty</span>
        <h3>Non-QM / Bank Statement</h3>
        <p>Self-employed and 1099 buyers who don't fit the bank's W-2 box.</p>
        <div class="program-detail">Qualify on 12–24 mo bank statements</div>
      </div>
      <div class="program-card" data-audience-tag="realtor">
        <span class="program-tag">Specialty</span>
        <h3>DSCR / Investor</h3>
        <p>Rental and investment loans that qualify on the property, not your pay stubs.</p>
        <div class="program-detail">Approves on rent-to-payment ratio (DSCR ≥ 1.0)</div>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Step 3: Append the Meet Chandler section**

```html
<section id="chandler" class="section chandler">
  <div class="section-inner chandler-grid">
    <div class="chandler-photo-wrap">
      <div class="chandler-photo">
        <img src="assets/chandler.jpg" alt="Chandler Atkinson, Houston mortgage loan officer with Bayway Mortgage Group">
      </div>
      <div class="chandler-photo-badge">Houston, TX · Houston market</div>
    </div>
    <div>
      <div class="eyebrow" style="color:var(--forest);opacity:0.7;">Meet Chandler</div>
      <h2 class="section-h2">The loans other lenders punt on are the ones I specialize in.</h2>
      <p class="chandler-bio">Chandler Atkinson is a Houston-based loan officer serving the greater Houston market with Bayway Mortgage Group. As a broker he shops each file across many wholesale lenders — and goes deep on the files banks turn away: self-employed and investor buyers, bank-statement income, and first-time buyers who need a hand with down payment.</p>
      <div class="chandler-stats">
        <div class="stat"><div class="stat-value">#1851744</div><div class="stat-label">Individual NMLS</div></div>
        <div class="stat"><div class="stat-value">TX · FL</div><div class="stat-label">Licensed states</div></div>
        <div class="stat" data-count-target="10" data-count-suffix=" days"><div class="stat-value">10 days</div><div class="stat-label">Closings as fast as</div></div>
        <div class="stat"><div class="stat-value">Multi</div><div class="stat-label">Lenders shopped / file</div></div>
      </div>
    </div>
  </div>
</section>
```

Note: `data-count-target`/`data-count-suffix` on the "10 days" stat is wired up in Task 11 (scrollytelling counters). Leave it inert for now.

- [ ] **Step 4: Append CSS for these three sections**

```css
.section-h2 {
  font-family: var(--font-display); font-weight: 400;
  font-size: clamp(28px, 3.6vw, 38px); line-height: 1.14;
  margin: 0 0 10px; color: var(--forest);
}
.section-lede { margin: 0; font-size: 16.5px; opacity: 0.78; }

.why-broker { background: var(--forest); color: #fff; }
.why-broker .section-h2 { color: #fff; }
.why-broker-head { max-width: 680px; margin-bottom: 44px; }
.why-broker-grid {
  display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 2px;
  background: rgba(255, 255, 255, 0.14); border-radius: 16px; overflow: hidden;
}
.why-broker-card { background: var(--forest); padding: 32px 30px; }
.why-broker-num { font-family: var(--font-mono); font-size: 14px; color: var(--meadow); margin-bottom: 14px; }
.why-broker-card h3 { font-size: 19px; font-weight: 600; margin: 0 0 8px; }
.why-broker-card p { margin: 0; font-size: 15px; opacity: 0.82; line-height: 1.6; }

.programs { background: var(--cream); }
.programs-head { max-width: 700px; margin-bottom: 40px; }
.programs-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 16px; }
.program-card {
  background: #fff; border: 1px solid var(--mist); border-radius: 16px;
  padding: 26px 24px; display: flex; flex-direction: column;
}
.program-card h3 { font-family: var(--font-display); font-weight: 500; font-size: 23px; margin: 0 0 8px; color: var(--forest); }
.program-card p { margin: 0 0 18px; font-size: 15px; opacity: 0.8; line-height: 1.55; flex: 1; }
.program-detail { border-top: 1px solid var(--mist); padding-top: 14px; font-family: var(--font-mono); font-size: 13.5px; color: var(--forest); }
.program-tag {
  align-self: flex-start; font-family: var(--font-mono); font-size: 11px; letter-spacing: 0.06em;
  text-transform: uppercase; color: var(--forest); background: var(--mist);
  padding: 4px 9px; border-radius: 6px; margin-bottom: 14px;
}

.chandler-grid {
  display: grid; grid-template-columns: minmax(0, 0.9fr) minmax(0, 1.1fr);
  gap: clamp(28px, 5vw, 64px); align-items: center;
}
.chandler-photo-wrap { position: relative; }
.chandler-photo { aspect-ratio: 4 / 5; border-radius: 20px; overflow: hidden; background: var(--mist); border: 1px solid var(--mist); }
.chandler-photo img { width: 100%; height: 100%; object-fit: cover; object-position: center top; display: block; }
.chandler-photo-badge {
  position: absolute; left: 18px; bottom: 18px; background: var(--wheat); color: var(--ink);
  padding: 8px 14px; border-radius: 10px; font-family: var(--font-mono); font-size: 12.5px; font-weight: 500;
}
.chandler-bio { margin: 0 0 26px; font-size: 16.5px; opacity: 0.82; line-height: 1.65; }
.chandler-stats {
  display: grid; grid-template-columns: 1fr 1fr; gap: 1px;
  background: var(--mist); border: 1px solid var(--mist); border-radius: 14px; overflow: hidden;
}
.stat { background: #fff; padding: 18px 20px; }
.stat-value { font-family: var(--font-mono); font-size: 20px; color: var(--forest); letter-spacing: -0.01em; }
.stat-label { font-size: 12.5px; opacity: 0.65; margin-top: 3px; }

@media (max-width: 760px) {
  .chandler-grid { grid-template-columns: 1fr; }
}
```

- [ ] **Step 5: Verify in browser**

Confirm: forest-green Why-a-Broker band renders with 3 cards, Programs grid shows all 6 cards (unfiltered — that's expected at this stage), Meet Chandler section shows photo + stat grid.

- [ ] **Step 6: Commit**

```bash
git add index.html styles.css
git commit -m "feat: add why-a-broker, programs baseline, and meet-chandler sections"
```

---

### Task 5: Value Tool baseline, Testimonials, FAQ, CTA, Footer, sticky bar

**Files:**
- Modify: `index.html` (append remaining sections)
- Modify: `styles.css` (append remaining section rules)
- Create: `script.js`

- [ ] **Step 1: Append the Value Tool section (baseline: checklist + credit table only — calculator added in Task 7, Realtor kit added in Task 10)**

```html
<section id="tool" class="section value-tool">
  <div class="section-inner value-tool-inner">
    <div class="value-tool-card" data-audience-view="buyer">
      <div class="eyebrow" style="color:var(--meadow);">Worth bookmarking</div>
      <h2 class="section-h2">What you actually need for pre-approval</h2>
      <p class="section-lede" style="margin-bottom:32px;">Have these ready and we can move fast — even if you're just getting organized.</p>
      <div class="value-tool-grid">
        <div>
          <h3 class="value-tool-subhead">Documents</h3>
          <div class="checklist">
            <div class="checklist-item"><span class="check-icon">✓</span><span>Two most recent pay stubs (last 30 days)</span></div>
            <div class="checklist-item"><span class="check-icon">✓</span><span>W-2s and tax returns — last 2 years</span></div>
            <div class="checklist-item"><span class="check-icon">✓</span><span>Two months of bank statements (all pages)</span></div>
            <div class="checklist-item"><span class="check-icon">✓</span><span>Government photo ID and Social Security number</span></div>
            <div class="checklist-item"><span class="check-icon">✓</span><span>Self-employed? 12–24 months of bank statements</span></div>
          </div>
        </div>
        <div>
          <h3 class="value-tool-subhead">Credit score, by program</h3>
          <div class="credit-table">
            <div class="credit-row"><span>Conventional</span><span class="credit-score">620+</span></div>
            <div class="credit-row"><span>FHA</span><span class="credit-score">580 (500 w/ 10% down)</span></div>
            <div class="credit-row"><span>VA</span><span class="credit-score">No set minimum</span></div>
          </div>
          <div class="rate-note"><strong style="color:var(--forest);">What moves your rate:</strong> credit score, down-payment size, loan type, and how long you lock. Ask me to model two scenarios before you commit — it's free.</div>
        </div>
      </div>
      <div class="value-tool-cta">
        <a href="#apply" class="btn btn-primary" style="background:var(--forest);color:#fff;">Get pre-approved →</a>
        <span class="value-tool-note">No credit pull to start the conversation.</span>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Step 2: Append Testimonials, FAQ, CTA, and Footer**

```html
<section class="section testimonials">
  <div class="section-inner">
    <h2 class="section-h2" style="max-width:560px;margin-bottom:36px;">In their words — buyers and the Realtors who send them.</h2>
    <div class="testimonials-grid">
      <figure class="testimonial-card">
        <span class="testimonial-tag">Realtor partner</span>
        <blockquote>"Quite frankly, this was one of the smoothest transactions with a 21 day close. Chandler was able to explain the entire process to my buyers and offer exceptional service. I will be using his services again in the near future."</blockquote>
        <figcaption>Gerardo M. <span>· Realtor</span></figcaption>
      </figure>
      <figure class="testimonial-card">
        <span class="testimonial-tag">First-time buyer</span>
        <blockquote>"Chandler was the best! He was so awesome to work with, made the process so easy and smooth. Even now after closing he is still so helpful with answering any questions I have. Overall great experience for my first time purchasing a home."</blockquote>
        <figcaption>Victoria S. <span>· Homebuyer</span></figcaption>
      </figure>
      <figure class="testimonial-card">
        <span class="testimonial-tag">Homebuyer</span>
        <blockquote>"His professionalism, efficiency, and expertise made the entire process smooth and stress-free. He was always available to answer my questions promptly and provided clear explanations at every step. Chandler and his team followed through and ensured an on-time closing."</blockquote>
        <figcaption>Jake J. <span>· Homebuyer</span></figcaption>
      </figure>
    </div>
  </div>
</section>

<section id="faq" class="section faq">
  <div class="section-inner faq-inner">
    <div class="faq-head">
      <div class="eyebrow" style="color:var(--forest);opacity:0.7;">Frequently asked</div>
      <h2 class="section-h2">Mortgage questions, answered straight</h2>
    </div>
    <div class="faq-list">
      <details class="faq-item">
        <summary>What does a mortgage broker do, and how is it different from a bank?<span class="faq-icon">+</span></summary>
        <div class="faq-answer">A broker isn't tied to one lender. I shop your loan across many wholesale lenders to find the best fit on rate, program and approval odds — instead of quoting a single bank's rate sheet. You complete one application and I do the comparison shopping for you.</div>
      </details>
      <details class="faq-item">
        <summary>How much do I need for a down payment to buy a home in Houston?<span class="faq-icon">+</span></summary>
        <div class="faq-answer">It depends on the loan. Conventional loans can start at 3% down, FHA at 3.5%, and VA loans require $0 down for eligible veterans. First-time buyers in Texas may also qualify for down-payment assistance of up to 5%.</div>
      </details>
      <details class="faq-item">
        <summary>What credit score do I need to qualify for a mortgage?<span class="faq-icon">+</span></summary>
        <div class="faq-answer">Conventional loans typically want a 620+ score, FHA can go as low as 580 (or 500 with 10% down), and VA has no set minimum. Self-employed and lower-credit buyers can often still qualify through Non-QM and bank-statement programs.</div>
      </details>
      <details class="faq-item">
        <summary>I'm self-employed. Can I still get a mortgage?<span class="faq-icon">+</span></summary>
        <div class="faq-answer">Yes. Non-QM and bank-statement loans let you qualify on 12 to 24 months of bank statements instead of W-2s and tax returns — a specialty of mine for business owners and 1099 earners.</div>
      </details>
      <details class="faq-item">
        <summary>How fast can you close a home loan?<span class="faq-icon">+</span></summary>
        <div class="faq-answer">Clean files can clear-to-close in as few as 10 business days. Timelines vary with the loan type and how quickly documents come in, but fast closings are one of the main reasons buyers and Realtors work with me.</div>
      </details>
      <details class="faq-item">
        <summary>What areas and states do you serve?<span class="faq-icon">+</span></summary>
        <div class="faq-answer">I'm based in Cypress, Texas and serve the greater Houston area. I'm licensed to originate home loans in Texas and Florida.</div>
      </details>
      <details class="faq-item">
        <summary>What does it cost to get pre-approved?<span class="faq-icon">+</span></summary>
        <div class="faq-answer">Nothing to start. An initial consultation and pre-qualification conversation are free, with no credit pull required to begin exploring your options.</div>
      </details>
    </div>
  </div>
</section>

<section id="cta" class="section cta-band">
  <div id="apply" class="cta-inner">
    <h2 class="section-h2" style="color:#fff;font-size:clamp(30px,4.4vw,48px);">Let's find your loan.</h2>
    <p class="cta-lede">One conversation tells you where you stand — no credit pull, no obligation, real answers.</p>
    <div class="cta-buttons">
      <a href="#apply" class="btn btn-primary">Apply Now</a>
      <a href="tel:8324747707" class="btn" style="border:1.5px solid rgba(255,255,255,0.5);color:#fff;">☏ 832-474-7707</a>
    </div>
    <div class="cta-email">chandler@baywayhtx.com</div>
  </div>
</section>

<footer class="footer">
  <div class="footer-inner">
    <div class="footer-top">
      <div class="footer-brand">
        <div class="footer-logo">Bayway <span style="color:var(--meadow);">Mortgage Group</span></div>
        <div class="footer-tagline">Houston Team · Lending with a Purpose.</div>
        <div class="footer-eqhousing">
          <span class="eq-badge">EQ&nbsp;HSG<br>LENDER</span>
          <span>Equal Housing Lender</span>
        </div>
      </div>
      <div class="footer-compliance">
        <div>Chandler Atkinson · NMLS #1851744</div>
        <div>Houston Team Branch · NMLS #1889639</div>
        <div>JRDB, Inc. dba Bayway Mortgage Group</div>
        <div>Company NMLS #1057426</div>
        <div>Licensed in TX &amp; FL</div>
      </div>
    </div>
    <p class="footer-disclosure">Bayway Mortgage Group is a mortgage broker, not a lender; all loans are arranged with third-party wholesale lenders. Loan approval is subject to underwriting and program guidelines. This is not a commitment to lend. Rates and programs are subject to change. Equal Housing Lender.</p>
    <div class="footer-links">
      <a href="https://www.nmlsconsumeraccess.org/" target="_blank" rel="noopener" class="footer-nmls-link">NMLS Consumer Access ↗</a>
      <a href="tel:8324747707">832-474-7707</a>
      <a href="mailto:chandler@baywayhtx.com">chandler@baywayhtx.com</a>
    </div>
  </div>
</footer>

<div class="sticky-bar" data-sticky-bar hidden>
  <a href="tel:8324747707" class="btn btn-secondary" style="flex:1;text-align:center;">Call Chandler</a>
  <a href="#apply" class="btn btn-primary" style="flex:1;text-align:center;">Apply Now</a>
</div>

<script type="module" src="script.js"></script>
</body>
</html>
```

- [ ] **Step 3: Append remaining CSS**

```css
.value-tool { background: var(--cream); }
.value-tool-card { background: #fff; border: 1px solid var(--mist); border-radius: 20px; padding: clamp(28px, 4vw, 48px); }
.value-tool-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(260px, 1fr)); gap: 32px; }
.value-tool-subhead { font-size: 15px; font-weight: 700; color: var(--forest); margin: 0 0 14px; text-transform: uppercase; letter-spacing: 0.04em; }
.checklist { display: flex; flex-direction: column; gap: 11px; }
.checklist-item { display: flex; gap: 11px; align-items: flex-start; font-size: 15px; }
.check-icon { flex: none; width: 18px; height: 18px; border-radius: 5px; background: var(--meadow); color: #fff; display: grid; place-items: center; font-size: 11px; margin-top: 2px; }
.credit-table { display: flex; flex-direction: column; gap: 1px; background: var(--mist); border: 1px solid var(--mist); border-radius: 12px; overflow: hidden; margin-bottom: 22px; }
.credit-row { background: #fff; display: flex; justify-content: space-between; gap: 12px; padding: 12px 16px; font-size: 14.5px; }
.credit-score { font-family: var(--font-mono); color: var(--forest); font-weight: 500; }
.rate-note { background: var(--cream); border-radius: 12px; padding: 16px 18px; font-size: 14px; opacity: 0.85; line-height: 1.55; }
.value-tool-cta { margin-top: 30px; display: flex; flex-wrap: wrap; gap: 12px; align-items: center; }
.value-tool-note { font-size: 13.5px; opacity: 0.6; }

.testimonials { background: var(--mist); }
.testimonials-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 16px; }
.testimonial-card { background: #fff; border-radius: 18px; padding: 28px 26px; margin: 0; display: flex; flex-direction: column; }
.testimonial-tag { font-family: var(--font-mono); font-size: 11px; letter-spacing: 0.08em; text-transform: uppercase; color: var(--meadow); margin-bottom: 16px; }
.testimonial-card blockquote { margin: 0 0 20px; font-size: 15.5px; line-height: 1.62; flex: 1; }
.testimonial-card figcaption { font-size: 14px; color: var(--forest); font-weight: 600; }
.testimonial-card figcaption span { opacity: 0.55; font-weight: 400; color: var(--ink); }

.faq-inner { max-width: 840px; }
.faq-head { text-align: center; margin-bottom: 40px; }
.faq-list { display: flex; flex-direction: column; gap: 12px; }
.faq-item { border: 1px solid var(--mist); border-radius: 14px; background: #fff; overflow: hidden; }
.faq-item summary { cursor: pointer; padding: 20px 24px; font-family: var(--font-display); font-size: clamp(17px, 2vw, 20px); color: var(--forest); display: flex; justify-content: space-between; gap: 18px; align-items: center; list-style: none; }
.faq-item summary::-webkit-details-marker { display: none; }
.faq-icon { flex: none; font-family: var(--font-body); font-weight: 400; font-size: 26px; line-height: 1; color: var(--meadow); transition: transform 0.22s ease; }
.faq-item[open] .faq-icon { transform: rotate(45deg); }
.faq-answer { padding: 0 24px 22px; font-size: 15.5px; opacity: 0.85; line-height: 1.68; }

.cta-band { background: var(--forest); color: #fff; padding-top: clamp(56px, 7vw, 96px); padding-bottom: clamp(56px, 7vw, 96px); }
.cta-inner { max-width: 820px; margin: 0 auto; text-align: center; }
.cta-lede { margin: 0 auto 34px; max-width: 520px; font-size: 17px; opacity: 0.85; }
.cta-buttons { display: flex; flex-wrap: wrap; gap: 14px; justify-content: center; align-items: center; }
.cta-email { margin-top: 20px; font-size: 14px; opacity: 0.7; font-family: var(--font-mono); }

.footer { background: #0a4c36; color: rgba(255, 255, 255, 0.82); padding: clamp(40px, 5vw, 64px) clamp(20px, 4vw, 44px); font-size: 13.5px; line-height: 1.6; }
.footer-inner { max-width: 1120px; margin: 0 auto; }
.footer-top { display: flex; flex-wrap: wrap; gap: 28px 48px; justify-content: space-between; align-items: flex-start; padding-bottom: 28px; border-bottom: 1px solid rgba(255, 255, 255, 0.16); }
.footer-brand { max-width: 380px; }
.footer-logo { font-family: var(--font-display); font-size: 26px; color: #fff; margin-bottom: 6px; }
.footer-tagline { opacity: 0.8; }
.footer-eqhousing { display: flex; align-items: center; gap: 12px; margin-top: 18px; }
.eq-badge { flex: none; width: 38px; height: 38px; border-radius: 8px; border: 1.5px solid rgba(255, 255, 255, 0.5); display: grid; place-items: center; font-size: 9px; font-weight: 700; text-align: center; line-height: 1.1; color: #fff; }
.footer-compliance { font-family: var(--font-mono); font-size: 13px; line-height: 1.9; }
.footer-disclosure { margin: 22px 0 0; opacity: 0.62; max-width: 820px; }
.footer-links { margin-top: 16px; display: flex; flex-wrap: wrap; gap: 18px; }
.footer-links a { text-decoration: none; color: rgba(255, 255, 255, 0.82); }
.footer-nmls-link { color: var(--meadow) !important; font-weight: 600; }

.sticky-bar {
  position: fixed; left: 0; right: 0; bottom: 0; z-index: 50; display: flex; gap: 10px;
  padding: 12px clamp(16px, 4vw, 24px); background: rgba(255, 255, 255, 0.94); backdrop-filter: blur(10px);
  border-top: 1px solid var(--mist); box-shadow: 0 -6px 24px -16px rgba(23, 35, 29, 0.5);
}
.sticky-bar[hidden] { display: none; }
```

- [ ] **Step 4: Write `script.js` with the sticky-bar scroll behavior (only feature that needs JS at this stage)**

```js
const stickyBar = document.querySelector('[data-sticky-bar]');

function updateStickyBar() {
  const pastHero = window.scrollY > 620;
  stickyBar.hidden = !pastHero;
}

window.addEventListener('scroll', updateStickyBar, { passive: true });
updateStickyBar();
```

- [ ] **Step 5: Verify in browser**

Confirm: Value Tool card renders with checklist + credit table, testimonials show all 3 cards, FAQ accordion opens/closes each item independently with the `+` rotating to `×`, CTA band and footer render with correct compliance text, and scrolling past ~620px reveals the sticky Call/Apply bar at the bottom (scrolling back up hides it).

- [ ] **Step 6: Commit**

```bash
git add index.html styles.css script.js
git commit -m "feat: add value-tool baseline, testimonials, faq, cta, footer, and sticky bar"
```

---

### Task 6: Payment calculator logic (TDD)

**Files:**
- Create: `calculator.js`
- Test: `tests/calculator.test.js`

- [ ] **Step 1: Write the failing test**

```js
// tests/calculator.test.js
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { calculateMonthlyPayment } from '../calculator.js';

test('calculates standard amortized payment for a 30-year fixed loan', () => {
  const result = calculateMonthlyPayment({
    price: 300000,
    downPayment: 60000,
    annualRatePercent: 6.5,
    termYears: 30,
  });
  assert.ok(Math.abs(result - 1516.97) < 0.5, `expected ~1516.97, got ${result}`);
});

test('handles a 0% rate as a simple principal / months split', () => {
  const result = calculateMonthlyPayment({
    price: 200000,
    downPayment: 40000,
    annualRatePercent: 0,
    termYears: 30,
  });
  assert.equal(result, 444.44);
});

test('a larger down payment lowers the monthly payment', () => {
  const base = calculateMonthlyPayment({ price: 300000, downPayment: 30000, annualRatePercent: 6.5, termYears: 30 });
  const moreDown = calculateMonthlyPayment({ price: 300000, downPayment: 90000, annualRatePercent: 6.5, termYears: 30 });
  assert.ok(moreDown < base, `expected ${moreDown} < ${base}`);
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `node --test tests/`
Expected: FAIL — `Cannot find module '../calculator.js'` (file doesn't exist yet).

- [ ] **Step 3: Write minimal implementation**

```js
// calculator.js
export function calculateMonthlyPayment({ price, downPayment, annualRatePercent, termYears }) {
  const principal = price - downPayment;
  const numPayments = termYears * 12;

  if (annualRatePercent === 0) {
    return round2(principal / numPayments);
  }

  const monthlyRate = annualRatePercent / 100 / 12;
  const growth = Math.pow(1 + monthlyRate, numPayments);
  const monthlyPayment = (principal * monthlyRate * growth) / (growth - 1);
  return round2(monthlyPayment);
}

function round2(value) {
  return Math.round(value * 100) / 100;
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `node --test tests/`
Expected: PASS — all 3 tests green.

- [ ] **Step 5: Commit**

```bash
git add calculator.js tests/calculator.test.js
git commit -m "feat: add unit-tested payment calculator logic"
```

---

### Task 7: Wire the payment calculator into the Value Tool section

**Files:**
- Modify: `index.html` (add calculator markup inside the buyer Value Tool view)
- Modify: `styles.css` (append calculator rules)
- Modify: `script.js` (import and wire calculator.js)

- [ ] **Step 1: Add calculator markup after the checklist/credit-table grid, inside `.value-tool-card`**

```html
<div class="calculator" data-calculator>
  <h3 class="value-tool-subhead">Model your own scenario</h3>
  <div class="calculator-grid">
    <label class="calculator-field">
      Home price
      <input type="number" data-calc-price value="300000" min="0" step="1000">
    </label>
    <label class="calculator-field">
      Down payment
      <input type="number" data-calc-down value="60000" min="0" step="1000">
    </label>
    <label class="calculator-field">
      Rate assumption (%)
      <input type="number" data-calc-rate value="6.5" min="0" max="15" step="0.125">
    </label>
    <label class="calculator-field">
      Term (years)
      <select data-calc-term>
        <option value="30" selected>30</option>
        <option value="15">15</option>
      </select>
    </label>
  </div>
  <div class="calculator-result">
    <span class="calculator-result-value" data-calc-result>$1,516.97</span>
    <span class="calculator-result-label">estimated monthly principal &amp; interest</span>
  </div>
  <p class="calculator-disclaimer">Estimate only, for illustration. Actual rate and payment depend on credit, program, and lock date — not a quote or commitment to lend.</p>
</div>
```

- [ ] **Step 2: Append calculator CSS**

```css
.calculator { margin-top: 32px; border-top: 1px solid var(--mist); padding-top: 28px; }
.calculator-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 16px; margin-bottom: 20px; }
.calculator-field { display: flex; flex-direction: column; gap: 6px; font-size: 13px; font-weight: 600; color: var(--forest); }
.calculator-field input, .calculator-field select {
  font-family: var(--font-mono); font-size: 15px; padding: 10px 12px; border: 1px solid var(--mist); border-radius: 8px; color: var(--ink);
}
.calculator-result { display: flex; flex-direction: column; gap: 4px; background: var(--cream); border-radius: 12px; padding: 18px 20px; margin-bottom: 14px; }
.calculator-result-value { font-family: var(--font-mono); font-size: 28px; color: var(--forest); font-weight: 500; }
.calculator-result-label { font-size: 13px; opacity: 0.7; }
.calculator-disclaimer { margin: 0; font-size: 12.5px; opacity: 0.6; line-height: 1.5; }
```

- [ ] **Step 3: Wire the calculator in `script.js`**

```js
import { calculateMonthlyPayment } from './calculator.js';

const calculator = document.querySelector('[data-calculator]');
if (calculator) {
  const priceInput = calculator.querySelector('[data-calc-price]');
  const downInput = calculator.querySelector('[data-calc-down]');
  const rateInput = calculator.querySelector('[data-calc-rate]');
  const termSelect = calculator.querySelector('[data-calc-term]');
  const resultEl = calculator.querySelector('[data-calc-result]');

  function formatCurrency(value) {
    return value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
  }

  function updateResult() {
    const price = Number(priceInput.value) || 0;
    const downPayment = Number(downInput.value) || 0;
    const annualRatePercent = Number(rateInput.value) || 0;
    const termYears = Number(termSelect.value) || 30;

    const monthly = calculateMonthlyPayment({ price, downPayment, annualRatePercent, termYears });
    resultEl.textContent = formatCurrency(monthly);
  }

  [priceInput, downInput, rateInput, termSelect].forEach((el) => {
    el.addEventListener('input', updateResult);
  });

  updateResult();
}
```

- [ ] **Step 4: Verify in browser**

Open `index.html`, scroll to the Value Tool section. Confirm the calculator shows a starting result of `$1,516.97`, and changing home price/down payment/rate/term updates the result live. Try down payment > home price and confirm it doesn't crash (shows `$0.00` or a negative-but-non-crashing value is acceptable — this is a self-serve estimate tool, not a validated loan application).

- [ ] **Step 5: Commit**

```bash
git add index.html styles.css script.js
git commit -m "feat: wire payment calculator into value tool section"
```

---

### Task 8: Fork mechanism — audience state and section filtering

**Files:**
- Create: `fork-state.js`
- Test: `tests/fork-state.test.js`
- Modify: `index.html` (add "Switch view" pill, `data-audience-tag`/`data-audience-view` already present from Tasks 4-5)
- Modify: `styles.css` (append filter + pill rules)
- Modify: `script.js` (wire fork buttons and filtering)

- [ ] **Step 1: Write the failing test for the storage module**

```js
// tests/fork-state.test.js
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { getStoredAudience, setStoredAudience } from '../fork-state.js';

function createMemoryStorage() {
  const data = new Map();
  return {
    getItem: (key) => (data.has(key) ? data.get(key) : null),
    setItem: (key, value) => data.set(key, value),
  };
}

test('getStoredAudience returns null when nothing stored', () => {
  const storage = createMemoryStorage();
  assert.equal(getStoredAudience(storage), null);
});

test('setStoredAudience then getStoredAudience round-trips', () => {
  const storage = createMemoryStorage();
  setStoredAudience('buyer', storage);
  assert.equal(getStoredAudience(storage), 'buyer');
});

test('setStoredAudience rejects invalid values', () => {
  const storage = createMemoryStorage();
  assert.throws(() => setStoredAudience('nope', storage), /Invalid audience/);
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `node --test tests/`
Expected: FAIL — `Cannot find module '../fork-state.js'`.

- [ ] **Step 3: Write minimal implementation**

```js
// fork-state.js
const STORAGE_KEY = 'bayway-audience';

export function getStoredAudience(storage) {
  const value = storage.getItem(STORAGE_KEY);
  return value === 'buyer' || value === 'realtor' ? value : null;
}

export function setStoredAudience(audience, storage) {
  if (audience !== 'buyer' && audience !== 'realtor') {
    throw new Error(`Invalid audience: ${audience}`);
  }
  storage.setItem(STORAGE_KEY, audience);
}
```

- [ ] **Step 4: Run test to verify it passes**

Run: `node --test tests/`
Expected: PASS — all 6 tests green (3 from Task 6, 3 new).

- [ ] **Step 5: Add the "Switch view" pill to `index.html`, right after the nav**

```html
<button type="button" class="switch-view-pill" data-switch-view hidden>
  Viewing: <span data-switch-view-label>Buyer</span> · Switch view
</button>
```

- [ ] **Step 6: Append CSS for filtering and the pill**

```css
/* Fork filtering: body gets data-audience="buyer"|"realtor"; hide the non-matching tagged elements */
body[data-audience="buyer"] [data-audience-tag="realtor"] { display: none; }
body[data-audience="realtor"] [data-audience-tag="buyer"] { display: none; }
body[data-audience="buyer"] [data-audience-view="realtor"] { display: none; }
body[data-audience="realtor"] [data-audience-view="buyer"] { display: none; }

.switch-view-pill {
  position: fixed; top: 70px; right: 20px; z-index: 39;
  background: #fff; border: 1px solid var(--mist); border-radius: 999px;
  padding: 8px 16px; font-size: 13px; font-family: var(--font-mono); color: var(--forest);
  cursor: pointer; box-shadow: 0 6px 20px -12px rgba(23, 35, 29, 0.4);
}
.switch-view-pill[hidden] { display: none; }
```

- [ ] **Step 7: Wire the fork in `script.js`**

```js
import { getStoredAudience, setStoredAudience } from './fork-state.js';

const forkButtons = document.querySelectorAll('[data-audience-choice]');
const switchPill = document.querySelector('[data-switch-view]');
const switchLabel = document.querySelector('[data-switch-view-label]');

function applyAudience(audience) {
  document.body.dataset.audience = audience;
  setStoredAudience(audience, window.localStorage);
  switchPill.hidden = false;
  switchLabel.textContent = audience === 'buyer' ? 'Buyer' : 'Realtor';
}

forkButtons.forEach((button) => {
  button.addEventListener('click', () => {
    applyAudience(button.dataset.audienceChoice);
  });
});

switchPill?.addEventListener('click', () => {
  const current = document.body.dataset.audience;
  applyAudience(current === 'buyer' ? 'realtor' : 'buyer');
});

const storedAudience = getStoredAudience(window.localStorage);
if (storedAudience) {
  applyAudience(storedAudience);
}
```

- [ ] **Step 8: Verify in browser**

Open `index.html`. Confirm: before choosing, all 6 program cards show. Click "I'm buying a home" — confirm Programs narrows to Conventional/FHA/VA/First-Time Buyer (4 cards) and the "Switch view" pill appears reading "Viewing: Buyer". Click the pill — confirm it flips to Realtor view (Non-QM/DSCR cards only). Reload the page — confirm the last-chosen audience persists (via `localStorage`).

- [ ] **Step 9: Commit**

```bash
git add fork-state.js tests/fork-state.test.js index.html styles.css script.js
git commit -m "feat: revive audience fork with persisted state and program filtering"
```

---

### Task 9: "What happens after you apply" timeline

**Files:**
- Modify: `index.html` (insert new section before or after the Value Tool section)
- Modify: `styles.css` (append timeline rules)

- [ ] **Step 1: Insert the timeline section into `index.html`, immediately after the Value Tool section's closing `</section>`**

```html
<section class="section timeline">
  <div class="section-inner">
    <div class="eyebrow" style="color:var(--forest);opacity:0.7;">What to expect</div>
    <h2 class="section-h2">What happens after you apply</h2>
    <p class="section-lede" style="margin-bottom:32px;">A realistic walk-through, not a promise — every file moves a little differently.</p>
    <div class="timeline-steps">
      <div class="timeline-step">
        <div class="timeline-day">Day 1–2</div>
        <h3>Application &amp; documents</h3>
        <p>You submit your application and the documents from the checklist above. I review for anything missing right away.</p>
      </div>
      <div class="timeline-step">
        <div class="timeline-day">Day 3–9</div>
        <h3>Underwriting</h3>
        <p>Your file goes to the lender we've matched you with for full underwriting review — income, credit, and the property itself.</p>
      </div>
      <div class="timeline-step">
        <div class="timeline-day">As few as 10 business days</div>
        <h3>Clear-to-close</h3>
        <p>Clean, complete files can reach clear-to-close in as few as 10 business days from application.</p>
      </div>
      <div class="timeline-step">
        <div class="timeline-day">Scheduled</div>
        <h3>Closing day</h3>
        <p>Once clear-to-close, we schedule your closing with the title company and you sign.</p>
      </div>
    </div>
    <p class="timeline-disclaimer">Timelines vary with loan type and how quickly documents come in — this is a realistic range, not a guarantee.</p>
  </div>
</section>
```

- [ ] **Step 2: Append timeline CSS**

```css
.timeline { background: #ffffff; }
.timeline-steps { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 20px; }
.timeline-step { border: 1px solid var(--mist); border-radius: 14px; padding: 22px 20px; }
.timeline-day { font-family: var(--font-mono); font-size: 13px; color: var(--wheat); background: var(--forest); display: inline-block; padding: 4px 10px; border-radius: 6px; margin-bottom: 12px; }
.timeline-step h3 { font-size: 17px; margin: 0 0 8px; color: var(--forest); }
.timeline-step p { margin: 0; font-size: 14.5px; opacity: 0.8; line-height: 1.55; }
.timeline-disclaimer { margin: 24px 0 0; font-size: 13px; opacity: 0.6; }
```

- [ ] **Step 3: Verify in browser**

Confirm the timeline section renders between the Value Tool and Testimonials sections, with 4 steps in a responsive grid, and the disclaimer is visible below.

- [ ] **Step 4: Commit**

```bash
git add index.html styles.css
git commit -m "feat: add application timeline content block"
```

---

### Task 10: Realtor co-marketing kit (gated via Netlify Forms)

**Files:**
- Modify: `index.html` (add Realtor Value Tool view + gated form)
- Modify: `styles.css` (append kit/form rules)
- Modify: `script.js` (wire the reveal-on-success behavior)

**Note on content:** per the brainstorm decision, the copy below is a realistic placeholder clearly marked for your review before launch — it is not Bayway's actual current Partner Program terms. Replace the marked block with real copy before this goes live.

- [ ] **Step 1: Add the Realtor Value Tool view, as a sibling to the buyer `.value-tool-card` inside `.value-tool-inner`**

```html
<div class="value-tool-card realtor-kit" data-audience-view="realtor" hidden>
  <div class="eyebrow" style="color:var(--wheat);">For Realtor partners</div>
  <h2 class="section-h2">The co-marketing kit</h2>
  <p class="section-lede" style="margin-bottom:32px;">Co-branded flyers, the Monday Market Brief, and Bayway's Realtor Partner Program terms — unlocked by email.</p>

  <!-- REPLACE WITH ACTUAL BAYWAY PARTNER TERMS before launch — this is placeholder body copy, not confirmed program details. -->
  <div class="realtor-kit-summary">
    <p>Bayway's Houston Team Realtor Partner Program includes co-branded single-property flyers and sites, a weekly Monday Market Brief you can forward to your sphere, and priority handling on your buyers' files — including the Non-QM and DSCR programs most lenders won't touch.</p>
  </div>
  <!-- END placeholder block -->

  <form
    name="realtor-kit"
    method="POST"
    data-netlify="true"
    netlify-honeypot="bot-field"
    class="realtor-kit-form"
    data-kit-form
  >
    <input type="hidden" name="form-name" value="realtor-kit">
    <p hidden><label>Don't fill this out: <input name="bot-field"></label></p>

    <label class="calculator-field">
      Name
      <input type="text" name="name" required>
    </label>
    <label class="calculator-field">
      Email
      <input type="email" name="email" required>
    </label>
    <label class="calculator-field">
      Brokerage (optional)
      <input type="text" name="brokerage">
    </label>
    <button type="submit" class="btn btn-primary" style="margin-top:16px;">Get the co-marketing kit →</button>
  </form>

  <div class="realtor-kit-unlocked" data-kit-unlocked hidden>
    <p>Thanks — check your email for the co-marketing kit, and welcome to the Monday Market Brief list.</p>
    <a href="#cta" class="btn btn-secondary">Or just partner with me directly →</a>
  </div>
</div>
```

- [ ] **Step 2: Append kit CSS**

```css
.realtor-kit-summary { background: var(--cream); border-radius: 12px; padding: 18px 20px; font-size: 14.5px; line-height: 1.6; margin-bottom: 24px; }
.realtor-kit-form { display: grid; gap: 16px; max-width: 420px; }
.realtor-kit-unlocked { background: var(--cream); border-radius: 12px; padding: 20px; display: flex; flex-direction: column; gap: 14px; align-items: flex-start; }
```

- [ ] **Step 3: Wire the AJAX submit + reveal in `script.js`**

Netlify Forms works with a plain HTML POST (no JS required at all), but submitting that way navigates away from the page. This intercepts the submit to keep the visitor on-page and reveal the unlock message instead.

```js
const kitForm = document.querySelector('[data-kit-form]');
const kitUnlocked = document.querySelector('[data-kit-unlocked]');

if (kitForm) {
  kitForm.addEventListener('submit', async (event) => {
    event.preventDefault();
    const formData = new FormData(kitForm);
    try {
      await fetch('/', {
        method: 'POST',
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
        body: new URLSearchParams(formData).toString(),
      });
      kitForm.hidden = true;
      kitUnlocked.hidden = false;
    } catch (error) {
      console.error('Realtor kit form submission failed', error);
      alert('Something went wrong submitting the form — please try again or email chandler@baywayhtx.com directly.');
    }
  });
}
```

- [ ] **Step 4: Verify in browser (local limitations apply)**

Open `index.html` locally and switch to the Realtor fork view. Confirm the co-marketing kit card renders with the summary text, the form, and the fields are all present. Submitting the form locally will fail the `fetch('/')` call (there's no Netlify Forms endpoint until deployed) — confirm the `catch` block's alert fires rather than the page crashing. Full working submission is verified after Task 12's deploy.

- [ ] **Step 5: Commit**

```bash
git add index.html styles.css script.js
git commit -m "feat: add gated realtor co-marketing kit with netlify forms"
```

---

### Task 11: Scrollytelling motion

**Files:**
- Modify: `index.html` (add `data-count-target` to remaining numeric stats, `data-stagger-group` to card grids)
- Modify: `styles.css` (append motion keyframes + reduced-motion overrides)
- Modify: `script.js` (add IntersectionObserver-driven counters, stagger, and ribbon draw)

- [ ] **Step 1: Tag the remaining numeric stat for count-up in `index.html`**

The "10 days" stat already has `data-count-target="10" data-count-suffix=" days"` from Task 4. Update its inner value markup so JS has an empty target to fill:

```html
<div class="stat" data-count-target="10" data-count-suffix=" days">
  <div class="stat-value" data-count-value>0 days</div>
  <div class="stat-label">Closings as fast as</div>
</div>
```

- [ ] **Step 2: Add `data-stagger-group` to the Programs grid and Testimonials grid wrappers**

```html
<div class="programs-grid" data-programs-grid data-stagger-group>
```
```html
<div class="testimonials-grid" data-stagger-group>
```

- [ ] **Step 3: Add motion CSS with `prefers-reduced-motion` overrides**

```css
[data-stagger-group] > * {
  opacity: 0;
  transform: translateY(10px) scale(0.98);
  transition: opacity 500ms ease-out, transform 500ms cubic-bezier(0.2, 0.8, 0.2, 1);
}
[data-stagger-group].in-view > *.stagger-visible {
  opacity: 1;
  transform: translateY(0) scale(1);
}

.ribbon-path { stroke-dasharray: 1400; stroke-dashoffset: 1400; }
.ribbon.in-view .ribbon-path { animation: ribbon-draw 1200ms ease-out forwards; }
@keyframes ribbon-draw { to { stroke-dashoffset: 0; } }

@media (prefers-reduced-motion: reduce) {
  [data-stagger-group] > * {
    opacity: 1 !important;
    transform: none !important;
    transition: none !important;
  }
  .ribbon-path { stroke-dashoffset: 0 !important; }
  .ribbon.in-view .ribbon-path { animation: none !important; }
}
```

- [ ] **Step 4: Add IntersectionObserver logic to `script.js`**

```js
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

if (!prefersReducedMotion) {
  // Count-up numbers
  const countTargets = document.querySelectorAll('[data-count-target]');
  const countObserver = new IntersectionObserver((entries, observer) => {
    entries.forEach((entry) => {
      if (!entry.isIntersecting) return;
      const el = entry.target;
      const target = Number(el.dataset.countTarget);
      const suffix = el.dataset.countSuffix || '';
      const valueEl = el.querySelector('[data-count-value]');
      const duration = 900;
      const start = performance.now();

      function tick(now) {
        const progress = Math.min((now - start) / duration, 1);
        const current = Math.round(progress * target);
        valueEl.textContent = `${current}${suffix}`;
        if (progress < 1) requestAnimationFrame(tick);
      }
      requestAnimationFrame(tick);
      observer.unobserve(el);
    });
  }, { threshold: 0.5 });
  countTargets.forEach((el) => countObserver.observe(el));

  // Staggered card entry
  const staggerGroups = document.querySelectorAll('[data-stagger-group]');
  const staggerObserver = new IntersectionObserver((entries, observer) => {
    entries.forEach((entry) => {
      if (!entry.isIntersecting) return;
      const group = entry.target;
      group.classList.add('in-view');
      Array.from(group.children).forEach((child, index) => {
        setTimeout(() => child.classList.add('stagger-visible'), index * 120);
      });
      observer.unobserve(group);
    });
  }, { threshold: 0.2 });
  staggerGroups.forEach((el) => staggerObserver.observe(el));

  // Ribbon self-draw
  const ribbon = document.querySelector('[data-ribbon]');
  if (ribbon) {
    const ribbonObserver = new IntersectionObserver((entries, observer) => {
      entries.forEach((entry) => {
        if (!entry.isIntersecting) return;
        entry.target.classList.add('in-view');
        observer.unobserve(entry.target);
      });
    }, { threshold: 0.3 });
    ribbonObserver.observe(ribbon);
  }
} else {
  // Reduced motion: render final states immediately, no observers needed.
  document.querySelectorAll('[data-count-target]').forEach((el) => {
    const valueEl = el.querySelector('[data-count-value]');
    if (valueEl) valueEl.textContent = `${el.dataset.countTarget}${el.dataset.countSuffix || ''}`;
  });
}
```

- [ ] **Step 5: Verify in browser**

Reload `index.html`, scroll slowly. Confirm: the "10 days" stat counts up from 0 once it enters view; Program cards and Testimonial cards pop in with a slight stagger the first time they scroll into view (and don't re-trigger if you scroll past and back); the hero ribbon draws itself once on load/scroll into view. Then, in browser DevTools, enable "prefers-reduced-motion: reduce" (Chrome DevTools → Rendering tab → Emulate CSS media feature), reload, and confirm everything renders in its final state immediately with no animation.

- [ ] **Step 6: Commit**

```bash
git add index.html styles.css script.js
git commit -m "feat: add scrollytelling motion with reduced-motion support"
```

---

### Task 12: Final QA and Netlify deploy

**Files:** none created — verification and deployment only.

- [ ] **Step 1: Run the full test suite**

Run: `node --test tests/`
Expected: PASS — all 9 tests (calculator + fork-state) green.

- [ ] **Step 2: Full manual walkthrough in browser**

Open `index.html` fresh (clear `localStorage` first via DevTools → Application → Local Storage → clear) and check, in order: nav links scroll correctly; hero fork buttons switch Programs and Value Tool content; Switch-view pill works; calculator computes and updates live; timeline renders with disclaimer; testimonials/FAQ/CTA/footer all render; sticky bar appears past the hero and hides above it; reduced-motion mode (per Task 11 Step 5) shows no animation.

- [ ] **Step 3: Deploy to Netlify**

If a Netlify site isn't already linked to the `chandleros-bit/bayway-landing-page` GitHub repo, connect it via the Netlify dashboard: "Add new site" → "Import an existing project" → select the repo → build command empty, publish directory `.` (matches `netlify.toml`). Netlify auto-detects the `realtor-kit` form from the deployed HTML.

- [ ] **Step 4: Verify the live form**

On the deployed Netlify URL, switch to the Realtor fork view, submit the co-marketing kit form with a real email address. Confirm the on-page "unlocked" message appears, and in the Netlify dashboard under Forms, confirm the submission was received.

- [ ] **Step 5: Commit and push any final fixes**

```bash
git add -A
git commit -m "chore: final QA fixes before launch"
git push
```

(Skip this step if QA found no issues to fix.)

---

## Self-Review Notes

- **Spec coverage:** every §03–§07 item from the design doc has a task — fork (Task 8), motion (Task 11), calculator (Tasks 6–7), Realtor kit (Task 10), timeline (Task 9), plus the baseline page port (Tasks 1–5) and deploy (Task 12).
- **Placeholder scan:** the only marked placeholder is the Realtor kit's summary copy in Task 10, which was an explicit, informed decision (not a plan gap) — clearly commented for pre-launch replacement, per the brainstorming session's decision.
- **Type/name consistency checked:** `calculateMonthlyPayment` (Task 6) is imported with the same name and parameter shape in Task 7; `getStoredAudience`/`setStoredAudience` (Task 8) match their test file and their only call site in `script.js`; `data-audience-tag`/`data-audience-view`/`data-audience-choice` attribute names are used consistently across Tasks 3, 4, 5, and 8.
