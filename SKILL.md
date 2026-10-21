---
name: salween-brand-visual
description: "Apply the correct visual identity when creating Word documents (.docx files) for Salween Group or its clients. Use this skill at the start of ANY .docx creation task — before writing a single line of document code — to determine which brand to apply and get the exact colours, fonts, and styling rules. Trigger whenever the user asks to create, generate, or produce a Word document, report, brief, deliverable, or any .docx output. Also trigger when the user says things like 'make it on-brand', 'use the brand colours', 'apply our styling', or 'use the Salween brand'. If another skill (e.g. positioning-dunford, b2b-content-strategy) is producing a .docx deliverable, this skill's brand rules take precedence over any generic styling defaults in that skill."
---

# Salween Brand Visual Identity

**Version:** 1.1.1

## What this skill does

This skill ensures every Word document produced by Salween Group — whether for internal use or client delivery — uses the correct visual identity. It handles two cases:

1. **Salween Group identity** — the default. Use this when producing documents under the Salween Group brand.
2. **Client identity** — use this when a client brand file exists and the user wants the document styled for the client, not for Salween.

It also applies a default **"DRAFT 1" watermark** to every document it generates (Step 3). The watermark is a standard Word watermark, so the recipient can remove it in Word via **Design ▸ Watermark ▸ Remove Watermark**.

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

**H1 only** uses **AP Style Title Case**:
- Capitalise the first and last word, and all "major" words (nouns, verbs, adjectives, adverbs)
- Lowercase articles (*a*, *an*, *the*), coordinating conjunctions (*and*, *but*, *or*, *nor*, *for*, *so*, *yet*), and prepositions of fewer than five letters (*at*, *by*, *for*, *in*, *of*, *on*, *to*, *up*)
- Always capitalise the first word after a colon

**H2 and below** use **sentence case** (capitalise only the first word and proper nouns).

**Examples:**
- ✅ H1: `Competitive Differentiation and Market Strategy`
- ✅ H1: `Three Goals for StartupXYZ: A Strategic Overview`
- ✅ H2: `Competitive differentiation and market strategy`
- ✅ H2: `Why content marketing works for B2B companies`
- ❌ H2: `Competitive Differentiation And Market Strategy`

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

## Step 3: Add the default DRAFT 1 watermark

Every document this skill produces carries a light-grey diagonal **"DRAFT 1"** watermark by default, on **both** the Salween and client identities. It is a genuine Word watermark living in the page header, so:

- It repeats on every page and sits behind the body text.
- The recipient can remove it in Word via **Design ▸ Watermark ▸ Remove Watermark** (the shape is named `PowerPlusWaterMarkObject…`, which is exactly what that command looks for), or by double-clicking into the header and deleting it.

docx-js (v9.x) has no native watermark API, so inject the standard Word VML watermark with `ImportedXmlComponent`. Add this once when building the document:

