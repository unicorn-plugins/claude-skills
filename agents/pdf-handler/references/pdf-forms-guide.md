# PDF Form Filling Strategy Guide

## Overview

Filling PDF forms requires choosing the right strategy based on the PDF's structure. This guide explains how to determine the best approach and execute each workflow.

## Strategy Decision Tree

```
START
  |
  ├─> Check for fillable fields
  |     |
  |     ├─> Has fillable fields? → Use Fillable Fields Strategy
  |     └─> No fillable fields? → Use Annotation Overlay Strategy
```

## Step 0: Determine Strategy

**CRITICAL: Run this check first.**

```bash
python scripts/check_fillable_fields.py <file.pdf>
```

**Output:**
- **"PDF has fillable form fields"** → Use Fillable Fields Strategy (Section 1)
- **"PDF has no fillable form fields"** → Use Annotation Overlay Strategy (Section 2)

---

# Section 1: Fillable Fields Strategy

## When to Use

PDF contains fillable form fields (interactive PDF form created with Adobe Acrobat or similar).

**Advantages:**
- Form structure already defined
- Field types enforced (text, checkbox, dropdown)
- Simpler implementation

**Disadvantages:**
- Limited to predefined fields
- Cannot add fields to non-form PDFs

## Workflow

### 1.1: Extract Form Field Information

```bash
python scripts/extract_form_field_info.py <input.pdf> <field_info.json>
```

**Output format (field_info.json):**
```json
[
  {
    "field_id": "last_name",
    "page": 1,
    "rect": [100, 600, 300, 620],
    "type": "text"
  },
  {
    "field_id": "Checkbox12",
    "page": 1,
    "type": "checkbox",
    "checked_value": "/On",
    "unchecked_value": "/Off"
  },
  {
    "field_id": "country_select",
    "page": 1,
    "type": "radio_group",
    "radio_options": [
      {"value": "/USA", "rect": [150, 500, 165, 515]},
      {"value": "/Canada", "rect": [150, 480, 165, 495]}
    ]
  },
  {
    "field_id": "state_dropdown",
    "page": 1,
    "type": "choice",
    "choice_options": [
      {"value": "CA", "text": "California"},
      {"value": "NY", "text": "New York"}
    ]
  }
]
```

**Field types:**
- `text`: Single-line or multi-line text input
- `checkbox`: Boolean checkbox
- `radio_group`: Radio button group (one selection)
- `choice`: Dropdown or listbox

### 1.2: Visual Analysis

Convert PDF to images to identify field purposes:

```bash
python scripts/convert_pdf_to_images.py <file.pdf> <output_directory>
```

Analyze images to determine what each field represents. Use bounding boxes from `field_info.json` to locate fields visually.

**Coordinate system:** PDF coordinates where y=0 is at bottom, y increases upward.

### 1.3: Create Field Values File

Create `field_values.json` with values to fill:

```json
[
  {
    "field_id": "last_name",
    "description": "User's last name",
    "page": 1,
    "value": "Simpson"
  },
  {
    "field_id": "Checkbox12",
    "description": "Check if 18 or over",
    "page": 1,
    "value": "/On"
  },
  {
    "field_id": "country_select",
    "description": "Country of residence",
    "page": 1,
    "value": "/USA"
  }
]
```

**Important:**
- `field_id` must match exactly from `field_info.json`
- For checkboxes: use `checked_value` to check, `unchecked_value` to uncheck
- For radio groups: use one of the `value` entries from `radio_options`
- For choice fields: use one of the `value` entries from `choice_options`

### 1.4: Fill Form

```bash
python scripts/fill_fillable_fields.py <input.pdf> <field_values.json> <output.pdf>
```

**Validation:** This script automatically verifies:
- Field IDs exist in the PDF
- Values are valid for field type
- Pages match

If errors occur, correct `field_values.json` and retry.

### 1.5: Verify Output

Convert filled PDF to images and verify:

