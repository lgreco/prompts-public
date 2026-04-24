# Agent for airplane log reviews

I'm preparing a pre-purchase maintenance review for aircraft **{{TAIL_NUMBER}}** (a **{{YEAR}} {{MAKE_MODEL}}**, S/N **{{SERIAL_NUMBER}}**). Attached are scanned logbook PDFs — every file whose name begins with **{{TAIL_NUMBER}}** is a source document for this review. Most source documents are logbooks, some may auxiliary documentation such as the sale listing of the aircraft.

The make, model, year, and serial number of the aircraft are mentioned in the attached logs. Infer the logbook type (airframe, engine, propeller, avionics, etc.) from contents; filenames may help but are not guaranteed. Also infer the time span covered by each logbook, by the contents of that logbook.

---

## **Task**

Visually read every page of every logbook (assume OCR may be unreliable). Transcribe **every maintenance entry** into a single chronological record.

For each entry capture:

* Date of work
* Tach and/or total-time reading (if shown)
* Summary of work performed *(preserve part numbers, AD references, work orders)*
* Shop name and/or signing mechanic with certificate number
* Category tag from:
  **AF, ENG, PROP, AVX, INSP, GEAR, DOC, GAP**

### **Transcription Rules**

* For unclear handwriting: transcribe best effort and mark uncertainty with `[?]`
* If pages are missing/torn/skipped: explicitly note
* Do not silently infer or “clean up” ambiguous technical details

---

## **Gap Detection**

* Identify periods ≥13 months with no entries
* Insert a **GAP row**:

  * Show date span
  * Estimate hours flown (if possible)
* GAP rows must be visually distinct (see Styling)
* Do **not** interpret or editorialize about gaps

---

## **Missing Logbooks**

If expected logs are absent:

* State clearly on **Cover Page**
* Repeat in **About This Document**

---

## **Output: Generate a single PDF (ReportLab)**

### **Global Layout Constraints (MANDATORY)**

These are critical to prevent formatting failures:

* Page size: Letter (8.5" × 11")
* Margins: **0.75" on all sides**
* Maximum usable width must be respected at all times
* **NO text may extend beyond margins** under any circumstance
* Use flowable elements only (no absolute-position text blocks unless bounded)
* All tables must:

  * Use fixed or percentage column widths that sum ≤ available width
  * Enable word wrapping (`wordWrap='CJK'` or equivalent)
  * Prevent overflow using `splitByRow=True`
* Long text (especially summaries) must wrap cleanly within column bounds
* Never allow a row to render off-page horizontally

---

## **Typography (STRICT)**

* **Body text:** 11 pt serif (e.g., Times-Roman)
* **Headers:** Helvetica (bold where appropriate)
* **Footers:** 9 pt serif
* Line spacing: 1.2–1.3
* Ensure consistent font usage throughout (no mixing defaults)

---

## **Document Structure**

### **1. Cover Page**

Include:

* Tail number, aircraft info
* Coverage span (earliest → latest entry)
* Bullet list of source logs:

  * filename or inferred label
  * page count
  * handwritten vs typed
  * era covered
* Category legend

---

### **2. About This Document**

* Paragraph 1: schema description
* Paragraph 2: note gaps and missing logs

---

### **3. Chronology**

* Group entries by **year**
* Strict chronological order

Each entry rendered as a **row with controlled layout**:

* Left column (fixed width): Date + Tach/TT
* Middle: Category pill
* Right (flex, wrapped): Summary
* Mechanic/shop: italicized line below summary

#### **Table Requirements**

* Column widths must be explicitly defined and sum ≤ usable page width
* Summary column must wrap (never overflow)
* Rows must split cleanly across pages if needed
* Avoid excessively tall rows (split long summaries if needed)

---

### **4. Preliminary Recommendation**

Short, direct:

* Should the club consider purchase?
* Identify risks and notable concerns
* No excessive narrative

---

### **5. Appendix: Summary Statistics**

Two-column table:

* Total entries
* Coverage span
* Airframe TT (first/latest)
* Hours flown
* Entries per category
* GAP periods (count + list)
* Engine overhauls
* Prop replacements
* Tach resets
* Ownership/location indicators

---

### **6. Disclaimer**

Standard non-authoritative transcription disclaimer

---

## **Styling**

* Header bar on every page (except cover):

  * Dark background
  * Helvetica font
  * Aircraft ID
* Footer:

  * Page number
  * “Consolidated from: …”
* Category pills:

  * Color-coded
* GAP rows:

  * Light red background tint
* Tables:

  * Subtle grid lines
  * Adequate padding (no cramped text)

---

## **Final QA Pass (REQUIRED)**

Before producing the final PDF, perform a **format validation step**:

1. **Check for horizontal overflow**

   * No text outside margins
   * No clipped content

2. **Check table integrity**

   * All columns respect width constraints
   * No cell exceeds its boundary

3. **Check typography**

   * Body = 11 pt serif
   * Headers = Helvetica
   * Consistent usage throughout

