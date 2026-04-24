---
name: salween-brand-visual
description: "Apply the correct visual identity when creating Word documents (.docx files) for Salween Group or its clients. Use this skill at the start of ANY .docx creation task — before writing a single line of document code — to determine which brand to apply and get the exact colours, fonts, and styling rules. Trigger whenever the user asks to create, generate, or produce a Word document, report, brief, deliverable, or any .docx output. Also trigger when the user says things like 'make it on-brand', 'use the brand colours', 'apply our styling', or 'use the Salween brand'. If another skill (e.g. b2b-positioning, b2b-content-strategy) is producing a .docx deliverable, this skill's brand rules take precedence over any generic styling defaults in that skill."
---

# Salween Brand Visual Identity

## What this skill does

This skill ensures every Word document produced by Salween Group — whether for internal use or client delivery — uses the correct visual identity. It handles two cases:

1. **Salween Group identity** — the default. Use this when producing documents under the Salween Group brand.
2. **Client identity** — use this when a client brand file exists and the user wants the document styled for the client, not for Salween.

Run this skill's brand-detection step at the very start of any `.docx` creation task, before writing any document code.

---

## Step 1: Detect which brand to use

Before generating the document, check whether a client brand file exists:

```bash
# Check for a client brand file in the working directory
ls brand/brand-visual.md 2>/dev/null
```

**If `brand/brand-visual.md` exists**, use `AskUserQuestion` to ask:

> "A client brand identity file was found (`brand/brand-visual.md`). Which visual identity should I use for this document?"
> - a) Salween Group visual identity (Salween Red / Platinum / Dark Jungle Green)
> - b) Client's brand identity (from `brand/brand-visual.md`)

If the user chooses **(b)**, read `brand/brand-visual.md` in full and apply whatever colours, fonts, and styling rules it specifies instead of the Salween defaults below. If the file is incomplete or ambiguous, ask the user to clarify before proceeding.

**If `brand/brand-visual.md` does not exist**, proceed with the Salween Group identity below — no need to ask.

---

## Step 2: Apply the brand

### Salween Group visual identity (default)

#### Colour palette

| Name | Hex | Use |
|------|-----|-----|
| Salween Red | `#E4032F` | H1 text, accent elements |
| Platinum | `#E8E5E3` | Background panels, callout shading |
| Dark Jungle Green | `#121E21` | Optional dark accent (dividers, covers) |
| Black | `#000000` | H2 text, body text |
| White | `#FFFFFF` | Page background, text on dark fills |

#### Typography

| Element | Font | Weight | Colour |
|---------|------|--------|--------|
| H1 (Heading 1) | DM Serif Display | Regular | Salween Red `#E4032F` |
| H2 (Heading 2) | DM Serif Display | Regular | Black `#000000` |
| H3 and below | DM Sans | Regular | Black `#000000` |
| Body / paragraph | DM Sans | Regular | Black `#000000` |
| Callout / caption | DM Sans | Regular | Black `#000000` |

#### Heading capitalisation

All H1 and H2 text must use **AP Style Title Case**:
- Capitalise the first and last word, and all "major" words (nouns, verbs, adjectives, adverbs)
- Lowercase articles (*a*, *an*, *the*), coordinating conjunctions (*and*, *but*, *or*, *nor*, *for*, *so*, *yet*), and prepositions of fewer than five letters (*at*, *by*, *for*, *in*, *of*, *on*, *to*, *up*)
- Always capitalise the first word after a colon

**Examples:**
- ✅ `Competitive Differentiation and Market Strategy`
- ✅ `Why Content Marketing Works for B2B Companies`
- ✅ `Three Goals for StartupXYZ: A Strategic Overview`
- ❌ `Competitive differentiation and market strategy`
- ❌ `Why content marketing works for B2B companies`

H3 and below are **sentence case** (capitalise only the first word and proper nouns).

> **Font availability note:** DM Serif Display and DM Sans are Google Fonts. They must be installed on the machine where Word will open the document, or embedded. When generating via `docx-js`, specify font names exactly as above — Word will substitute if not installed, but the document will render correctly on machines where the fonts are present.

#### Applying styles in docx-js

Use this styles block as your base when creating a new document with the `docx` npm package:

```javascript
const SALWEEN_RED = "E4032F";
const BLACK = "000000";
const PLATINUM = "E8E5E3";
const DARK_JUNGLE_GREEN = "121E21";

styles: {
  default: {
    document: {
      run: { font: "DM Sans", size: 24, color: BLACK } // 12pt body default
    }
  },
  paragraphStyles: [
    {
      id: "Heading1", name: "Heading 1",
      basedOn: "Normal", next: "Normal", quickFormat: true,
      run: { font: "DM Serif Display", size: 40, color: SALWEEN_RED, bold: false },
      paragraph: { spacing: { before: 320, after: 160 }, outlineLevel: 0 }
    },
    {
      id: "Heading2", name: "Heading 2",
      basedOn: "Normal", next: "Normal", quickFormat: true,
      run: { font: "DM Serif Display", size: 32, color: BLACK, bold: false },
      paragraph: { spacing: { before: 240, after: 120 }, outlineLevel: 1 }
    },
    {
      id: "Heading3", name: "Heading 3",
      basedOn: "Normal", next: "Normal", quickFormat: true,
      run: { font: "DM Sans", size: 26, color: BLACK, bold: true },
      paragraph: { spacing: { before: 180, after: 80 }, outlineLevel: 2 }
    },
  ]
}
```

#### Common brand elements

**Callout / highlighted panel** — use Platinum as the shading fill:
```javascript
shading: { fill: "E8E5E3", type: ShadingType.CLEAR }
```

**Section divider line** — use Salween Red as a paragraph border:
```javascript
paragraph: {
  border: { bottom: { style: BorderStyle.SINGLE, size: 6, color: "E4032F", space: 1 } }
}
```

**Header text** — "SALWEEN GROUP" in DM Sans, Dark Jungle Green:
```javascript
new Header({
  children: [new Paragraph({
    children: [new TextRun({ text: "SALWEEN GROUP", font: "DM Sans", color: "121E21", size: 18 })]
  })]
})
```

---

## Step 3: Hand off to the docx skill

Once the brand is determined, proceed with document generation using the `docx` skill. The docx skill handles all the technical mechanics (page size, tables, images, validation). This skill provides the brand layer — colours and fonts — that the docx skill applies.

When writing document generation code:
- Always import this skill's colour constants and styles block first
- Override any generic Arial/blue defaults in the docx skill's examples with the Salween values above
- Validate the output with `python scripts/office/validate.py` after generation

---

## Quick reference card

```
Salween Red  →  #E4032F   (H1 text, accents)
Platinum     →  #E8E5E3   (panel backgrounds)
Dark Jungle  →  #121E21   (dark accents, header)
Black        →  #000000   (H2, body)

H1  →  DM Serif Display, Regular, #E4032F, AP Title Case
H2  →  DM Serif Display, Regular, #000000, AP Title Case
H3+ →  DM Sans, Regular, #000000, sentence case
Body →  DM Sans, Regular, #000000
```
