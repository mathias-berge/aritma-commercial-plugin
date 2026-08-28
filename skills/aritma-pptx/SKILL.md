---
name: aritma-pptx
description: >
  Creates on-brand PowerPoint presentations and single-page documents for Aritma, including pitch decks, product one-pagers, customer guides, reference decks, and sales enablement material. Use this skill whenever anyone at Aritma asks to create, build, or update a PowerPoint, slide deck, presentation, one-pager, or any customer-facing document in .pptx format. Also triggers for internal decks, all-hands slides, investor decks, or any sales/marketing collateral. Always applies the current Aritma brand guidelines, the Aritma Figma design system, and Aritma product knowledge. Never produce generic-looking slides. Every output must feel unmistakably Aritma.
---

# Aritma PPTX Skill

## Before you start

Read `/mnt/skills/public/pptx/pptxgenjs.md` for the full PptxGenJS API reference. This skill adds Aritma brand constraints on top of it.

Use this authority order when values conflict:

1. Aritma Figma Design System: `https://www.figma.com/design/DPq9HRIkW8EVjjuijfOF3f/Aritma-%7C-Design-System?node-id=2-3&t=dDErqL1hImM746Mz-1`
2. Uploaded Aritma Brand Guidelines
3. This `SKILL.md`
4. Older presentation or asset-library examples

Do not use colors, typefaces, or layout conventions from older decks if they conflict with the current Figma design system or brand guidelines.

---

## Quick decision tree

| Output type | Approach |
|------------|---------|
| Slide deck (3+ slides) | PptxGenJS from scratch using the current Aritma brand system |
| Product one-pager | Single landscape slide, dense card layout |
| Customer guide / reference | Multi-slide, off-white content slides with section dividers |
| Investor / all-hands | Dark-heavy approach: title + section dividers in Indigo, content slides in Off-white |

---

## Aritma brand system

### Colors (no `#` prefix in PptxGenJS)

Use these exact values unless the Figma file has newer token values.

```javascript
// Core brand colors
INDIGO        = "45095A"   // primary brand color, dark backgrounds, headings, deep accents
PINK          = "FB8CAF"   // primary accent, preferred logo color on light backgrounds, highlights
BLUE          = "2549D2"   // interactive accent, buttons, links, callouts
RED           = "F33F3F"   // danger, destructive actions, error states
OFF_WHITE     = "F9F0EE"   // default slide background for content slides
BLACK         = "110F0E"   // primary text on light backgrounds
WHITE         = "FFFFFF"   // text on dark backgrounds and white cards

// Indigo / purple scale
INDIGO_900    = "0C0348"   // darkest backgrounds
INDIGO_800    = "2C0372"
INDIGO_700    = "45095A"   // brand primary
INDIGO_600    = "6B0F8A"
INDIGO_500    = "8C15B0"
INDIGO_400    = "9E4AC0"   // hover states
INDIGO_300    = "B673D1"
INDIGO_200    = "D1A5E5"   // light accents
INDIGO_100    = "E7D8F4"   // subtle backgrounds
INDIGO_50     = "F3EFF9"   // lightest tint

// Blue scale
BLUE_900      = "10183E"
BLUE_700      = "1A3470"
BLUE_500      = "2549D2"   // interactive primary
BLUE_400      = "5479E2"   // hover states
BLUE_300      = "80A0E5"
BLUE_200      = "ADC4EA"
BLUE_100      = "D8E3F8"

// Neutral scale
NEUTRAL_900   = "141418"   // near-black text
NEUTRAL_700   = "434546"   // dark gray text
NEUTRAL_500   = "75717A"   // secondary text, captions
NEUTRAL_300   = "D7D4DB"   // borders and dividers
NEUTRAL_100   = "ECEDEF"   // light backgrounds
NEUTRAL_50    = "F6F6F7"   // subtle backgrounds
```

### Color usage rules

