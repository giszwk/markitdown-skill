---
name: markitdown-skill
description: >
  Convert various document formats (PDF, Word, Excel, PowerPoint, HTML, images, audio, EPUB, ZIP, CSV, JSON, XML, YouTube) to Markdown for LLM consumption using the markitdown Python library. Trigger whenever the user mentions converting, transforming, or extracting content from any document file into Markdown format. Also trigger when user says things like "turn this PDF into md", "extract this docx as markdown", "convert this file for AI processing", "make this readable for an LLM", or similar file-to-markdown conversion requests. Do NOT use for docx creation/editing (use docx skill instead) or for basic text file operations.
---

# Markitdown Converter

Convert any supported document format to clean Markdown using Microsoft's [markitdown](https://github.com/microsoft/markitdown) library. Designed for feeding documents into LLM pipelines and text analysis.

## Use This Skill

Use this skill when the user wants a file, URL, or supported document bundle converted into Markdown for reading, review, search, or LLM processing.

Do not use this skill for:
- Editing or creating `.docx` files; use a Word/document skill instead.
- High-fidelity visual conversion where layout must match the original.
- Plain text file reads where no conversion is needed.

## Safety and Scope

MarkItDown reads local files and remote resources with the privileges of the current process. Treat user-provided paths, ZIP files, and URLs as untrusted input unless the user confirms the source.

- Prefer explicit local file paths supplied by the user.
- Avoid broad recursive conversion unless the user asks for a folder or archive.
- For URLs, confirm that network access is appropriate and avoid authenticated/private resources unless the user requested them.
- For ZIP files, inspect or list contents before converting when the archive source is untrusted.
- Write Markdown output next to the source file or to the user-specified destination. Do not overwrite existing files unless the user asks for replacement.

## Supported Formats

| Format | Extension | Notes |
|--------|-----------|-------|
| PDF | `.pdf` | Text extraction + optional OCR via plugin |
| Word | `.docx` | Headings, lists, tables, links preserved |
| PowerPoint | `.pptx` | Slide content as markdown |
| Excel | `.xlsx`, `.xls` | Sheet data as tables |
| HTML | `.html`, `.htm` | Strips styling, keeps structure |
| Images | `.jpg`, `.png`, etc. | EXIF metadata + optional LLM image description |
| Audio | `.mp3`, `.wav` | EXIF metadata + speech transcription |
| EPUB | `.epub` | E-book to markdown |
| CSV/JSON/XML | `.csv`, `.json`, `.xml` | Structured data as markdown |
| YouTube | URL | Fetch transcript via YouTube API |
| ZIP | `.zip` | Iterates over contained files |

## Quick Start

### Install markitdown

**Requirements:** Python 3.10+

Prefer installing in the project virtual environment to avoid dependency conflicts:

```bash
python -m venv .venv
source .venv/bin/activate
pip install 'markitdown[all]'
```

该命令会安装所有可选依赖。若只需部分格式：
```bash
pip install 'markitdown[pdf, docx, pptx]'      # PDF + Word + PowerPoint
pip install 'markitdown[pdf]'                   # PDF only
pip install 'markitdown[audio-transcription]'   # Audio transcription
pip install 'markitdown[youtube-transcription]' # YouTube transcripts
```

**可选依赖说明：**
- `[all]` — 全部格式
- `[pptx]` — PowerPoint
- `[docx]` — Word
- `[xlsx]` / `[xls]` — Excel
- `[pdf]` — PDF
- `[outlook]` — Outlook 邮件
- `[az-doc-intel]` — Azure Document Intelligence（提升 PDF 精度）
- `[audio-transcription]` — 音频转写（wav/mp3）
- `[youtube-transcription]` — YouTube 字幕获取

### CLI Usage

Convert a file and output to stdout:
```bash
markitdown path/to/file.pdf > output.md
```

Convert and save to file:
```bash
markitdown path/to/file.pdf -o output.md
```

Pipe content:
```bash
cat path/to/file.pdf | markitdown
```

### Python API

Basic conversion:
```python
from markitdown import MarkItDown

md = MarkItDown()
result = md.convert("path/to/file.xlsx")
print(result.text_content)
```

With Azure Document Intelligence (better PDF quality):
```python
from markitdown import MarkItDown

md = MarkItDown(docintel_endpoint="<endpoint>")
result = md.convert("test.pdf")
print(result.text_content)
```

With LLM image descriptions (for PPTX and images):
```python
from markitdown import MarkItDown
from openai import OpenAI

client = OpenAI()
md = MarkItDown(llm_client=client, llm_model="gpt-4o")
result = md.convert("presentation.pptx")
print(result.text_content)
```

### Plugins

MarkItDown supports 3rd-party plugins. Plugins are disabled by default. Enable with:
```bash
# List installed plugins
markitdown --list-plugins

# Convert with plugins enabled
markitdown --use-plugins path-to-file.pdf
```

**OCR plugin** (`markitdown-ocr`) extracts text from embedded images in PDF/DOCX/PPTX/XLSX:
```bash
pip install markitdown-ocr
```

```python
from markitdown import MarkItDown
from openai import OpenAI

md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
)
result = md.convert("document_with_images.pdf")
print(result.text_content)
```

### Docker
Only use Docker from the upstream MarkItDown source repository or another directory that actually contains a Dockerfile:

```sh
docker build -t markitdown:latest .
docker run --rm -i markitdown:latest < ~/your-file.pdf > output.md
```

## Workflow

1. **Confirm source and destination** — identify the input path/URL and output `.md` path. If no output path is specified, use the same basename with `.md`.
2. **Check dependencies** — if `markitdown` is not installed, install the narrowest useful extra set, or use `pip install 'markitdown[all]'` for broad format support.
3. **Choose method** — use the CLI for one-off conversions and Python API for batch conversion or integrations.
4. **Run conversion** — preserve document structure such as headings, tables, lists, and links when the source format exposes them.
5. **Validate output** — check that the Markdown file exists, is non-empty, and contains expected headings/tables/text. For scanned PDFs or image-heavy files, check whether OCR/LLM support is needed.
6. **Optional enhancement** — use Azure Document Intelligence for higher-fidelity PDF conversion, or enable plugins for OCR/LLM vision features.

## Troubleshooting

- `markitdown: command not found`: activate the virtual environment or run `python -m markitdown`.
- `ModuleNotFoundError`: install MarkItDown in the same Python environment used by the command or script.
- Empty or sparse PDF output: the PDF may be scanned or image-based; try `markitdown-ocr` with an OpenAI-compatible vision model or Azure Document Intelligence.
- Excel output is missing formulas or formatting: MarkItDown is intended for text/table extraction, not spreadsheet rendering fidelity.
- Audio or YouTube conversion fails: install the relevant optional extra and confirm network/API availability.
- Plugin output is missing OCR text: confirm `markitdown --list-plugins`, pass `--use-plugins`, and provide `llm_client`/`llm_model` for Python API usage.