```bash
python scripts/convert_pdf_to_images.py <output.pdf> <verify_images/>
```

---

# Section 2: Annotation Overlay Strategy

## When to Use

PDF has no fillable fields (scanned document or static PDF form).

**Advantages:**
- Works on any PDF
- Can add text anywhere
- Flexible positioning

**Disadvantages:**
- Manual coordinate mapping required
- More complex workflow

## Approach Selection

Choose one of three approaches based on PDF structure:

| Approach | When to Use | Accuracy |
|----------|-------------|----------|
| **A: Structure-Based** | Text labels extractable from PDF | Highest |
| **B: Visual Estimation** | Scanned/image-based PDF | Lower (requires zoom refinement) |
| **C: Hybrid** | Mix of extractable and missing elements | Medium |

### Determining Approach

Run structure extraction:

```bash
python scripts/extract_form_structure.py <input.pdf> form_structure.json
```

**Check results:**
- Labels present and meaningful → Use Approach A
- No labels or gibberish text → Use Approach B
- Some labels, some missing → Use Approach C

---

## Approach A: Structure-Based Coordinates

### A.1: Analyze Extracted Structure

Review `form_structure.json`:

```json
{
  "labels": [
    {"text": "Last", "x0": 43, "top": 63, "x1": 65, "bottom": 73},
    {"text": "Name", "x0": 68, "top": 63, "x1": 87, "bottom": 73}
  ],
  "lines": [
    {"x0": 90, "top": 79, "x1": 260, "bottom": 79}
  ],
  "checkboxes": [
    {"x0": 285, "top": 197, "x1": 292, "bottom": 205}
  ],
  "row_boundaries": [63, 79, 95, ...]
}
```

**Coordinate system:** PDF coordinates where y=0 is at TOP of page, y increases downward.

### A.2: Identify Missing Elements

Structure extraction may not detect:
- Circular checkboxes (only square rectangles detected)
- Faded or light-colored elements
- Complex graphics

For missing fields, use visual analysis (see Approach C: Hybrid).

### A.3: Calculate Entry Coordinates

**Text fields:**
```
entry_x0 = label_x1 + gap (typically 5 points)
entry_x1 = next_label_x0 or row_boundary
entry_top = label_top
entry_bottom = next_row_boundary or (label_bottom + row_height)
```

**Checkboxes:**
```
Use checkbox rectangle directly from form_structure.json
entry_bounding_box = [checkbox.x0, checkbox.top, checkbox.x1, checkbox.bottom]
```

### A.4: Create fields.json

Use `pdf_width` and `pdf_height` (signals PDF coordinates):

```json
{
  "pages": [
    {"page_number": 1, "pdf_width": 612, "pdf_height": 792}
  ],
  "form_fields": [
    {
      "page_number": 1,
      "description": "Last name entry field",
      "field_label": "Last Name",
      "label_bounding_box": [43, 63, 87, 73],
      "entry_bounding_box": [92, 63, 260, 79],
      "entry_text": {"text": "Smith", "font_size": 10}
    },
    {
      "page_number": 1,
      "description": "US Citizen checkbox",
      "field_label": "Yes",
      "label_bounding_box": [260, 200, 280, 210],
      "entry_bounding_box": [285, 197, 292, 205],
      "entry_text": {"text": "X"}
    }
  ]
}
```

---

## Approach B: Visual Estimation

### B.1: Convert to Images

```bash
python scripts/convert_pdf_to_images.py <input.pdf> <images_dir/>
```

### B.2: Initial Identification

Examine page images and note rough field positions (pixel coordinates). Precision not required yet.

### B.3: Zoom Refinement (CRITICAL)

For each field, create zoomed crop to determine precise coordinates.

**Create crop with ImageMagick:**
```bash
magick <page_image> -crop <width>x<height>+<x>+<y> +repage <crop_output.png>
```