- Use **Indigo `45095A`** for dark slide backgrounds, primary headings, deep accents, and brand-heavy moments.
- Use **Pink `FB8CAF`** for logo usage on light backgrounds, accent bars, decorative highlights, and selected dark-slide accents.
- Use **Blue `2549D2`** for buttons, links, badges, callouts, and interactive-looking elements.
- Use **Off-white `F9F0EE`** as the default content slide background.
- Use **Black `110F0E`** or Neutral 900 `141418` for primary text on light backgrounds.
- Use **Neutral 700 `434546`** for body text on light backgrounds.
- Use **Neutral 500 `75717A`** for captions, labels, and muted metadata.
- Use **Neutral 300 `D7D4DB`** for borders and dividers.
- Do not use the older off-palette values from the previous skill, such as `2D1B4E`, `F5A0B5`, `E8799A`, `F5EDE6`, `5C5270`, `8B7FA3`, or `D4C4E8`, unless the user explicitly asks to recreate an old visual.

### Accessibility

- Normal text must meet WCAG AA contrast: 4.5:1 minimum.
- Large text must meet 3:1 minimum.
- UI components and visual states must meet 3:1 minimum.
- Do not rely on color alone to communicate status or meaning.
- Avoid Blue text on Indigo backgrounds unless contrast has been checked.

---

## Typography

### Font families

| Use | Preferred font | Fallback |
|---|---|---|
| Display / hero / title slide | Aritma Uten 1.0 | Inter, then system sans-serif |
| Body / content slides | Inter | Arial or Calibri when Inter is unavailable |
| Technical fallback in PptxGenJS | Inter | Arial |

PptxGenJS can only use fonts available to the environment opening the deck. Use `Aritma Uten` and `Inter` in generated files, but expect PowerPoint to substitute fonts if they are not installed.

```javascript
const FONT_DISPLAY = "Aritma Uten";
const FONT_BODY = "Inter";
const FONT_FALLBACK = "Arial";
```

### Slide typography scale

| Element | Size | Weight | Color on light bg | Color on dark bg | Font |
|---------|------|--------|-------------------|------------------|------|
| Cover title | 44-52pt | Regular or SemiBold | n/a | WHITE | Aritma Uten |
| Section divider title | 44-52pt | Regular or SemiBold | n/a | WHITE | Aritma Uten |
| Content slide title | 32-36pt | SemiBold | INDIGO | n/a | Inter |
| Subtitle / description | 13-15pt | Regular | NEUTRAL_500 or PINK | PINK or INDIGO_200 | Inter |
| Card heading | 15-18pt | SemiBold | INDIGO | WHITE | Inter |
| Body text | 11-13pt | Regular | NEUTRAL_700 | INDIGO_200 | Inter |
| Stat value | 22-30pt | SemiBold | INDIGO | WHITE | Inter |
| Stat label | 9-10pt | Medium | NEUTRAL_500 | INDIGO_200 | Inter |
| Caption / "Confidential" | 9pt | Italic or Regular | NEUTRAL_500 | INDIGO_200 | Inter |
| Page number | 9pt | Regular | NEUTRAL_500 | INDIGO_200 | Inter |
| Section number ("01") | 16pt | Regular | n/a | PINK | Inter |

### Typography rules

- Use sentence case for headings.
- Avoid all caps except acronyms or short labels.
- For all-caps labels, add letter spacing where technically possible.
- Keep body text concise. Prefer short, scannable sentences.
- Use Inter for most presentation content because it is more readable than display type at small sizes.

---

## Slide format

- Layout: `LAYOUT_16x9` (10" × 5.625")
- Default content background: Off-white `F9F0EE`
- Default dark background: Indigo `45095A`
- Default body font: Inter, with Arial fallback
- Use an 8px-derived spacing logic translated to PowerPoint inches. Prefer consistent spacing over pixel-perfect conversion.

---

## Page furniture (every slide)

Apply these to every slide, light and dark:

- A small confidentiality label in the top-left.
- A logo in the footer, not the top-right, unless a user-provided template dictates otherwise.
- A page number in the bottom-right.

Logo rules:

- Light/off-white slides: use the Pink logo as first choice.
- Dark/Indigo slides: use a White logo if available. If the packaged skill only contains `logo_pink.png` and `logo_purple.png`, use Pink on dark slides as the fallback.
- Use Indigo logo only as a secondary option on light backgrounds when Pink is visually too prominent.
- Do not stretch, rotate, recolor, shadow, outline, or distort logos.

