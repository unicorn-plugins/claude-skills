# PDF Processing API Reference

## Overview

This document provides detailed API references for Python and JavaScript PDF processing libraries. For basic operations and quick start guides, see the main SKILL.md file.

## Python Libraries

### pypdf - Basic Operations

#### Core Classes

```python
from pypdf import PdfReader, PdfWriter
```

##### PdfReader

**Constructor:**
```python
reader = PdfReader(filename)  # From file path
reader = PdfReader(file_object)  # From file-like object
```

**Properties:**
```python
len(reader.pages)  # Number of pages
reader.metadata  # DocumentInformation object
reader.is_encrypted  # Boolean
```

**Methods:**
```python
reader.decrypt(password)  # Decrypt encrypted PDF
reader.pages[n]  # Access page by index (0-based)
```

##### PdfWriter

**Constructor:**
```python
writer = PdfWriter()
```

**Methods:**
```python
writer.add_page(page)  # Add PageObject
writer.append(reader)  # Append all pages from reader
writer.append(reader, pages=[0, 1, 2])  # Append specific pages
writer.insert_page(page, index)  # Insert at specific position
writer.write(output_file)  # Write to file or stream
writer.encrypt(user_password, owner_password=None)  # Add password
```

##### PageObject

**Properties:**
```python
page.mediabox  # Page dimensions (RectangleObject)
page.rotation  # Current rotation (0, 90, 180, 270)
```

**Methods:**
```python
page.extract_text()  # Extract text as string
page.rotate(angle)  # Rotate by angle (90, 180, 270)
page.merge_page(page2)  # Overlay another page
page.scale(sx, sy)  # Scale page
page.scale_to(width, height)  # Scale to dimensions
```

#### Complete Examples

**Merge PDFs:**
```python
from pypdf import PdfWriter, PdfReader

writer = PdfWriter()
for pdf_file in ["doc1.pdf", "doc2.pdf", "doc3.pdf"]:
    reader = PdfReader(pdf_file)
    for page in reader.pages:
        writer.add_page(page)

with open("merged.pdf", "wb") as output:
    writer.write(output)
```

**Split PDF:**
```python
reader = PdfReader("input.pdf")
for i, page in enumerate(reader.pages):
    writer = PdfWriter()
    writer.add_page(page)
    with open(f"page_{i+1}.pdf", "wb") as output:
        writer.write(output)
```

**Extract and Modify Metadata:**
```python
reader = PdfReader("document.pdf")
writer = PdfWriter()

for page in reader.pages:
    writer.add_page(page)

# Copy existing metadata and update
metadata = reader.metadata
writer.add_metadata({
    "/Title": "Updated Title",
    "/Author": "New Author",
    "/Subject": metadata.get("/Subject", ""),
})

with open("updated.pdf", "wb") as output:
    writer.write(output)
```

**Password Protection:**
```python
reader = PdfReader("input.pdf")
writer = PdfWriter()

for page in reader.pages:
    writer.add_page(page)

# User password: required to open
# Owner password: required for editing
writer.encrypt(user_password="user123", owner_password="owner456")

with open("encrypted.pdf", "wb") as output:
    writer.write(output)
```

**Crop Pages:**
```python
reader = PdfReader("input.pdf")
writer = PdfWriter()

page = reader.pages[0]
# Coordinates in points (left, bottom, right, top)
page.mediabox.left = 50
page.mediabox.bottom = 50
page.mediabox.right = 550
page.mediabox.top = 750

writer.add_page(page)
with open("cropped.pdf", "wb") as output:
    writer.write(output)
```

---

### pdfplumber - Text and Table Extraction

#### Core API

```python
import pdfplumber

with pdfplumber.open("document.pdf") as pdf:
    # Access pages
    page = pdf.pages[0]  # First page
    for page in pdf.pages:  # Iterate all pages
        ...
```

#### Page Object

