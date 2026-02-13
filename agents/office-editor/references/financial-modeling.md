# Financial Modeling Standards for Excel

## Overview

Financial models require strict formatting, color coding, and formula construction standards to ensure clarity, auditability, and error-free operation. This guide defines industry-standard practices for Excel-based financial models.

## Color Coding Standards

Color coding distinguishes input cells from calculations, making models easier to audit and reducing errors.

### Standard Color Scheme

| Element | Color | RGB | Purpose |
|---------|-------|-----|---------|
| **Hardcoded inputs** | Blue | `0, 0, 255` | Numbers users enter or change for scenarios |
| **Formulas** | Black | `0, 0, 0` | All calculations and references within sheet |
| **Cross-sheet links** | Green | `0, 128, 0` | References to other worksheets in same workbook |
| **External links** | Red | `255, 0, 0` | Links to other files (warning: may break) |
| **Key assumptions** | Yellow background | `255, 255, 0` | Critical cells needing attention or update |

### Color Coding in openpyxl

```python
from openpyxl.styles import Font, PatternFill

# Blue text for hardcoded inputs
blue_font = Font(color='0000FF')
sheet['B5'].font = blue_font
sheet['B5'].value = 0.05  # Growth rate assumption

# Black text for formulas
black_font = Font(color='000000')
sheet['C5'].font = black_font
sheet['C5'].value = '=B5*(1+$B$6)'  # Calculated value

# Green text for cross-sheet references
green_font = Font(color='008000')
sheet['D5'].font = green_font
sheet['D5'].value = '=Summary!B10'  # Link to Summary sheet

# Red text for external links
red_font = Font(color='FF0000')
sheet['E5'].font = red_font
sheet['E5'].value = "='[OtherWorkbook.xlsx]Sheet1'!A1"  # External file

# Yellow background for key assumptions
yellow_fill = PatternFill(start_color='FFFF00', end_color='FFFF00', fill_type='solid')
sheet['B6'].fill = yellow_fill
sheet['B6'].value = 0.10  # Critical assumption
```

### When NOT to Use Color Coding

Do not override color coding when:
- User specifies custom template colors
- Existing template has established color scheme
- Template conventions explicitly differ

**Template preservation rule**: Existing template conventions ALWAYS override these standards.

## Number Formatting Standards

### Currency and Numbers

#### Format Rules

| Type | Format Code | Example | Notes |
|------|-------------|---------|-------|
| Currency | `$#,##0` | $1,234 | No decimals unless required |
| Currency with decimals | `$#,##0.00` | $1,234.56 | Use when precision matters |
| Currency (millions) | `$#,##0` | $5 | Specify units in header: "Revenue ($mm)" |
| Negative currency | `$#,##0;($#,##0);-` | ($1,234) | Parentheses for negatives |
| Zeros as dash | `$#,##0;($#,##0);-` | - | Third section controls zero display |
| Percentage | `0.0%` | 15.2% | One decimal default |
| Multiples | `0.0x` | 12.5x | For valuation multiples (EV/EBITDA, P/E) |
| Thousands separator | `#,##0` | 1,234 | Always use for readability |

#### Years as Text

**Critical**: Format years as text strings, not numbers:

```python
# WRONG - formats as number with comma
sheet['B1'].value = 2024
sheet['B1'].number_format = '#,##0'  # Shows as "2,024"

# CORRECT - text string
sheet['B1'].value = '2024'  # Shows as "2024"
```

#### Zero Display

Make all zeros display as dash `-` for cleaner appearance:

```python
# Currency with zeros as dash
sheet['B5'].number_format = '$#,##0;($#,##0);-'

# Percentage with zeros as dash
sheet['C5'].number_format = '0.0%;(0.0%);-'
```

### Units in Headers

Always specify units in column/row headers, never in cell formatting:

```python
# CORRECT
sheet['A1'].value = 'Revenue ($mm)'
sheet['B1'].value = 2024
sheet['B2'].value = 150  # Represents $150 million

# WRONG - don't put "mm" in each cell
sheet['B2'].value = '150mm'  # Loses numeric properties
```

### Number Formatting in openpyxl

```python
from openpyxl.styles import numbers

# Currency (no decimals, zeros as dash)
sheet['B5'].number_format = '$#,##0;($#,##0);-'

# Currency with decimals
sheet['C5'].number_format = '$#,##0.00;($#,##0.00);-'

# Percentage (one decimal)
sheet['D5'].number_format = '0.0%'

# Multiples (for ratios like P/E, EV/EBITDA)
sheet['E5'].number_format = '0.0x'

# Thousands separator
sheet['F5'].number_format = '#,##0'
```

## Formula Construction Rules

### No Hardcoded Constants

**Never embed constants in formulas**. Always reference assumption cells.

```python
# WRONG - hardcoded growth rate
sheet['C5'].value = '=B5*1.05'  # What is 1.05?

# CORRECT - reference assumption
sheet['B6'].value = 0.05
sheet['B6'].font = Font(color='0000FF')  # Blue for input
sheet['C5'].value = '=B5*(1+$B$6)'  # Clear reference
```

### Named Ranges

Use named ranges for key assumptions:

```python
from openpyxl.workbook.defined_name import DefinedName

# Define named range for growth rate
wb.defined_names['GrowthRate'] = DefinedName('GrowthRate', attr_text='Assumptions!$B$6')

# Use in formula
sheet['C5'].value = '=B5*(1+GrowthRate)'
```

### Absolute vs Relative References

| Reference Type | Syntax | Behavior | Use Case |
|---------------|--------|----------|----------|
| Relative | `A1` | Changes when copied | Row/column patterns |
| Absolute | `$A$1` | Never changes | Fixed assumptions |
| Mixed (column) | `$A1` | Column fixed, row varies | Lookup tables |
| Mixed (row) | `A$1` | Row fixed, column varies | Time series headers |

