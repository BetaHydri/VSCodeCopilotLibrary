---
name: pdf-to-markdown
description: >-
  Convert PDF files to well-structured Markdown using .NET-native PDF parsing
  in PowerShell — no external tools (Python, pdftotext, Word COM) required.
  Decompresses zlib/deflate content streams, decodes hex-encoded text operators
  (BT/ET, Td, Tj), reconstructs lines by Y-coordinate positioning, and produces
  clean Markdown with tables and formatting. Handles German-locale PDFs
  (ISO-8859-1 encoding, umlauts, ß, special chars). Best suited for structured
  documents like payslips, invoices, reports.
  USE FOR: convert PDF to markdown, PDF to md, extract text from PDF, parse PDF,
  PDF text extraction, PDF to text, Entgeltabrechnung PDF, payslip PDF, invoice
  PDF, read PDF in PowerShell, decode PDF hex, PDF content stream, deflate PDF
  stream, structured PDF extraction, German PDF, Gehaltsabrechnung.
  DO NOT USE FOR: scanned/image-based PDFs (use OCR tools), PDFs with complex
  vector graphics, PDF form filling, PDF editing, PDF creation.
---

# PDF to Markdown Conversion

## When to Use

- Extracting text from PDF files in a workspace for downstream analysis
- Converting structured documents (payslips, invoices, reports, letters) to Markdown
- No Python, pdftotext, or external PDF tools available on the system
- PDF contains text-based content (not scanned images)

## Approach Decision Tree

```
PDF file in workspace
├── Is Python + pymupdf/pdfplumber available?
│   └── YES → Use Python (simplest, most reliable)
├── Is pdftotext (xpdf/poppler) available?
│   └── YES → Use pdftotext (fast, preserves layout)
├── Is the PDF small (<100KB) and text-based?
│   └── YES → Use .NET native parsing (Recipe 1 below)
└── Otherwise
    └── Ask user to paste text or install a tool
```

### Check Tool Availability

```powershell
# Check in priority order
$tools = @(
    @{ Name = 'pdftotext'; Check = { Get-Command pdftotext -EA 0 } }
    @{ Name = 'python+pymupdf'; Check = { python -c "import pymupdf" 2>$null; $LASTEXITCODE -eq 0 } }
    @{ Name = 'python+pdfplumber'; Check = { python -c "import pdfplumber" 2>$null; $LASTEXITCODE -eq 0 } }
)
foreach ($t in $tools) { if (& $t.Check) { Write-Host "Use: $($t.Name)"; break } }
```

## Recipe 1: .NET Native PDF Parsing (No External Tools)

This approach works for text-based PDFs using standard PDF operators. It uses
only .NET classes available in PowerShell 7+ — no NuGet packages needed.

### How PDF Text Storage Works

PDF files store text in **content streams** that are typically zlib-compressed.
Each stream contains PostScript-like operators:

| Operator | Meaning | Example |
|----------|---------|---------|
| `BT` / `ET` | Begin/End text block | |
| `Td` | Set text position (x, y) | `148.80 811.20 Td` |
| `Tf` | Set font and size | `/F001 8.00 Tf` |
| `Tj` | Show text string | `<48656C6C6F>Tj` = "Hello" |
| `TJ` | Show text array (with kerning) | `[(He) -10 (llo)]TJ` |
| `Tw` | Set word spacing | `0 Tw` |

Text strings in `<...>Tj` are **hex-encoded** — each pair of hex digits is one byte.

### Step 1: Extract Content Streams

```powershell
$pdfPath = 'C:\path\to\file.pdf'
$bytes = [System.IO.File]::ReadAllBytes($pdfPath)
$raw = [System.Text.Encoding]::GetEncoding('ISO-8859-1').GetString($bytes)

# Find all stream/endstream blocks
$streamMatches = [regex]::Matches(
    $raw,
    'stream\r?\n(.+?)\r?\nendstream',
    [System.Text.RegularExpressions.RegexOptions]::Singleline
)
Write-Host "Found $($streamMatches.Count) streams"
```

### Step 2: Decompress Streams (zlib/deflate)

Most PDF streams use FlateDecode (zlib). The first 2 bytes are the zlib header —
skip them to get raw deflate data for .NET's `DeflateStream`:

```powershell
foreach ($i in 0..($streamMatches.Count - 1)) {
    $streamData = [System.Text.Encoding]::GetEncoding('ISO-8859-1').GetBytes(
        $streamMatches[$i].Groups[1].Value
    )
    try {
        # Skip 2-byte zlib header for raw deflate
        $ms = [System.IO.MemoryStream]::new($streamData, 2, ($streamData.Length - 2))
        $ds = [System.IO.Compression.DeflateStream]::new(
            $ms, [System.IO.Compression.CompressionMode]::Decompress
        )
        $sr = [System.IO.StreamReader]::new(
            $ds, [System.Text.Encoding]::GetEncoding('ISO-8859-1')
        )
        $text = $sr.ReadToEnd()
        $sr.Close()
        Write-Host "Stream $i : $($text.Length) chars"
        # The stream with BT/ET text operators is the content stream
        if ($text -match 'BT\r?\n') {
            $contentStream = $text
            Write-Host "  -> This is the text content stream"
        }
    } catch {
        Write-Host "Stream $i : not deflate-compressed"
    }
}
```