**Properties:**
```python
page.width  # Page width in points
page.height  # Page height in points
page.bbox  # Bounding box (x0, top, x1, bottom)
page.chars  # List of character objects
page.lines  # List of line objects
page.rects  # List of rectangle objects
```

**Methods:**
```python
page.extract_text()  # Extract plain text
page.extract_text(layout=True)  # Preserve layout
page.extract_text(x_tolerance=3, y_tolerance=3)  # Adjust spacing
page.extract_tables()  # Extract all tables as lists
page.extract_table()  # Extract first table
page.crop(bbox)  # Crop to bounding box
page.within_bbox(bbox)  # Filter to bbox
page.to_image(resolution=150)  # Convert to image
```

#### Character Objects

Each character dict contains:
```python
{
    "text": "A",  # Character
    "x0": 100.0,  # Left edge
    "y0": 200.0,  # Top edge
    "x1": 110.0,  # Right edge
    "y1": 212.0,  # Bottom edge
    "width": 10.0,
    "height": 12.0,
    "fontname": "Times-Roman",
    "size": 12.0
}
```

#### Table Extraction

**Basic extraction:**
```python
with pdfplumber.open("document.pdf") as pdf:
    page = pdf.pages[0]
    tables = page.extract_tables()
    for table in tables:
        for row in table:
            print(row)  # List of cell values
```

**Custom table settings:**
```python
table_settings = {
    "vertical_strategy": "lines",  # or "text", "explicit"
    "horizontal_strategy": "lines",  # or "text", "explicit"
    "snap_tolerance": 3,  # Snap close lines together
    "join_tolerance": 3,  # Join nearly-touching lines
    "edge_min_length": 10,  # Minimum line length
    "intersection_tolerance": 15,  # Join intersections
}

tables = page.extract_tables(table_settings)
```

**Convert to pandas DataFrame:**
```python
import pandas as pd

with pdfplumber.open("document.pdf") as pdf:
    page = pdf.pages[0]
    table = page.extract_table()
    if table:
        df = pd.DataFrame(table[1:], columns=table[0])
        df.to_excel("output.xlsx", index=False)
```

#### Visual Debugging

```python
with pdfplumber.open("document.pdf") as pdf:
    page = pdf.pages[0]

    # Create image with annotations
    img = page.to_image(resolution=150)
    img.draw_rects(page.rects)  # Draw rectangles
    img.draw_lines(page.lines)  # Draw lines
    img.save("debug.png")
```

#### Bounding Box Operations

```python
# Extract text from specific region (x0, top, x1, bottom)
bbox = (100, 100, 400, 200)
cropped = page.within_bbox(bbox)
text = cropped.extract_text()

# Extract all characters in region
chars_in_bbox = [char for char in page.chars
                 if bbox[0] <= char['x0'] and char['x1'] <= bbox[2]
                 and bbox[1] <= char['y0'] and char['y1'] <= bbox[3]]
```

---

### reportlab - Create PDFs

#### Canvas API (Low-Level)

```python
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas

c = canvas.Canvas("output.pdf", pagesize=letter)
width, height = letter  # 612 x 792 points (8.5" x 11")

# Drawing methods
c.drawString(x, y, "text")  # Simple text
c.drawCentredString(x, y, "text")  # Centered text
c.drawRightString(x, y, "text")  # Right-aligned text

c.line(x1, y1, x2, y2)  # Line
c.rect(x, y, width, height)  # Rectangle outline
c.rect(x, y, width, height, fill=1)  # Filled rectangle
c.circle(x, y, radius)  # Circle outline
c.circle(x, y, radius, fill=1)  # Filled circle

c.setFont("Helvetica", 12)  # Font and size
c.setFillColorRGB(r, g, b)  # Fill color (0-1)
c.setStrokeColorRGB(r, g, b)  # Stroke color (0-1)

c.showPage()  # Start new page
c.save()  # Write file
```