```javascript
const fs = require("fs");
const path = require("path");

function imgB64(file) {
  return "image/png;base64," + fs.readFileSync(path.join(__dirname, "assets", file)).toString("base64");
}

const LOGO_PINK_B64 = imgB64("logo_pink.png");
const LOGO_INDIGO_B64 = imgB64("logo_purple.png");
const LOGO_WHITE_B64 = fs.existsSync(path.join(__dirname, "assets", "logo_white.png"))
  ? imgB64("logo_white.png")
  : LOGO_PINK_B64;

function addPageFurniture(slide, slideNum, isDark = false, opts = {}) {
  const FONT_BODY = "Inter";
  const logoData = isDark ? LOGO_WHITE_B64 : (opts.useIndigoLogo ? LOGO_INDIGO_B64 : LOGO_PINK_B64);

  // Confidential label, top-left
  slide.addText(opts.confidentialLabel || "Confidential", {
    x: 0.5, y: 0.2, w: 2.0, h: 0.22,
    fontSize: 9, italic: true, fontFace: FONT_BODY,
    color: isDark ? "D1A5E5" : "75717A",
    margin: 0
  });

  // Aritma logo, footer-left
  slide.addImage({
    data: logoData,
    x: 0.5, y: 5.18, w: 1.49, h: 0.22
  });

  // Page number, footer-right
  slide.addText(String(slideNum), {
    x: 9.3, y: 5.2, w: 0.5, h: 0.22,
    fontSize: 9, fontFace: FONT_BODY, align: "right",
    color: isDark ? "D1A5E5" : "75717A",
    margin: 0
  });
}
```

The `assets/` folder sits next to `generate.js` in the working directory. Copy `logo_pink.png` and `logo_purple.png` from the skill's own `assets/` folder to your working directory before running. Add `logo_white.png` when available from the official logo package.

---

## Slide types

### Title slide (dark)

```javascript
slide.background = { color: "45095A" };

// Decorative circles, use Indigo scale with transparency
slide.addShape(pres.shapes.OVAL, {
  x: -1.5, y: 0.5, w: 4.5, h: 4.5,
  fill: { color: "6B0F8A", transparency: 60 },
  line: { color: "6B0F8A", transparency: 100 }
});
slide.addShape(pres.shapes.OVAL, {
  x: 7.5, y: 3.0, w: 3.5, h: 3.5,
  fill: { color: "2C0372", transparency: 70 },
  line: { color: "2C0372", transparency: 100 }
});

// Title centered
slide.addText("Presentation title", {
  x: 1, y: 1.75, w: 8, h: 1.25,
  fontSize: 48, bold: false, fontFace: "Aritma Uten",
  color: "FFFFFF", align: "center",
  margin: 0
});

// Thin divider below title
slide.addShape(pres.shapes.LINE, {
  x: 3.5, y: 3.03, w: 3.0, h: 0,
  line: { color: "D1A5E5", width: 0.75 }
});

// Subtitle
slide.addText("Subtitle or date", {
  x: 1, y: 3.18, w: 8, h: 0.4,
  fontSize: 16, fontFace: "Inter",
  color: "FB8CAF", align: "center",
  margin: 0
});
```

### Section divider (dark)

```javascript
slide.background = { color: "45095A" };

// Section number
slide.addText("01", {
  x: 0.7, y: 1.55, w: 1.0, h: 0.35,
  fontSize: 16, fontFace: "Inter", color: "FB8CAF", margin: 0
});

// Section title
slide.addText("Section name", {
  x: 0.7, y: 1.95, w: 7.4, h: 1.2,
  fontSize: 48, bold: false, fontFace: "Aritma Uten",
  color: "FFFFFF", margin: 0
});

// Section description
slide.addText("Optional description text", {
  x: 0.7, y: 3.2, w: 6.2, h: 0.5,
  fontSize: 14, fontFace: "Inter", color: "FB8CAF", margin: 0
});

// Optional geometric watermark
slide.addText(">", {
  x: 8.1, y: 3.35, w: 0.7, h: 0.8,
  fontSize: 54, bold: true, fontFace: "Inter",
  color: "6B0F8A", transparency: 35, margin: 0
});
```

### Content slide (light/off-white)