### Step 3: Identify the Content Stream

Among the decoded streams, look for the one containing `BT` (Begin Text) operators.
Other streams are typically images, fonts, or metadata:

- **Stream with repeated `0xFF` bytes** → image data (e.g., bitmap mask)
- **Stream with color/coordinate data** → graphics/drawing commands
- **Stream with `BT`/`ET` + `Td` + `Tj`** → **this is the text content stream**

### Step 4: Parse Text Positions and Decode Hex

```powershell
# Pattern: BT block with position (Td) and hex text (Tj)
$pattern = 'BT\r?\n\s*([\d.]+)\s+([\d.]+)\s+Td\r?\n\s*\d+\s+Tw\r?\n\s*<([0-9A-Fa-f]+)>Tj\r?\n\s*ET'
$blocks = [regex]::Matches($contentStream, $pattern)

# Build a map of Y-position -> list of (X, decoded-text)
$lineMap = @{}
foreach ($b in $blocks) {
    $x = [double]$b.Groups[1].Value
    $y = $b.Groups[2].Value  # Keep as STRING to avoid locale decimal issues
    $hex = $b.Groups[3].Value

    # Decode hex pairs to characters
    $decoded = -join (for ($j = 0; $j -lt $hex.Length; $j += 2) {
        [char][Convert]::ToInt32($hex.Substring($j, 2), 16)
    })

    if (-not $lineMap.ContainsKey($y)) {
        $lineMap[$y] = [System.Collections.Generic.List[PSCustomObject]]::new()
    }
    $lineMap[$y].Add([PSCustomObject]@{ X = $x; Text = $decoded })
}
```

### Step 5: Reconstruct Lines (Y-Coordinate Ordering)

PDF Y-coordinates increase upward — highest Y = top of page:

```powershell
$sortedYs = $lineMap.Keys | Sort-Object { [double]$_ } -Descending
$textLines = foreach ($yKey in $sortedYs) {
    $parts = $lineMap[$yKey] | Sort-Object X
    ($parts | ForEach-Object { $_.Text }) -join ''
}
```

### Step 6: Convert to Markdown

Once you have the text lines, analyze the structure and format as Markdown:

- **Separator lines** (all `_` or `-`) → use `---` or table borders
- **Header lines** (larger font, centered) → `#` / `##` headings
- **Columnar data** → Markdown tables (`| col1 | col2 |`)
- **Labeled values** (e.g., `Name: John`) → key-value tables

## Pitfalls and Lessons Learned

### Terminal Blocking
**CRITICAL**: Never use Word COM (`New-Object -ComObject Word.Application`) for
PDF conversion in VS Code terminals. Word COM:
- Blocks the terminal thread while processing
- Hangs on `Documents.Open()` for PDFs (conversion dialog)
- Leaves orphaned `WINWORD.EXE` processes
- Is unreliable even when run in detached processes

Use the .NET native approach instead.

### Locale/Culture Decimal Separator
When using `[double]` values as hashtable keys, use the **raw string** from the
PDF (e.g., `"820.20"`) — not `.ToString()` which may produce `"820,20"` on
German-locale systems, causing all entries to collide into one key.

### PDF Encoding
German PDFs typically use **ISO-8859-1** (Latin-1) encoding for text. This
correctly handles `ü` (`FC`), `ö` (`F6`), `ä` (`E4`), `ß` (`DF`), `§` (`A7`).
Do **not** assume UTF-8 for PDF content streams.

### Hex vs Parenthesis Text
PDFs encode text strings in two forms:
- `<hex>Tj` — hex-encoded (most common in generated PDFs)
- `(text)Tj` — literal parenthesis-encoded (also common)
Check for both patterns when parsing.

### Multi-Page PDFs
Each page may have its own content stream. The page objects in the PDF
cross-reference table link to their content streams. For multi-page documents,
iterate all streams that contain `BT` operators.

### When This Approach Fails
- **Scanned PDFs**: No text operators — need OCR (Tesseract, Azure AI)
- **CIDFont/ToUnicode mapping**: Some PDFs use glyph IDs instead of character
  codes — requires parsing the font's ToUnicode CMap table
- **Complex layouts**: Multi-column, overlapping text, rotated text
- **Encrypted PDFs**: Content streams are encrypted — need decryption first

## Recipe 2: pdftotext (If Available)

The fastest option when xpdf/poppler tools are installed:

```powershell
# Install via chocolatey (requires admin)
choco install xpdf-utils -y

# Or via winget (search for available package)
winget search pdftotext

# Extract with layout preservation
pdftotext -layout input.pdf output.txt

# Then convert the text file to Markdown manually or with formatting logic
```

## Recipe 3: Python pymupdf (If Available)

```powershell
# Install
pip install pymupdf

# Extract
python -c "
import pymupdf
doc = pymupdf.open('input.pdf')
for page in doc:
    print(page.get_text())
"
```