**Parameters:**
- `x, y`: Top-left corner of crop (rough estimate minus padding)
- `width, height`: Size of crop region (field area + ~50px padding each side)

**Example:** Refine "Name" field estimated at (100, 150):
```bash
magick images_dir/page_1.png -crop 300x80+50+120 +repage crops/name_field.png
```

**Fallback:** If `magick` unavailable, try `convert` with same arguments.

**Examine cropped image** and identify exact pixel boundaries:
1. Entry area start (after label)
2. Entry area end (before next field)
3. Top and bottom edges

**Convert back to full image coordinates:**
```
full_x = crop_x + crop_offset_x
full_y = crop_y + crop_offset_y
```

Example: If crop started at (50, 120) and entry box starts at (52, 18) in crop:
```
entry_x0 = 52 + 50 = 102
entry_top = 18 + 120 = 138
```

### B.4: Create fields.json

Use `image_width` and `image_height` (signals image coordinates):

```json
{
  "pages": [
    {"page_number": 1, "image_width": 1700, "image_height": 2200}
  ],
  "form_fields": [
    {
      "page_number": 1,
      "description": "Last name entry",
      "field_label": "Last Name",
      "label_bounding_box": [120, 175, 242, 198],
      "entry_bounding_box": [255, 175, 720, 218],
      "entry_text": {"text": "Smith", "font_size": 10}
    }
  ]
}
```

---

## Approach C: Hybrid (Structure + Visual)

### C.1: Use Structure for Detected Fields

Extract structure and use PDF coordinates for all detected fields (see Approach A).

### C.2: Visual Analysis for Missing Fields

Convert to images and use zoom refinement for missing fields (see Approach B.3).

### C.3: Coordinate Conversion

Convert visually-estimated image coordinates to PDF coordinates:

```
pdf_x = image_x * (pdf_width / image_width)
pdf_y = image_y * (pdf_height / image_height)
```

Example: Image (1700x2200), PDF (612x792), entry at (850, 440):
```
pdf_x = 850 * (612 / 1700) = 306
pdf_y = 440 * (792 / 2200) = 158
```

### C.4: Create Unified fields.json

Use single coordinate system (`pdf_width`/`pdf_height`) with all coordinates in PDF points:

```json
{
  "pages": [
    {"page_number": 1, "pdf_width": 612, "pdf_height": 792}
  ],
  "form_fields": [
    {
      "description": "From structure extraction",
      "entry_bounding_box": [92, 63, 260, 79]
    },
    {
      "description": "From visual estimation (converted to PDF coords)",
      "entry_bounding_box": [306, 158, 450, 175]
    }
  ]
}
```

---

## Common Workflow Steps (All Approaches)

### Validate Bounding Boxes

**Always validate before filling:**

```bash
python scripts/check_bounding_boxes.py fields.json
```

**Checks for:**
- Intersecting bounding boxes (overlapping text)
- Entry boxes too small for font size

Fix any errors in `fields.json` before proceeding.

### Fill Form

```bash
python scripts/fill_pdf_form_with_annotations.py <input.pdf> fields.json <output.pdf>
```

**Auto-detection:** Script detects coordinate system from `pdf_width`/`pdf_height` vs `image_width`/`image_height` and converts automatically.

### Verify Output

```bash
python scripts/convert_pdf_to_images.py <output.pdf> <verify_images/>
```

**If text mispositioned:**
- Approach A: Verify using PDF coordinates from `form_structure.json`
- Approach B: Check image dimensions match and pixel coordinates accurate
- Approach C: Verify coordinate conversions correct

---

## Coordinate System Reference

### PDF Coordinates (Approach A, C)

```
Origin: Top-left corner
X-axis: Left to right (0 to pdf_width)
Y-axis: Top to bottom (0 to pdf_height)

Standard page sizes:
- US Letter: 612 x 792 points
- A4: 595 x 842 points

1 point = 1/72 inch
```

### Image Coordinates (Approach B)