**Complete Canvas Example:**
```python
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas

c = canvas.Canvas("invoice.pdf", pagesize=letter)
width, height = letter

# Header
c.setFont("Helvetica-Bold", 18)
c.drawString(50, height - 50, "INVOICE")

# Company info
c.setFont("Helvetica", 10)
c.drawString(50, height - 80, "Acme Corporation")
c.drawString(50, height - 95, "123 Main St")
c.drawString(50, height - 110, "City, ST 12345")

# Invoice details
c.drawString(400, height - 80, "Invoice #: 12345")
c.drawString(400, height - 95, "Date: 2025-01-15")

# Line items table
y_position = height - 150
c.setFont("Helvetica-Bold", 10)
c.drawString(50, y_position, "Item")
c.drawString(300, y_position, "Qty")
c.drawString(400, y_position, "Price")
c.drawString(500, y_position, "Total")

c.setFont("Helvetica", 10)
y_position -= 20
c.drawString(50, y_position, "Widget")
c.drawString(300, y_position, "2")
c.drawString(400, y_position, "$50.00")
c.drawString(500, y_position, "$100.00")

# Total
y_position -= 40
c.setFont("Helvetica-Bold", 12)
c.drawString(400, y_position, "Total:")
c.drawString(500, y_position, "$100.00")

c.save()
```

#### Platypus API (High-Level)

```python
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle, PageBreak
from reportlab.lib.styles import getSampleStyleSheet
from reportlab.lib import colors
from reportlab.lib.pagesizes import letter

doc = SimpleDocTemplate("report.pdf", pagesize=letter)
styles = getSampleStyleSheet()
story = []

# Add elements to story
story.append(Paragraph("Title", styles['Title']))
story.append(Spacer(1, 12))  # Vertical space (width, height)
story.append(Paragraph("Body text", styles['Normal']))
story.append(PageBreak())  # New page

# Build PDF
doc.build(story)
```

**Table Example:**
```python
data = [
    ['Product', 'Q1', 'Q2', 'Q3', 'Q4'],
    ['Widgets', '120', '135', '142', '158'],
    ['Gadgets', '85', '92', '98', '105']
]

table = Table(data)
table.setStyle(TableStyle([
    ('BACKGROUND', (0, 0), (-1, 0), colors.grey),  # Header background
    ('TEXTCOLOR', (0, 0), (-1, 0), colors.whitesmoke),  # Header text
    ('ALIGN', (0, 0), (-1, -1), 'CENTER'),  # Center all cells
    ('FONTNAME', (0, 0), (-1, 0), 'Helvetica-Bold'),  # Header font
    ('FONTSIZE', (0, 0), (-1, 0), 14),
    ('BOTTOMPADDING', (0, 0), (-1, 0), 12),
    ('BACKGROUND', (0, 1), (-1, -1), colors.beige),  # Data background
    ('GRID', (0, 0), (-1, -1), 1, colors.black)  # Grid lines
]))

story.append(table)
```

**Important**: Never use Unicode subscript/superscript characters (₀₁₂, ⁰¹²). Use XML markup instead:

```python
# CORRECT
chemical = Paragraph("H<sub>2</sub>O", styles['Normal'])
equation = Paragraph("x<super>2</super> + y<super>2</super>", styles['Normal'])

# WRONG - renders as black boxes
water = Paragraph("H₂O", styles['Normal'])  # ❌
```

---

### pypdfium2 - Rendering and Images

```python
import pypdfium2 as pdfium
from PIL import Image

# Load PDF
pdf = pdfium.PdfDocument("document.pdf")

# Render page to image
page = pdf[0]  # First page
bitmap = page.render(
    scale=2.0,  # Resolution multiplier
    rotation=0  # 0, 90, 180, 270
)

# Convert to PIL Image
img = bitmap.to_pil()
img.save("page_1.png", "PNG")

# Process multiple pages
for i, page in enumerate(pdf):
    bitmap = page.render(scale=1.5)
    img = bitmap.to_pil()
    img.save(f"page_{i+1}.jpg", "JPEG", quality=90)

# Extract text
text = page.get_text()
```

---

## Command-Line Tools

