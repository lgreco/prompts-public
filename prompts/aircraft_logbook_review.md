# Agent for airplane log reviews

## Preamble: Template Variables

| Variable | Value |
|----------|-------|
| `{{CLUB_NAME}}` | Fox Flying Club |
| `{{TAIL_NUMBER}}` | *(set at runtime)* |
| `{{YEAR}}` | *(set at runtime)* |
| `{{MAKE_MODEL}}` | *(set at runtime)* |
| `{{SERIAL_NUMBER}}` | *(set at runtime)* |

---

I'm preparing a pre-purchase maintenance review for aircraft **{{TAIL_NUMBER}}** (a **{{YEAR}} {{MAKE_MODEL}}**, S/N **{{SERIAL_NUMBER}}**). Attached are source documents for this review. Source files may be supplied in any of the following forms:

* **PDF files** — scanned logbook pages or typed records
* **Image files** — JPEG, PNG, TIFF, or similar scans
* **A ZIP archive** — a compressed collection of PDFs and/or image files; unpack it in memory, recursing into any nested folders, and treat every file found at any depth as a source document (do not confuse with the transcript cache archive described below)

Every file — or, if a ZIP archive is supplied, every file extracted from it — whose name begins with **{{TAIL_NUMBER}}** is a source document for this review. Most source documents are logbooks; some may be auxiliary documentation such as the sale listing of the aircraft.

The make, model, year, and serial number of the aircraft are mentioned in the attached logs. Infer the logbook type (airframe, engine, propeller, avionics, etc.) from contents; filenames may help but are not guaranteed. Also infer the time span covered by each logbook, by the contents of that logbook.

---

## Transcript Caching

Transcripts are plain-text renderings of source files, stored between runs in a zip archive so that previously processed files never need to be re-read visually.

**Transcript filename convention** — each transcript is named by prepending `text_transcript_of_` to the original filename, replacing the dot before the extension with an underscore, and appending `.txt`:

```
text_transcript_of_<original_filename>_<ext>.txt
```

For example: `text_transcript_of_N12345_airframe_log_pdf.txt`

**Archive name convention:**

```
{{TAIL_NUMBER}}_text_transcripts.zip
```

For example: `N12345_text_transcripts.zip`

**On each run:**

1. Check whether the user has attached a file named `{{TAIL_NUMBER}}_text_transcripts.zip`.
2. If the zip is present, unpack it in memory and load all transcripts it contains.
3. For each source file, check whether a matching transcript was found in the zip.
   - If yes: use that transcript — **do not re-process the source file visually**.
   - If no: visually read the source file and generate its transcript now, before moving to the next file.
4. After all source files have been processed, determine whether any new transcripts were generated this run.
   - If **no new transcripts**: proceed directly to the review — do not produce a zip.
   - If **new transcripts were generated**: assemble a new zip containing **all** transcripts (pre-existing ones from the user-supplied zip plus every newly generated one) and **return this zip to the user** at the end of the session, before the PDF. Label it clearly: `{{TAIL_NUMBER}}_text_transcripts.zip`. Instruct the user to save this file and attach it on the next run to avoid re-processing the same source files.

**Transcript content:** A faithful plain-text rendering of the entire source document — every legible entry, date, tach reading, maintenance description, and signer — with `[?]` markers for uncertain readings and explicit notes for missing, torn, or skipped pages.

### Transcription Rules

* For unclear handwriting: transcribe best effort and mark uncertainty with `[?]`
* If pages are missing/torn/skipped: explicitly note
* Do not silently infer or "clean up" ambiguous technical details

---

## Sale Listing as Trusted Source

If any attached file is identified as a **sale listing** (e.g., a broker listing, Trade-A-Plane entry, controller.com page, or similar market document):

* Use the sale listing as the **authoritative source** for:

  * **Total Time Airframe (TTAF)**
  * **Engine time since overhaul** and **overhaul type**:

    * **SFOH** — factory new or factory-remanufactured overhaul
    * **SMOH** — major overhaul performed by a repair station or A&P
    * **STOH** — top overhaul (cylinders only; not a full teardown)
    * **SFRM** — since factory remanufacture
    * Other designations (SNEW, SREM, etc.) as stated in the listing
  * **Avionics** — treat the listing's equipment description as the reference inventory for installed avionics
  * **Asking price** — record as stated in the listing