```
Origin: Top-left corner
X-axis: Left to right (0 to image_width pixels)
Y-axis: Top to bottom (0 to image_height pixels)

Conversion to PDF:
pdf_x = image_x * (pdf_width / image_width)
pdf_y = image_y * (pdf_height / image_height)
```

---

## QA Checklist

### Before Filling

- [ ] Strategy determined (fillable vs annotation)
- [ ] Coordinates extracted or estimated
- [ ] Bounding boxes validated
- [ ] Font sizes appropriate for entry boxes
- [ ] No overlapping fields

### After Filling

- [ ] Output PDF opens without errors
- [ ] All text visible and positioned correctly
- [ ] Checkboxes marked correctly
- [ ] No text overflow or cutoff
- [ ] Text alignment correct (left/center/right)

---

## Troubleshooting

### Text Not Appearing

**Causes:**
- Entry box too small for font size
- Coordinates outside page boundaries
- Text color same as background

**Solutions:**
- Increase entry box dimensions
- Verify coordinates within page bounds
- Check text color in fields.json

### Text Overlapping

**Causes:**
- Intersecting bounding boxes
- Incorrect coordinate system

**Solutions:**
- Run `check_bounding_boxes.py`
- Verify using correct coordinate system (PDF vs image)

### Checkboxes Not Marking

**Causes:**
- Incorrect checkbox coordinates
- Font size too large for checkbox

**Solutions:**
- Verify checkbox bounding box accurate
- Use font_size 10-12 for "X" marks
- Ensure entry_bounding_box matches checkbox size

---

## Custom Tool Pipeline (8 Tools)

For complex form filling workflows, use the custom tool pipeline:

| Step | Tool | Purpose |
|------|------|---------|
| 1 | `check_fillable_fields` | Determine strategy |
| 2a | `extract_form_field_info` | Extract fillable field structure |
| 2b | `extract_form_structure` | Extract PDF structure (text/lines/boxes) |
| 3 | `convert_pdf_to_images` | Visual analysis |
| 4 | `check_bounding_boxes` | Validate coordinates |
| 5a | `fill_fillable_fields` | Fill fillable forms |
| 5b | `fill_pdf_form_with_annotations` | Fill via annotations |
| 6 | `convert_pdf_to_images` | Verify output |

**Typical pipelines:**

**Fillable fields:**
```
check_fillable_fields → extract_form_field_info → convert_pdf_to_images →
fill_fillable_fields → convert_pdf_to_images (verify)
```

**Annotation overlay (structure-based):**
```
check_fillable_fields → extract_form_structure → check_bounding_boxes →
fill_pdf_form_with_annotations → convert_pdf_to_images (verify)
```

**Annotation overlay (visual):**
```
check_fillable_fields → convert_pdf_to_images → [manual zoom crops] →
check_bounding_boxes → fill_pdf_form_with_annotations →
convert_pdf_to_images (verify)
```

---

## Summary

### Decision Matrix

| PDF Type | Strategy | Tools |
|----------|----------|-------|
| Has fillable fields | Fillable Fields | extract_form_field_info, fill_fillable_fields |
| Text-based, no fields | Annotation (Structure) | extract_form_structure, fill_pdf_form_with_annotations |
| Scanned/image-based | Annotation (Visual) | convert_pdf_to_images, fill_pdf_form_with_annotations |
| Mixed | Annotation (Hybrid) | Both structure and visual tools |

### Key Principles

1. **Always check first**: Use `check_fillable_fields` to determine strategy
2. **Validate before filling**: Use `check_bounding_boxes` to catch errors early
3. **Verify output**: Always convert output to images for visual inspection
4. **Use appropriate coordinates**: PDF coords for structure-based, image coords for visual (with conversion)
5. **Zoom for precision**: Visual estimation requires zoomed crops for accuracy
6. **Document sources**: Note where coordinates came from (structure extraction vs manual estimation)