### pdftotext (poppler-utils)

**Basic text extraction:**
```bash
pdftotext input.pdf output.txt
```

**Options:**
```bash
pdftotext -layout input.pdf output.txt  # Preserve layout
pdftotext -f 1 -l 5 input.pdf output.txt  # Pages 1-5
pdftotext -bbox-layout input.pdf output.xml  # With bounding boxes
```

**Output formats:**
- Plain text (default)
- `-layout`: Maintain physical layout
- `-bbox-layout`: XML with coordinates

---

### qpdf

**Merge PDFs:**
```bash
qpdf --empty --pages file1.pdf file2.pdf -- merged.pdf
```

**Split PDFs:**
```bash
# Extract specific pages
qpdf input.pdf --pages . 1-5 -- pages1-5.pdf
qpdf input.pdf --pages . 6-10 -- pages6-10.pdf

# Split every N pages
qpdf --split-pages=3 input.pdf output_group_%02d.pdf
```

**Merge specific pages from multiple PDFs:**
```bash
qpdf --empty --pages doc1.pdf 1-3 doc2.pdf 5-7 doc3.pdf 2,4 -- combined.pdf
```

**Rotate pages:**
```bash
qpdf input.pdf output.pdf --rotate=+90:1  # Rotate page 1 by 90°
qpdf input.pdf output.pdf --rotate=+180:2-4  # Rotate pages 2-4 by 180°
```

**Encryption:**
```bash
# Add password
qpdf --encrypt user_pass owner_pass 256 --print=none --modify=none -- input.pdf encrypted.pdf

# Check encryption
qpdf --show-encryption encrypted.pdf

# Remove password (requires password)
qpdf --password=secret123 --decrypt encrypted.pdf decrypted.pdf
```

**Optimization:**
```bash
# Linearize for web (fast page display)
qpdf --linearize input.pdf optimized.pdf

# Remove unused objects and compress
qpdf --optimize-level=all input.pdf compressed.pdf
```

**Repair:**
```bash
# Check PDF structure
qpdf --check input.pdf

# Attempt repair
qpdf --fix-qdf damaged.pdf repaired.pdf

# Show structure for debugging
qpdf --show-all-pages input.pdf > structure.txt
```

---

### pdfimages (poppler-utils)

**Extract all images:**
```bash
pdfimages -j input.pdf output_prefix
```

**Options:**
```bash
pdfimages -all input.pdf images/img  # All formats
pdfimages -list input.pdf  # List without extracting
pdfimages -j -p input.pdf page_images  # Include page numbers
```

---

### pdftoppm (poppler-utils)

**Convert to images:**
```bash
# PNG output
pdftoppm -png -r 300 input.pdf output_prefix

# JPEG output with quality
pdftoppm -jpeg -jpegopt quality=85 -r 200 input.pdf output

# Specific page range
pdftoppm -png -r 600 -f 1 -l 3 input.pdf high_res_pages
```

**Options:**
- `-r DPI`: Resolution (default 150)
- `-f N`: First page
- `-l N`: Last page
- `-png | -jpeg | -tiff`: Output format
- `-jpegopt quality=N`: JPEG quality (0-100)

---

## JavaScript Libraries

### pdf-lib (MIT License)

**Load and modify PDF:**
```javascript
import { PDFDocument } from 'pdf-lib';
import fs from 'fs';

async function modifyPDF() {
    const existingPdfBytes = fs.readFileSync('input.pdf');
    const pdfDoc = await PDFDocument.load(existingPdfBytes);

    // Get page count
    const pageCount = pdfDoc.getPageCount();

    // Add new page
    const newPage = pdfDoc.addPage([600, 400]);
    newPage.drawText('Added by pdf-lib', { x: 100, y: 300, size: 16 });

    const pdfBytes = await pdfDoc.save();
    fs.writeFileSync('modified.pdf', pdfBytes);
}
```