Example: Time series calculation referencing fixed assumption

```python
# Assumption in B6
sheet['B6'].value = 0.05
sheet['B6'].font = Font(color='0000FF')

# Formulas in time series (columns C, D, E, ...)
sheet['C10'].value = '=B10*(1+$B$6)'  # B6 is absolute
# When copied to D10: =C10*(1+$B$6)  ← B6 stays fixed
```

## Formula Error Prevention

### Pre-Deployment Checklist

Before delivering financial model:

- [ ] **Zero formula errors**: Use `scripts/recalc.py` to check for #REF!, #DIV/0!, #VALUE!, #N/A, #NAME?
- [ ] **Verify cell references**: Check all references point to intended cells
- [ ] **Test edge cases**: Zero values, negative numbers, large numbers
- [ ] **Check ranges**: Verify no off-by-one errors (e.g., SUM(A1:A9) vs SUM(A1:A10))
- [ ] **Circular references**: Ensure no unintended circular dependencies
- [ ] **External links**: Verify external file paths are correct or remove

### Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `#REF!` | Invalid cell reference (deleted row/column) | Update reference to valid cell |
| `#DIV/0!` | Division by zero | Add error handling: `=IFERROR(A1/B1, 0)` |
| `#VALUE!` | Wrong data type in formula | Check cell contains number, not text |
| `#N/A` | VLOOKUP/MATCH not found | Use `IFERROR` or verify lookup range |
| `#NAME?` | Unrecognized function/name | Check spelling, verify named range exists |

### Division by Zero Protection

```python
# Wrap division formulas with IFERROR
sheet['C5'].value = '=IFERROR(A5/B5, 0)'

# Or check denominator explicitly
sheet['C5'].value = '=IF(B5<>0, A5/B5, 0)'
```

## Documentation Requirements

### Hardcoded Value Documentation

Every hardcoded input MUST be documented with source:

```python
from openpyxl.comments import Comment

# Add source documentation as comment
sheet['B5'].value = 125.7
sheet['B5'].font = Font(color='0000FF')  # Blue for input
sheet['B5'].comment = Comment(
    'Source: Company 10-K, FY2024, Page 45, Revenue Note, https://sec.gov/...',
    'Author'
)
```

### Documentation Format

```
Source: [System/Document], [Date], [Specific Reference], [URL if applicable]
```

Examples:
- `Source: Company 10-K, FY2024, Page 45, Revenue Note, [SEC EDGAR URL]`
- `Source: Company 10-Q, Q2 2025, Exhibit 99.1, [SEC EDGAR URL]`
- `Source: Bloomberg Terminal, 8/15/2025, AAPL US Equity`
- `Source: FactSet, 8/20/2025, Consensus Estimates Screen`
- `Source: Management interview, 8/25/2025, CFO call notes`

### Section Headers

Use section headers to organize model:

```python
# Section header formatting
header_font = Font(bold=True, size=14, color='FFFFFF')
header_fill = PatternFill(start_color='4472C4', end_color='4472C4', fill_type='solid')

sheet['A1'].value = 'ASSUMPTIONS'
sheet['A1'].font = header_font
sheet['A1'].fill = header_fill
```

## Cell Layout Best Practices

### Inputs and Assumptions Section

Group all assumptions at top of sheet or on dedicated "Assumptions" sheet:

```python
# Assumptions section
sheet['A1'].value = 'ASSUMPTIONS'
sheet['A3'].value = 'Revenue Growth Rate'
sheet['B3'].value = 0.05
sheet['B3'].font = Font(color='0000FF')

sheet['A4'].value = 'Operating Margin'
sheet['B4'].value = 0.20
sheet['B4'].font = Font(color='0000FF')

# Calculations section below
sheet['A6'].value = 'INCOME STATEMENT'
# ... calculations reference B3, B4, etc.
```

### Time Series Layout

For projection models, use consistent time series structure:

```
         | 2024 | 2025 | 2026 | 2027 | 2028 |
---------|------|------|------|------|------|
Revenue  | 100  | =B10*(1+GrowthRate) | =C10*(1+GrowthRate) | ...
COGS     | 60   | =B11*COGSRatio | =C11*COGSRatio | ...
```

### Vertical Alignment

Align related items vertically for easy comparison:

```python
# Income statement vertical layout
sheet['A10'].value = 'Revenue'
sheet['A11'].value = 'COGS'
sheet['A12'].value = 'Gross Profit'
sheet['A13'].value = 'Operating Expenses'
sheet['A14'].value = 'EBIT'
```

## Summary

### Key Principles

1. **Color code systematically**: Blue inputs, black formulas, green cross-sheet, red external
2. **No hardcoded values in formulas**: Reference assumption cells
3. **Format zeros as dash**: Use `#,##0;(#,##0);-` format
4. **Document all inputs**: Add comments with source and date
5. **Use named ranges**: For key assumptions
6. **Specify units in headers**: Not in cell formatting
7. **Test for errors**: Zero formula errors before delivery
8. **Preserve templates**: Existing conventions override these standards

### Quick Reference

| Task | Code Example |
|------|--------------|
| Blue input | `sheet['B5'].font = Font(color='0000FF')` |
| Currency format | `sheet['B5'].number_format = '$#,##0;($#,##0);-'` |
| Percentage format | `sheet['C5'].number_format = '0.0%'` |
| Zero as dash | Third section of format: `...; -` |
| Named range | `wb.defined_names['Rate'] = DefinedName(...)` |
| Comment documentation | `sheet['B5'].comment = Comment('Source: ...')` |
| Error prevention | `'=IFERROR(A5/B5, 0)'` |