* When logbook entries **conflict** with the sale listing on any of the above, **defer to the sale listing** and flag the discrepancy: note it in the affected chronology row and summarize all conflicts on the Cover Page.

* If no sale listing is present, derive TTAF, engine time, overhaul type, and avionics from the logbooks and note that the logbooks are the sole source.

---

## Task

Using transcripts where available (see **Transcript Caching**) and visually reading only source files that lack a transcript, compile **every maintenance entry** into a single chronological record.

For each entry capture:

* Date of work
* Tach and/or total-time reading (if shown)
* Summary of work performed *(preserve part numbers, AD references, work orders)*
* Shop name and/or signing mechanic with certificate number
* Category tag from:
  **AF, ENG, PROP, AVX, INSP, GEAR, DOC, GAP**

---

## Category Tag Definitions

| Tag | Meaning | Notes |
|-----|---------|-------|
| **AF** | Airframe | Structural work, control surfaces, skin, doors, windows, and general airframe maintenance not covered by another category |
| **ENG** | Engine | All powerplant work: oil changes, cylinder work, magnetos, carburetor or fuel injection, exhaust, engine mounts |
| **PROP** | Propeller | Propeller inspections, overhauls, replacements, and strike reports |
| **AVX** | Avionics | Radios, navigation equipment, transponder, autopilot, ELT, and panel work |
| **INSP** | Inspection | Annual inspections, 100-hour inspections, progressive inspections, and pre-purchase inspections |
| **GEAR** | Landing gear | Struts, wheels, brakes, tires, and fairings on all aircraft. On retractable-gear aircraft (e.g., PA-28R Piper Arrow), also includes actuators, gear doors, hydraulics, and squat switches — and any gear-up incident. On fixed-gear aircraft (e.g., PA-28A Piper Archer, C172 Cessna Skyhawk), flag any entry suggesting a gear collapse or hard landing involving the gear. GEAR entries are always significant and must be noted in the Preliminary Recommendation. |
| **DOC** | Documentation | Ownership transfers, registration updates, weight-and-balance revisions, 337 forms, STC records, and other administrative entries |
| **GAP** | Coverage gap | Synthesized row — not a real logbook entry — marking a period of ≥13 months with no entries |

---

## Gap Detection

* Identify periods ≥13 months with no entries
* Insert a **GAP row**:

  * Show date span
  * Estimate hours flown by interpolating from the tach or total-time readings that immediately precede and follow the gap; if no bracketing readings are available, leave the estimate blank
* GAP rows must be visually distinct (see Styling)
* Do **not** interpret or editorialize about gaps

---

## Compression Test Data

* Scan all engine log entries for compression test results. These typically appear as per-cylinder readings in the form `72/80` (measured / reference) and are most often recorded during annual inspections or pre-purchase inspections.
* Collect every test found. For the Cover Page, present the **5 most recent tests in reverse chronological order**.
* Preserve the exact per-cylinder readings as recorded. If cylinder numbering is shown, include it (e.g., `C1: 72/80, C2: 74/80, …`). If only a string of values is given with no cylinder labels, transcribe them in the order recorded.
* If no compression data is found in the logs, note its absence on the Cover Page.

---

## Annual Inspection Status

* Identify the most recent logbook entry that is an **annual inspection** — look for phrases such as "annual inspection," "annual," "FAR 91.409," or equivalent. Do not confuse with 100-hour inspections or progressive inspections.
* An annual is valid for **12 calendar months** from the month it was performed. An annual completed in March 2024, for example, remains valid through March 31, 2025.
* Compute validity against today's date:

  * **Current** — note the expiration month and year in the Key Specifications block and the Cover Page.
  * **Lapsed** — flag as a **blocking issue** (see Preliminary Recommendation). State how many months ago it expired.
  * **Indeterminate** — if no clear annual can be identified in the logs, state that explicitly on the Cover Page and treat it as a blocking issue.

---

## Missing Logbooks

**Expected logbooks for any single-engine piston GA aircraft:**

* Airframe log — always required
* Engine log — always required
* Propeller log — always required

**Additional expected records for specific aircraft types:**

* Retractable-gear aircraft (e.g., PA-28R): gear and hydraulic system service history. If no dedicated gear log exists, check whether gear-related entries appear in the airframe log. Flag if absent.
* Aircraft with a factory-installed or STC'd autopilot or significant avionics suite: an avionics log is desirable. Its absence is notable but not blocking.