**Create PDF from scratch:**
```javascript
import { PDFDocument, rgb, StandardFonts } from 'pdf-lib';

async function createPDF() {
    const pdfDoc = await PDFDocument.create();
    const helveticaFont = await pdfDoc.embedFont(StandardFonts.Helvetica);
    const page = pdfDoc.addPage([595, 842]);  // A4
    const { width, height } = page.getSize();

    // Add text
    page.drawText('Invoice #12345', {
        x: 50,
        y: height - 50,
        size: 18,
        font: helveticaFont,
        color: rgb(0.2, 0.2, 0.8)
    });

    // Add rectangle
    page.drawRectangle({
        x: 40,
        y: height - 100,
        width: width - 80,
        height: 30,
        color: rgb(0.9, 0.9, 0.9)
    });

    const pdfBytes = await pdfDoc.save();
    fs.writeFileSync('created.pdf', pdfBytes);
}
```

**Merge PDFs:**
```javascript
async function mergePDFs() {
    const mergedPdf = await PDFDocument.create();

    const pdf1Bytes = fs.readFileSync('doc1.pdf');
    const pdf2Bytes = fs.readFileSync('doc2.pdf');

    const pdf1 = await PDFDocument.load(pdf1Bytes);
    const pdf2 = await PDFDocument.load(pdf2Bytes);

    // Copy all pages from first PDF
    const pdf1Pages = await mergedPdf.copyPages(pdf1, pdf1.getPageIndices());
    pdf1Pages.forEach(page => mergedPdf.addPage(page));

    // Copy specific pages from second PDF
    const pdf2Pages = await mergedPdf.copyPages(pdf2, [0, 2, 4]);
    pdf2Pages.forEach(page => mergedPdf.addPage(page));

    const mergedPdfBytes = await mergedPdf.save();
    fs.writeFileSync('merged.pdf', mergedPdfBytes);
}
```

---

## Performance Tips

### Large PDF Processing

**Memory-efficient page splitting:**
```python
from pypdf import PdfReader, PdfWriter

def process_large_pdf(pdf_path, chunk_size=10):
    reader = PdfReader(pdf_path)
    total_pages = len(reader.pages)

    for start_idx in range(0, total_pages, chunk_size):
        end_idx = min(start_idx + chunk_size, total_pages)
        writer = PdfWriter()

        for i in range(start_idx, end_idx):
            writer.add_page(reader.pages[i])

        with open(f"chunk_{start_idx//chunk_size}.pdf", "wb") as output:
            writer.write(output)
```

### Tool Selection

| Task | Fastest Tool |
|------|-------------|
| Text extraction | `pdftotext` (command-line) |
| Table extraction | `pdfplumber` (Python) |
| Image extraction | `pdfimages` (command-line) |
| Page rendering | `pypdfium2` (Python) |
| Merge/split | `qpdf` (command-line) |
| Create from scratch | `reportlab` (Python) or `pdf-lib` (JS) |

### Resolution Guidelines

| Use Case | DPI |
|----------|-----|
| Screen preview | 72-96 |
| Web display | 150 |
| Printing | 300 |
| High-quality printing | 600 |

---

## Troubleshooting

### Encrypted PDFs

```python
from pypdf import PdfReader

reader = PdfReader("encrypted.pdf")
if reader.is_encrypted:
    reader.decrypt("password")

# Now can access pages
text = reader.pages[0].extract_text()
```

### Corrupted PDFs

```bash
# Repair with qpdf
qpdf --check corrupted.pdf
qpdf --replace-input corrupted.pdf
```

### OCR for Scanned PDFs

```python
import pytesseract
from pdf2image import convert_from_path

images = convert_from_path('scanned.pdf')
text = ""
for image in images:
    text += pytesseract.image_to_string(image)
```

---

## License Information

| Library | License |
|---------|---------|
| pypdf | BSD |
| pdfplumber | MIT |
| pypdfium2 | Apache/BSD |
| reportlab | BSD |
| poppler-utils | GPL-2 |
| qpdf | Apache |
| pdf-lib | MIT |