4. **Check wrapping**

   * Long summaries wrap cleanly
   * No truncation

5. **Check pagination**

   * No orphaned headers
   * No broken rows mid-render (unless properly split)

6. If any issue is detected:

   * **Adjust layout and reflow before finalizing**
   * Do not emit a broken PDF

---

## **Output Requirement**

Use the ReportLab template below.

Return **only the final PDF** (well-formatted, print-ready). Do not include intermediate artifacts.

## ReportLab Template

```python
from reportlab.lib import colors
from reportlab.lib.enums import TA_LEFT, TA_CENTER
from reportlab.lib.pagesizes import LETTER
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib.units import inch
from reportlab.platypus import (
    SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle,
    PageBreak, KeepTogether
)

PAGE_SIZE = LETTER
MARGIN = 0.75 * inch
USABLE_WIDTH = PAGE_SIZE[0] - 2 * MARGIN

SERIF = "Times-Roman"
SERIF_BOLD = "Times-Bold"
SERIF_ITALIC = "Times-Italic"
SANS = "Helvetica"
SANS_BOLD = "Helvetica-Bold"

CATEGORY_COLORS = {
    "AF": colors.HexColor("#D9EAF7"),
    "ENG": colors.HexColor("#E5F2D8"),
    "PROP": colors.HexColor("#FFF1CC"),
    "AVX": colors.HexColor("#E8DDF5"),
    "INSP": colors.HexColor("#DDEFEA"),
    "GEAR": colors.HexColor("#F7E0D4"),
    "DOC": colors.HexColor("#E6E6E6"),
    "GAP": colors.HexColor("#F8D7DA"),
}

def build_styles():
    styles = getSampleStyleSheet()

    styles.add(ParagraphStyle(
        name="Body11",
        fontName=SERIF,
        fontSize=11,
        leading=14,
        spaceAfter=6,
        wordWrap="CJK",
    ))

    styles.add(ParagraphStyle(
        name="BodyItalic11",
        fontName=SERIF_ITALIC,
        fontSize=10.5,
        leading=13,
        textColor=colors.HexColor("#444444"),
        wordWrap="CJK",
    ))

    styles.add(ParagraphStyle(
        name="Header",
        fontName=SANS_BOLD,
        fontSize=14,
        leading=17,
        spaceBefore=10,
        spaceAfter=6,
    ))

    styles.add(ParagraphStyle(
        name="YearHeader",
        fontName=SANS_BOLD,
        fontSize=13,
        leading=16,
        textColor=colors.white,
        backColor=colors.HexColor("#333333"),
        leftIndent=4,
        spaceBefore=10,
        spaceAfter=4,
    ))

    styles.add(ParagraphStyle(
        name="Small",
        fontName=SERIF,
        fontSize=9,
        leading=11,
        wordWrap="CJK",
    ))

    styles.add(ParagraphStyle(
        name="CategoryPill",
        fontName=SANS_BOLD,
        fontSize=8,
        leading=10,
        alignment=TA_CENTER,
        textColor=colors.black,
    ))

    return styles


def header_footer(canvas, doc):
    canvas.saveState()

    width, height = PAGE_SIZE

    if doc.page > 1:
        canvas.setFillColor(colors.HexColor("#222222"))
        canvas.rect(0, height - 0.45 * inch, width, 0.45 * inch, fill=1, stroke=0)

        canvas.setFillColor(colors.white)
        canvas.setFont(SANS_BOLD, 10)
        canvas.drawString(
            MARGIN,
            height - 0.30 * inch,
            f"{doc.aircraft_id}"
        )

    canvas.setFillColor(colors.HexColor("#555555"))
    canvas.setFont(SERIF, 9)
    canvas.drawString(MARGIN, 0.35 * inch, f"Consolidated from: {doc.source_short}")
    canvas.drawRightString(width - MARGIN, 0.35 * inch, f"Page {doc.page}")

    canvas.restoreState()


def category_pill(cat, styles):
    return Paragraph(cat, styles["CategoryPill"])


def chronology_row(entry, styles):
    """
    entry = {
        "date": "2006-07-10",
        "tach_tt": "Tach 1234.5 / TT 4567.8",
        "category": "AF",
        "summary": "...",
        "signer": "Signed by ..."
    }
    """

    cat = entry.get("category", "DOC")
    bg = CATEGORY_COLORS.get(cat, colors.lightgrey)

    date_text = f"<b>{entry.get('date', '')}</b><br/>{entry.get('tach_tt', '')}"
    summary_text = entry.get("summary", "")
    signer_text = entry.get("signer", "")

    summary_block = [
        Paragraph(summary_text, styles["Body11"]),
        Paragraph(signer_text, styles["BodyItalic11"]) if signer_text else Spacer(1, 0)
    ]

    row = [
        Paragraph(date_text, styles["Small"]),
        category_pill(cat, styles),
        summary_block
    ]

    table = Table(
        [row],
        colWidths=[
            1.15 * inch,          # Date / tach
            0.45 * inch,          # Category pill
            USABLE_WIDTH - 1.60 * inch  # Wrapped summary
        ],
        splitByRow=True,
        repeatRows=0,
        hAlign="LEFT"
    )

    table.setStyle(TableStyle([
        ("VALIGN", (0, 0), (-1, -1), "TOP"),
        ("BACKGROUND", (0, 0), (-1, -1), CATEGORY_COLORS["GAP"] if cat == "GAP" else colors.white),
        ("BACKGROUND", (1, 0), (1, 0), bg),
        ("BOX", (0, 0), (-1, -1), 0.25, colors.HexColor("#CCCCCC")),
        ("INNERGRID", (0, 0), (-1, -1), 0.25, colors.HexColor("#DDDDDD")),
        ("LEFTPADDING", (0, 0), (-1, -1), 5),
        ("RIGHTPADDING", (0, 0), (-1, -1), 5),
        ("TOPPADDING", (0, 0), (-1, -1), 5),
        ("BOTTOMPADDING", (0, 0), (-1, -1), 5),
    ]))

    return table


def stats_table(rows, styles):
    data = [
        [
            Paragraph(f"<b>{label}</b>", styles["Body11"]),
            Paragraph(value, styles["Body11"])
        ]
        for label, value in rows
    ]

    table = Table(
        data,
        colWidths=[2.35 * inch, USABLE_WIDTH - 2.35 * inch],
        splitByRow=True,
        hAlign="LEFT"
    )

    table.setStyle(TableStyle([
        ("VALIGN", (0, 0), (-1, -1), "TOP"),
        ("GRID", (0, 0), (-1, -1), 0.25, colors.HexColor("#CCCCCC")),
        ("BACKGROUND", (0, 0), (0, -1), colors.HexColor("#F2F2F2")),
        ("LEFTPADDING", (0, 0), (-1, -1), 6),
        ("RIGHTPADDING", (0, 0), (-1, -1), 6),
        ("TOPPADDING", (0, 0), (-1, -1), 5),
        ("BOTTOMPADDING", (0, 0), (-1, -1), 5),
    ]))

    return table


def build_pdf(
    output_path,
    aircraft_id,
    source_short,
    cover_info,
    source_logs,
    chronology_by_year,
    recommendation,
    stats_rows,
    disclaimer,
):
    styles = build_styles()

    doc = SimpleDocTemplate(
        output_path,
        pagesize=PAGE_SIZE,
        rightMargin=MARGIN,
        leftMargin=MARGIN,
        topMargin=0.65 * inch,
        bottomMargin=0.65 * inch,
    )

    doc.aircraft_id = aircraft_id
    doc.source_short = source_short

    story = []

    # Cover page
    story.append(Paragraph(aircraft_id, ParagraphStyle(
        name="CoverTitle",
        fontName=SANS_BOLD,
        fontSize=24,
        leading=28,
        alignment=TA_CENTER,
        spaceAfter=18,
    )))

    story.append(Paragraph(cover_info, styles["Body11"]))
    story.append(Spacer(1, 12))

    story.append(Paragraph("Source Logbooks Consolidated", styles["Header"]))
    for src in source_logs:
        story.append(Paragraph(f"• {src}", styles["Body11"]))

    story.append(Spacer(1, 12))
    story.append(Paragraph("Category Legend", styles["Header"]))
    legend = ", ".join(CATEGORY_COLORS.keys())
    story.append(Paragraph(legend, styles["Body11"]))

    story.append(PageBreak())

    # About
    story.append(Paragraph("About This Document", styles["Header"]))
    story.append(Paragraph(
        "This document consolidates maintenance-log entries into a chronological reference record. "
        "Each entry includes the work date, tachometer and/or total-time reading when available, "
        "category, maintenance summary, and signing shop or mechanic when legible.",
        styles["Body11"]
    ))

    story.append(Paragraph(
        "Flagged gaps and missing or unavailable logbooks are noted where identified. "
        "This document is intended to support review and does not replace the original logbooks.",
        styles["Body11"]
    ))

    story.append(PageBreak())

    # Chronology
    story.append(Paragraph("Chronology", styles["Header"]))

    for year, entries in chronology_by_year.items():
        story.append(Paragraph(str(year), styles["YearHeader"]))

        for entry in entries:
            story.append(chronology_row(entry, styles))
            story.append(Spacer(1, 4))

    story.append(PageBreak())

    # Recommendation
    story.append(Paragraph("Preliminary Recommendation to the Board of Fox Flying Club", styles["Header"]))
    story.append(Paragraph(recommendation, styles["Body11"]))

    story.append(PageBreak())

    # Stats
    story.append(Paragraph("Appendix: Summary Statistics", styles["Header"]))
    story.append(stats_table(stats_rows, styles))

    story.append(PageBreak())

    # Disclaimer
    story.append(Paragraph("Disclaimer", styles["Header"]))
    story.append(Paragraph(disclaimer, styles["Body11"]))

    doc.build(story, onFirstPage=header_footer, onLaterPages=header_footer)
```