```javascript
slide.background = { color: "F9F0EE" };

// Title block
slide.addText("Slide title", {
  x: 0.5, y: 0.6, w: 8.7, h: 0.6,
  fontSize: 34, bold: true, fontFace: "Inter",
  color: "45095A", margin: 0
});

// Optional subtitle
slide.addText("Subtitle or context", {
  x: 0.5, y: 1.24, w: 8.7, h: 0.35,
  fontSize: 13, fontFace: "Inter",
  color: "75717A", margin: 0
});

// Content starts at y ≈ 1.7
```

### Closing/contact slide (dark)

Same as title slide: Indigo background, subtle decorative circles, White heading, Pink accent, Indigo-200 or White contact details.

---

## Card system

Cards are the primary content container on off-white slides. Do not place body content directly on Off-white. Use White cards with controlled spacing and a Pink accent where useful.

```javascript
function addCard(slide, x, y, w, h, opts = {}) {
  const makeShadow = () => ({
    type: "outer", blur: 6, offset: 2, angle: 135,
    color: "000000", opacity: 0.06
  });

  // Card body, rounded to match brand radius guidance
  slide.addShape(pres.shapes.ROUNDED_RECTANGLE, {
    x, y, w, h,
    fill: { color: "FFFFFF" },
    line: { color: opts.borderColor || "FFFFFF", transparency: opts.borderColor ? 0 : 100 },
    rectRadius: 0.08,
    shadow: makeShadow()
  });

  // Optional Pink accent bar, inset to preserve rounded card corners
  if (opts.accent !== false) {
    slide.addShape(pres.shapes.RECTANGLE, {
      x: x + 0.06, y: y + 0.12, w: 0.055, h: h - 0.24,
      fill: { color: "FB8CAF" },
      line: { color: "FB8CAF", transparency: 100 }
    });
  }

  const contentX = x + (opts.accent === false ? 0.22 : 0.22);
  const contentW = w - 0.35;

  // Card heading
  if (opts.heading) {
    slide.addText(opts.heading, {
      x: contentX, y: y + 0.16, w: contentW, h: 0.35,
      fontSize: opts.headingSize || 16, bold: true, fontFace: "Inter",
      color: opts.headingColor || "45095A", margin: 0
    });
  }

  // Card body text
  if (opts.body) {
    slide.addText(opts.body, {
      x: contentX, y: y + (opts.heading ? 0.58 : 0.22),
      w: contentW, h: h - (opts.heading ? 0.75 : 0.35),
      fontSize: opts.bodySize || 12, fontFace: "Inter",
      color: opts.bodyColor || "434546", margin: 0,
      breakLine: false,
      fit: "shrink"
    });
  }
}
```

### Card layout patterns

| Pattern | Dimensions |
|---------|-----------|
| Full-width | x: 0.5, w: 9.0 |
| Two-column | each w: 4.3, gap: 0.4 |
| Three-column | each w: 2.8, gap: 0.35 |
| Four-box (2×2) | each 4.3 × 1.6 |
| Six-box (2×3) | each 2.8 × 1.5 |

### Card rules

- Use White cards on Off-white backgrounds.
- Use Pink accent bars sparingly, not on every tiny element if it creates noise.
- Use Blue for actionable CTA cards or link-like elements.
- Use Neutral 300 for subtle dividers and card borders.
- Use a consistent radius: approximate 8-12px visually.

---

## Aritma product knowledge

### Product portfolio

**Aritma Open Finance Platform**: The core API platform enabling Nordic bank integrations. ERP-agnostic middleware connecting ERPs to banks via ISO 20022 and bank-specific file formats. Licensed payment institution in Norway.

**Aritma Payments** (Pay in some older icon libraries): Payment initiation module. Sends payment files (`pain.001` ISO 20022) to banks and processes return files (`pain.002`, `camt.054`). Handles SFTP/API connectivity per bank. Primary market: Microsoft Dynamics 365 Business Central customers in the Nordics.

**Aritma Reconciliation** (Control in some older icon libraries): Bank reconciliation and cash management. Fetches bank statements (`camt.052` intraday, `camt.053` end-of-day), matches transactions against ERP entries. Key value: automated bank reconciliation within Business Central.