```javascript
const { ImportedXmlComponent } = require("docx");

const WATERMARK_TEXT = "DRAFT 1"; // change to "DRAFT 2", "CONFIDENTIAL", etc.; omit the paragraph entirely for a final version

const watermarkXml = `<w:p xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main" xmlns:v="urn:schemas-microsoft-com:vml" xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:w10="urn:schemas-microsoft-com:office:word">
  <w:r>
    <w:rPr><w:noProof/></w:rPr>
    <w:pict>
      <v:shapetype id="_x0000_t136" coordsize="21600,21600" o:spt="136" adj="10800" path="m@7,l@8,m@5,21600l@6,21600e">
        <v:formulas>
          <v:f eqn="sum #0 0 10800"/><v:f eqn="prod #0 2 1"/><v:f eqn="sum 21600 0 @1"/>
          <v:f eqn="sum 0 0 @2"/><v:f eqn="sum 21600 0 @3"/><v:f eqn="if @0 @3 0"/>
          <v:f eqn="if @0 21600 @1"/><v:f eqn="if @0 0 @2"/><v:f eqn="if @0 @4 21600"/>
          <v:f eqn="mid @5 @6"/><v:f eqn="mid @8 @5"/><v:f eqn="mid @7 @8"/>
          <v:f eqn="mid @6 @7"/><v:f eqn="sum @6 0 @5"/>
        </v:formulas>
        <v:path textpathok="t" o:connecttype="custom" o:connectlocs="@9,0;@10,10800;@11,21600;@12,10800" o:connectangles="270,180,90,0"/>
        <v:textpath on="t" fitshape="t"/>
        <o:lock v:ext="edit" text="t" shapetype="t"/>
      </v:shapetype>
      <v:shape id="PowerPlusWaterMarkObject1" o:spid="_x0000_s2049" type="#_x0000_t136" style="position:absolute;margin-left:0;margin-top:0;width:468pt;height:117pt;rotation:315;z-index:-251654144;mso-position-horizontal:center;mso-position-horizontal-relative:margin;mso-position-vertical:center;mso-position-vertical-relative:margin" o:allowincell="f" fillcolor="#D8D8D8" stroked="f">
        <v:textpath style="font-family:&quot;DM Sans&quot;;font-size:1pt" string="${WATERMARK_TEXT}"/>
      </v:shape>
    </w:pict>
  </w:r>
</w:p>`;

// CRITICAL: take `.root[0]`. fromXmlString wraps the parsed XML in a phantom root
// element, so passing the raw result emits a stray <undefined> tag that corrupts
// the .docx. `.root[0]` unwraps it to the real <w:p>.
const watermarkParagraph = ImportedXmlComponent.fromXmlString(watermarkXml).root[0];
```

Wire it into the section's **header** (not the body) so it repeats on every page:

```javascript
sections: [{
  properties: { page: { size: { width: 12240, height: 15840 } } },
  headers: {
    default: new Header({ children: [
      watermarkParagraph,
      // ...any other header content (e.g. the "SALWEEN GROUP" line below) goes here too
    ]})
  },
  children: [ /* document body */ ]
}]
```

**Notes**
- **One instance per header.** `ImportedXmlComponent` objects are single-use during rendering — build a fresh `ImportedXmlComponent.fromXmlString(watermarkXml).root[0]` for each header you attach it to; don't reuse the same variable.
- **Title-page / even-odd headers.** If the section sets `titlePage: true` or distinct even/odd headers, add a watermark paragraph to each of those `Header` objects too, giving each `<v:shape>` a unique `id` (`PowerPlusWaterMarkObject1`, `…2`, `…3`) so it appears on every page.
- **Multi-section documents.** Repeat the watermark in each section's header.
- The text auto-scales to the shape (`fitshape="t"`), so leave `font-size:1pt` as-is. Adjust `fillcolor` for a lighter/darker mark.

---

## Step 4: Hand off to the docx skill

Once the brand is determined, proceed with document generation using the `docx` skill. The docx skill handles all the technical mechanics (page size, tables, images, validation). This skill provides the brand layer — colours, fonts, and the default watermark — that the docx skill applies.

When writing document generation code:
- Always import this skill's colour constants and styles block first
- Add the default DRAFT 1 watermark to the section header (Step 3)
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
H2  →  DM Serif Display, Regular, #000000, sentence case
H3+ →  DM Sans, Regular, #000000, sentence case
Body →  DM Sans, Regular, #000000

Watermark →  "DRAFT 1", diagonal, #D8D8D8, in page header (removable in Word)
```

## Version history

| Version | Date | Change |
|---|---|---|
| 1.1.1 | 2026-09-18 | Cross-reference updated for skill renames |
| 1.1.0 | 2026-09-14 | H1 takes AP Title Case; H2 and below are sentence case (rule, examples and quick-reference table) |
| 1.0.0 | 2026-04-22 | Initial release: Salween/client brand detection, typography, colours, docx-js styles block; DRAFT 1 watermark added 2026-05-28 |
