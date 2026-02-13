# OOXML Common Structure Reference

## Overview

Office Open XML (OOXML) is the file format used by Microsoft Office documents (.docx, .pptx, .xlsx). All OOXML files share a common ZIP-based structure containing XML files that define the document's content, formatting, and metadata.

## ZIP Archive Structure

Every OOXML file is a ZIP archive with a standardized directory structure:

```
document.{docx|pptx|xlsx}/
├── [Content_Types].xml          # MIME types for all parts
├── _rels/
│   └── .rels                    # Relationships for package-level items
├── docProps/                    # Document properties
│   ├── app.xml                  # Application-specific properties
│   └── core.xml                 # Core metadata (author, title, dates)
└── {word|ppt|xl}/               # Format-specific content
    ├── document.xml             # Main content (Word)
    ├── presentation.xml         # Main content (PowerPoint)
    ├── workbook.xml             # Main content (Excel)
    ├── _rels/                   # Relationships
    │   └── document.xml.rels
    ├── theme/                   # Theme definitions
    ├── media/                   # Embedded images and media
    └── ...                      # Format-specific files
```

### Key Files

| File | Purpose |
|------|---------|
| `[Content_Types].xml` | Maps file extensions and parts to MIME types |
| `_rels/.rels` | Defines relationships between package parts |
| `{word\|ppt\|xl}/_rels/document.xml.rels` | Relationships for main content (images, hyperlinks, etc.) |
| `{word\|ppt\|xl}/document.xml` | Main content file |

## XML Namespaces

OOXML uses multiple XML namespaces. Understanding these is essential for editing XML content.

### Common Namespaces

| Prefix | Namespace URI | Purpose |
|--------|---------------|---------|
| `w:` | `http://schemas.openxmlformats.org/wordprocessingml/2006/main` | Word processing (docx) |
| `a:` | `http://schemas.openxmlformats.org/drawingml/2006/main` | DrawingML (graphics, shapes) |
| `r:` | `http://schemas.openxmlformats.org/officeDocument/2006/relationships` | Relationships |
| `p:` | `http://schemas.openxmlformats.org/presentationml/2006/main` | PresentationML (pptx) |
| `x:` | `http://schemas.openxmlformats.org/spreadsheetml/2006/main` | SpreadsheetML (xlsx) |
| `mc:` | `http://schemas.openxmlformats.org/markup-compatibility/2006` | Markup compatibility |
| `wp:` | `http://schemas.openxmlformats.org/drawingml/2006/wordprocessingDrawing` | Word drawings |
| `pic:` | `http://schemas.openxmlformats.org/drawingml/2006/picture` | Pictures |

### Namespace Declaration Example

```xml
<w:document
    xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main"
    xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships"
    xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main"
    xmlns:wp="http://schemas.openxmlformats.org/drawingml/2006/wordprocessingDrawing"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006">
    <!-- Content -->
</w:document>
```

## Relationships (_rels)

Relationships connect parts of the document. Every relationship has:
- **Id**: Unique identifier (e.g., `rId5`)
- **Type**: URL defining the relationship type
- **Target**: Path to the related file

### Relationship Types

| Type | URI Fragment | Purpose |
|------|-------------|---------|
| Image | `/image` | Embedded images |
| Hyperlink | `/hyperlink` | External links |
| Header | `/header` | Document headers |
| Footer | `/footer` | Document footers |
| Theme | `/theme` | Color themes |
| Styles | `/styles` | Style definitions |

### Example: Image Relationship

```xml
<!-- word/_rels/document.xml.rels -->
<Relationship
    Id="rId5"
    Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/image"
    Target="media/image1.png"/>
```

Referenced in document.xml:
```xml
<a:blip r:embed="rId5"/>
```

## Content Types

`[Content_Types].xml` maps file extensions and specific parts to MIME types.

### Common Content Types

| Extension/Part | Content Type |
|----------------|-------------|
| `.xml` | `application/xml` |
| `.rels` | `application/vnd.openxmlformats-package.relationships+xml` |
| `.png` | `image/png` |
| `.jpeg`, `.jpg` | `image/jpeg` |

### Example Content Types File

```xml
<Types xmlns="http://schemas.openxmlformats.org/package/2006/content-types">
  <!-- Default types by extension -->
  <Default Extension="rels" ContentType="application/vnd.openxmlformats-package.relationships+xml"/>
  <Default Extension="xml" ContentType="application/xml"/>
  <Default Extension="png" ContentType="image/png"/>
  <Default Extension="jpeg" ContentType="image/jpeg"/>

  <!-- Override types for specific parts -->
  <Override PartName="/word/document.xml"
            ContentType="application/vnd.openxmlformats-officedocument.wordprocessingml.document.main+xml"/>
  <Override PartName="/word/styles.xml"
            ContentType="application/vnd.openxmlformats-officedocument.wordprocessingml.styles+xml"/>
</Types>
```

## Format Preservation Principles

When editing OOXML files, follow these principles to avoid corruption:

### 1. Preserve Existing Attributes

Always retain attributes you don't understand. Example:

```xml
<!-- WRONG - removes unknown attributes -->
<w:r><w:t>text</w:t></w:r>

<!-- CORRECT - preserves all attributes -->
<w:r w:rsidR="00AB1234" w:rsidRPr="00CD5678">
  <w:rPr><w:b/></w:rPr>
  <w:t>text</w:t>
</w:r>
```

### 2. Maintain Element Order

XML Schema defines strict element ordering. For example, in `<w:pPr>`:

```xml
<!-- CORRECT order -->
<w:pPr>
  <w:pStyle w:val="Heading1"/>
  <w:numPr>...</w:numPr>
  <w:spacing w:before="240"/>
  <w:ind w:left="720"/>
  <w:jc w:val="left"/>
  <w:rPr>...</w:rPr>  <!-- Always last -->
</w:pPr>

<!-- WRONG - rPr before jc -->
<w:pPr>
  <w:pStyle w:val="Heading1"/>
  <w:rPr>...</w:rPr>
  <w:jc w:val="left"/>
</w:pPr>
```

### 3. Preserve Unknown Elements

If you encounter elements you don't recognize, keep them:

```xml
<!-- Keep mc:AlternateContent blocks -->
<mc:AlternateContent>
  <mc:Choice Requires="w14">
    <w14:textFill>...</w14:textFill>
  </mc:Choice>
  <mc:Fallback>
    <w:color w:val="000000"/>
  </mc:Fallback>
</mc:AlternateContent>
```

### 4. Handle Whitespace Correctly

Use `xml:space="preserve"` for text with leading/trailing spaces:

```xml
<!-- WRONG - spaces lost -->
<w:t>  indented text  </w:t>

<!-- CORRECT -->
<w:t xml:space="preserve">  indented text  </w:t>
```

## Common XML Editing Patterns

### Adding Content

When adding new content, copy the structure of existing elements:

```xml
<!-- Existing paragraph -->
<w:p>
  <w:pPr><w:pStyle w:val="Normal"/></w:pPr>
  <w:r><w:t>Existing text</w:t></w:r>
</w:p>

<!-- Add new paragraph - copy structure -->
<w:p>
  <w:pPr><w:pStyle w:val="Normal"/></w:pPr>
  <w:r><w:t>New text</w:t></w:r>
</w:p>
```

### Modifying Formatting

Preserve the `<w:rPr>` (run properties) or `<w:pPr>` (paragraph properties) block:

```xml
<!-- Original run with bold formatting -->
<w:r>
  <w:rPr><w:b/><w:sz w:val="24"/></w:rPr>
  <w:t>Old text</w:t>
</w:r>

<!-- Modified run - preserve formatting -->
<w:r>
  <w:rPr><w:b/><w:sz w:val="24"/></w:rPr>
  <w:t>New text</w:t>
</w:r>
```

### Replacing Text

Use exact string replacement, preserving XML structure:

```xml
<!-- Replace text content only -->
<w:t>Old value: 30</w:t>
<!-- becomes -->
<w:t>New value: 60</w:t>
```

## RSIDs (Revision Save IDs)

RSIDs track edits in Word documents. They're 8-digit hex strings:

```xml
<w:r w:rsidR="00AB1234" w:rsidRPr="00CD5678">
  <w:t>text</w:t>
</w:r>
```

- `rsidR`: Revision ID for the run
- `rsidRPr`: Revision ID for run properties
- `rsidP`: Revision ID for paragraph
- `rsidRDefault`: Default revision ID

**Best practice**: Preserve existing RSIDs; use a consistent new RSID for all new content (e.g., `00000001`).

## Smart Quotes and Special Characters

Use XML entities for smart quotes and special characters:

| Character | Entity | Description |
|-----------|--------|-------------|
| ' | `&#x2018;` | Left single quote |
| ' | `&#x2019;` | Right single quote / apostrophe |
| " | `&#x201C;` | Left double quote |
| " | `&#x201D;` | Right double quote |
| — | `&#x2014;` | Em dash |
| – | `&#x2013;` | En dash |
| & | `&amp;` | Ampersand |
| < | `&lt;` | Less than |
| > | `&gt;` | Greater than |

Example:
```xml
<w:t>Here&#x2019;s a quote: &#x201C;Hello&#x201D;</w:t>
```

## Validation and Schema Compliance

### Common Validation Errors

1. **durableId >= 0x7FFFFFFF**: Comment/revision IDs too large
   - **Fix**: Regenerate with valid 32-bit signed integer

2. **Missing xml:space="preserve"**: Text with whitespace
   - **Fix**: Add `xml:space="preserve"` attribute

3. **Invalid element order**: Elements in wrong sequence
   - **Fix**: Reorder elements per schema

4. **Malformed XML**: Unclosed tags, invalid nesting
   - **Fix**: Repair XML structure manually

### Auto-Repair vs Manual Fix

**Auto-repairable**:
- durableId regeneration
- xml:space attribute addition

**Requires manual fix**:
- Malformed XML
- Invalid element nesting
- Missing relationships
- Schema violations (wrong element order)

## Tools and Workflows

### Unpack → Edit → Pack Workflow

1. **Unpack**: Extract ZIP contents and pretty-print XML
   ```bash
   python scripts/office/unpack.py document.docx unpacked/
   ```

2. **Edit**: Modify XML files with text editor or scripts

3. **Pack**: Validate, condense, and create new file
   ```bash
   python scripts/office/pack.py unpacked/ output.docx --original document.docx
   ```

### Validation

Always validate after packing:
```bash
python scripts/office/validate.py output.docx
```

## Best Practices Summary

1. **Always preserve**:
   - Unknown attributes
   - Unknown elements
   - Existing RSIDs
   - Element order

2. **Always validate**:
   - After packing
   - Before distributing

3. **Use smart quotes**:
   - XML entities, not Unicode characters

4. **Handle whitespace**:
   - `xml:space="preserve"` for leading/trailing spaces

5. **Copy structure**:
   - When adding new content, copy existing patterns

6. **Minimal edits**:
   - Only change what's necessary
   - Replace smallest possible units

7. **Test thoroughly**:
   - Open in target application
   - Verify formatting preserved
   - Check for corruption warnings