**Smart Bookkeeping**: A feature within Reconciliation currently available for customers using Visma's BNXT ERP. When creating content for BNXT audiences, highlight Smart Bookkeeping as the relevant Reconciliation capability.

**Aritma Finance Manager**: The Business Central-embedded app combining Payments and Reconciliation in one user interface. Target audience: finance teams in Business Central-using companies.

**Gateway** (in development, placeholder name): A new access point to the Aritma platform. Not ready for customer-facing content yet. Do not include in decks unless explicitly asked.

> Note: Aritma Commerce is no longer marketed or sold. Do not include it in customer-facing content unless the user explicitly asks for historical or legacy material.

### Key differentiators

- Payment institution license, allowing Aritma to hold funds and initiate payments on behalf of customers where relevant.
- Direct bank connections across Nordic banks, including DNB, Nordea, SEB, Handelsbanken, Swedbank, Danske Bank, and more.
- ISO 20022 native: `pain.001`, `pain.002`, `camt.052`, `camt.053`, `camt.054`, `pacs`.
- Business Central embedded: works inside Dynamics 365 Business Central with no additional UI for BC users.
- Nordic-specific formats and schemes: BG MAX, Bankgiro, OCR/KID, eFaktura, AvtaleGiro.
- Regulatory foundation: PSD2 PISP/AISP licensing and ISO 20022 migration support.

### Target customers

- Nordic mid-market companies using Microsoft Dynamics 365 Business Central.
- Treasury teams managing multi-bank payment flows.
- Companies with high payment volumes or multi-currency complexity.
- ERP partners, resellers, and VARs building Business Central solutions.
- Visma BNXT users for Reconciliation and Smart Bookkeeping.

### Pricing tiers (Aritma Payments)

Verify current pricing before using in customer-facing material.

- Small: NOK 2 500/month: 1 000 payment + 1 000 receivable transactions, NOK 2.50/excess.
- Medium: NOK 5 000/month: 2 500 + 2 500, NOK 2.00/excess.
- Large: NOK 8 750/month: 5 000 + 5 000, NOK 1.75/excess.
- Enterprise: talk to sales.

### Pricing tiers (Aritma Reconciliation)

Verify current pricing before using in customer-facing material.

- Small: NOK 2 500/month: unlimited users, 1 000 transactions, NOK 2.50/excess.
- Medium: NOK 5 000/month: 5 000 transactions, NOK 1.00/excess.
- Large: NOK 8 750/month: 10 000 transactions, NOK 0.875/excess.
- Enterprise: talk to sales.

---

## Icons from the brand asset library

Use official Aritma/Heydays custom icons where available. Use `react-icons` equivalents only when building programmatically and no official SVG asset is available. Match colors: Indigo `45095A` for light backgrounds, Pink `FB8CAF` for emphasis, White for dark backgrounds, and Blue `2549D2` for interactive elements.

| Category | Use cases | Suggested react-icons fallback |
|---------|-----------|-------------------------------|
| Essential icons | General UI, arrows, chevrons | `FaChevronUp`, `FaArrowRight`, `FaLock`, `FaEye` |
| Financial icons | Payments, banking, invoices | `FaUniversity`, `FaCreditCard`, `FaFileInvoiceDollar`, `FaChartLine` |
| Ecommerce icons | Transactions, carts, orders | `FaShoppingCart`, `FaBox`, `FaExchangeAlt` |
| Infographics icons | KPIs, dashboards, data | `FaChartBar`, `FaTachometerAlt`, `FaBullseye` |
| Marketing icons | Growth, campaigns, leads | `FaBullhorn`, `FaRocket`, `FaHandshake` |
| Teamwork icons | People, collaboration | `FaUsers`, `FaUserTie`, `FaHandshake` |
| Media icons | Communications, files | `FaEnvelope`, `FaPhone`, `FaFileAlt` |

### Icon-in-circle pattern