**If any expected log is absent:**

* State clearly on the **Cover Page**
* Repeat in **About This Document**
* Treat a missing airframe or engine log as a **blocking issue** (see Preliminary Recommendation)

---

## PDF Generation (ReportLab)

### Global Layout Constraints (MANDATORY)

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

## Typography (STRICT)

* **Body text:** 11 pt serif (e.g., Times-Roman)
* **Headers:** Helvetica (bold where appropriate)
* **Footers:** 9 pt serif
* Line spacing: 1.2–1.3
* Ensure consistent font usage throughout (no mixing defaults)

---

## Document Structure

### 1. Cover Page

Include:

* Tail number, aircraft info
* Asking price (if sale listing is present)
* **Key Specifications** block (table format, prominently placed):

  * TTAF — total time airframe (source: sale listing or logbook)
  * Engine time since overhaul and overhaul type (SMOH / SFOH / STOH / etc.)
  * Prop time (if available)
  * Most recent annual inspection date and whether it is currently valid
* **Compression Tests** — 5 most recent results in reverse chronological order (table format: date + per-cylinder readings). If none found, state explicitly.
* Coverage span (earliest → latest entry)
* Bullet list of source logs:

  * filename or inferred label
  * page count
  * handwritten vs typed
  * era covered
* Category legend

---

### 2. About This Document

* Paragraph 1: schema description
* Paragraph 2: note gaps and missing logs

---

### 3. Chronology

* Group entries by **year**
* Strict chronological order

Each entry rendered as a **row with controlled layout**:

* Left column (fixed width): Date + Tach/TT
* Middle: Category pill
* Right (flex, wrapped): Summary
* Mechanic/shop: italicized line below summary

#### Table Requirements

* Column widths must be explicitly defined and sum ≤ usable page width
* Summary column must wrap (never overflow)
* Rows must split cleanly across pages if needed
* Avoid excessively tall rows (split long summaries if needed)

---

### 4. Preliminary Recommendation

Short, direct:

* Should the club consider purchase?
* Identify risks and notable concerns
* No excessive narrative
* **Blocking issues** — conditions that by themselves warrant not proceeding without resolution — must be listed first and labeled **BLOCKING**. Blocking conditions include:

  * Lapsed or indeterminate annual inspection
  * Missing airframe or engine log
  * Any gear-up or gear-collapse incident (GEAR entries) without documented repair and return to service
  * Engine or prop with no documented overhaul history

---

### 5. Appendix: Summary Statistics

Two-column table:

* Asking price (from sale listing, if present)
* Total entries
* Coverage span
* Airframe TT (first/latest)
* Hours flown
* Entries per category
* GAP periods (count + list)
* Engine overhauls (with overhaul type: SFOH / SMOH / STOH / SFRM / etc., and source: sale listing or logbook)
* Prop replacements
* Tach resets
* Ownership/location indicators

---

### 6. Disclaimer

Standard non-authoritative transcription disclaimer

---

## Styling

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

## Final QA Pass (REQUIRED)

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

## Output Requirement

Use the ReportLab template below.

Return outputs in this order:

1. **Transcript zip** (`{{TAIL_NUMBER}}_text_transcripts.zip`) — only if new transcripts were generated this run (see **Transcript Caching**). Instruct the user to save it and attach it on the next run.
2. **Final PDF** — well-formatted, print-ready. Do not include intermediate artifacts.

---

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
            0.60 * inch,          # Category pill (wide enough for "PROP" at 8pt with padding)
            USABLE_WIDTH - 1.75 * inch  # Wrapped summary
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
    key_specs,            # list of (label, value) pairs for Key Specifications block
    compression_tests,    # list of (date, readings_string) pairs, newest first; may be empty
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

    if key_specs:
        story.append(Paragraph("Key Specifications", styles["Header"]))
        story.append(stats_table(key_specs, styles))
        story.append(Spacer(1, 12))

    story.append(Paragraph("Compression Tests", styles["Header"]))
    if compression_tests:
        story.append(stats_table(compression_tests, styles))
    else:
        story.append(Paragraph("No compression test data found in logs.", styles["Body11"]))
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
    story.append(Paragraph("Preliminary Recommendation to the Board of {{CLUB_NAME}}", styles["Header"]))
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