```javascript
async function addIconInCircle(slide, IconComp, color, bgColor, x, y, size = 0.45) {
  const iconData = await iconToBase64Png(IconComp, color, 256);

  // Circle background
  slide.addShape(pres.shapes.OVAL, {
    x, y, w: size, h: size,
    fill: { color: bgColor },
    line: { color: bgColor, transparency: 100 }
  });

  // Icon centered in circle
  const iconSize = size * 0.5;
  const offset = (size - iconSize) / 2;
  slide.addImage({ data: iconData, x: x + offset, y: y + offset, w: iconSize, h: iconSize });
}

// Usage: Indigo icon on subtle Indigo tint
// await addIconInCircle(slide, FaUniversity, "45095A", "E7D8F4", 0.7, 1.8);

// Usage: Blue icon on subtle Blue tint for interactive or CTA context
// await addIconInCircle(slide, FaArrowRight, "2549D2", "D8E3F8", 0.7, 1.8);
```

---

## Numbered badge (agenda slides)

```javascript
function addBadge(slide, num, x, y) {
  slide.addShape(pres.shapes.ROUNDED_RECTANGLE, {
    x, y, w: 0.42, h: 0.42,
    fill: { color: "45095A" },
    line: { color: "45095A", transparency: 100 },
    rectRadius: 0.08
  });
  slide.addText(String(num).padStart(2, "0"), {
    x, y, w: 0.42, h: 0.42,
    fontSize: 13, bold: true, fontFace: "Inter",
    color: "FB8CAF", align: "center", valign: "middle",
    margin: 0
  });
}
```

---

## Stat callout pattern

```javascript
function addStat(slide, label, value, x, y, w = 1.8) {
  slide.addText(label.toUpperCase(), {
    x, y, w, h: 0.25,
    fontSize: 9, bold: true, fontFace: "Inter",
    color: "75717A", margin: 0
  });
  slide.addText(value, {
    x, y: y + 0.28, w, h: 0.6,
    fontSize: 26, bold: true, fontFace: "Inter",
    color: "45095A", margin: 0
  });
}
```

---

## SVG illustrations (programmatic)

Five standard illustration types, generated inline using PptxGenJS image from an SVG buffer:

| Type | When to use |
|------|-------------|
| `network` | Integration, connectivity, bank connections |
| `flow` | Process, workflow, onboarding steps |
| `growth` | Performance, volume growth, KPIs |
| `shield` | Security, compliance, PSD2, licensing |
| `gears` | Operations, technical, ERP integration |

Use Indigo `45095A`, Pink `FB8CAF`, Blue `2549D2`, and light tints such as Indigo-100 `E7D8F4` or Blue-100 `D8E3F8` in illustrations. Generate as SVG strings rendered via `sharp` to PNG base64.

---

## Template conventions

At the start of any deck or one-pager request, ask whether to:

1. Base it on an existing template, asking the user to share it, or
2. Create a new layout from scratch.

For product one-pagers, there may be a standard template. If the user has not shared it in the current conversation, ask whether they want to use it before proceeding.

Remember any templates or reference documents shared in conversation and offer to reuse them in follow-up requests.

---

## Tone of voice for copy

- Professional but approachable: B2B fintech, clear, and grounded.
- Lead with the problem, then present Aritma as the answer.
- Short sentences. Minimal jargon, or explain jargon when it is needed.
- Metrics and facts over adjectives.
- Use active voice.
- Keep one idea per sentence where possible.
- Use "Confidential" on external-facing decks and "Proprietary & Confidential" for internal or strategic decks when appropriate.

---

## Infographic and layout library (brand asset file)

The file `Aritma_Logo_Icons_and_Infographics.pptx` is Aritma's official brand asset library. It contains ready-made slide layouts for copy-paste use in presentations. When a deck needs one of these components, refer to this file.

> Setup: Place the file at `assets/Aritma_Logo_Icons_and_Infographics.pptx` next to this skill's `SKILL.md`. Without it, layout references still work as guidance, but you will not be able to copy elements directly from the file.

> Important: The library may contain historical slides or discontinued product references. The Figma design system and the uploaded brand guidelines are authoritative for current color, typography, and product-use decisions.

### What's in the file

| Section | Slides | What it contains |
|---------|--------|-----------------|
| Logos | 2 | Logo variants and reference for correct logo proportions. |
| Product Icons | 3 | Custom icons for Open Finance Platform, Pay/Payments, Control/Reconciliation, and legacy Commerce. Do not use Commerce in customer-facing material unless explicitly requested. |
| Diverse Icons & Illustrations | 4-5 | Geometric pattern icons and flat illustration scenes. |
| Essential Icons | 6-7, 22-23 | General UI icons: navigation, actions, notifications, locks, settings, and related symbols. |
| Financial Icons | 8-9 | Banking, payments, invoices, charts, wallets, currencies, POS terminals. |
| Ecommerce Icons | 10-11, 24-25 | Shopping, orders, delivery, products, carts. Use only when contextually relevant. |
| Infographics Icons | 12-13 | Data visualization, KPIs, analytics, maps, charts. |
| Marketing Icons | 14-15 | Funnels, megaphones, targeting, campaigns, growth. |
| Media Icons | 16-17 | Communications, files, email, phone, video, social media. |
| Teamwork Icons | 18-21 | People, collaboration, organizations, meetings. |
| Europe Map | 26-29 | Editable Europe map. Copy whole map or individual countries. Comes in variants: simple map, map + stat callout, map + donut chart, map + bar chart. |
| Pricing plans | 30-31 | Pre-built pricing tables for Payments and Reconciliation. Verify current pricing before use. Ignore Commerce pricing unless explicitly asked for historical material. |
| Stakeholder Maps | 33-52 | Approximately 20 stakeholder diagram variants: ring diagrams, hub-and-spoke, grid matrices, org-style trees, Venn-style overlaps. |
| KPI Dashboards | 53-76 | Approximately 24 dashboard layouts: stat cards with sparklines, gauge widgets, bar/line/area charts, donut charts, comparisons, multi-metric layouts. |
| Timeline | 77-84 | Approximately 8 timeline variants: horizontal milestone, vertical milestone, zigzag, compact table-style. |
| Vision Statements | 85-103 | Approximately 19 content layout slides: 2-up, 4-up, quadrant layouts, list + icon combinations. |

### Visual style of the library

Use layouts from the library as structural references. Before generating new content, normalize all colors to the current Aritma brand system:

- Indigo `45095A`
- Pink `FB8CAF`
- Blue `2549D2`
- Red `F33F3F`
- Off-white `F9F0EE`
- Black `110F0E`
- Neutral scale from the current brand guidelines

### When to use which section

- Need a map of Europe: slides 26-29.
- Presenting pricing to a customer: slides 30-31, after verifying pricing.
- Need a process or milestone flow: slides 77-84.
- Showing KPIs or financial metrics: slides 53-76.
- Need a 2-4 column content layout: slides 85-103.
- Need icons for a card or feature list: slides 6-21.

---

## QA checklist

After generating, always:

1. Run `extract-text output.pptx` and check for missing or truncated content.
2. Convert to images and visually inspect:
   - Logo appears in the footer on every slide unless a user-provided template specifies otherwise.
   - Confidential label appears top-left on every slide.
   - Page number appears bottom-right on every slide.
   - Pink accent bars use `FB8CAF`.
   - Content slides use Off-white `F9F0EE`, not `F5EDE6`.
   - Dark slides use Indigo `45095A`, not `2D1B4E`.
   - Light slide headings use Indigo `45095A`.
   - Body text uses Black `110F0E`, Neutral 900 `141418`, or Neutral 700 `434546`.
   - No text overflows card boundaries.
   - Cards use White surfaces and consistent corner radius.
3. Check no incorrect legacy skill colors slipped in:
   - `2D1B4E`
   - `F5A0B5`
   - `E8799A`
   - `F5EDE6`
   - `5C5270`
   - `8B7FA3`
   - `D4C4E8`
4. Check that Aritma Uten is used only for display/title moments and Inter is used for readable body content.
5. Check that all customer-facing pricing and product claims have been verified if they are decision-critical.

---

## Setup for each generation

```bash
npm install -g pptxgenjs react react-dom react-icons sharp
```

Install or make available the preferred fonts where possible:

- Aritma Uten 1.0 for display/title text.
- Inter for content and body text.

Copy logos to the working directory:

```bash
mkdir -p ./assets
cp /path/to/skill/assets/logo_pink.png ./assets/
cp /path/to/skill/assets/logo_purple.png ./assets/
# Optional when available from the official logo package:
cp /path/to/official-assets/logo_white.png ./assets/
```

Then run:

```bash
node generate.js
```
