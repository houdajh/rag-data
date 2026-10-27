# library/

Knowledge base of the portfolio RAG chatbot. This file and `index.json` live at the root of `library/` on purpose: only `md/` (and `portfolio/`, outside this folder) is read by the indexer, so neither of them is indexed.

## Structure

```
library/
├── README.md                 this file (not indexed)
├── index.json                list of all documents (not indexed)
├── Jouhar_Houda_CV_English.pdf   the CV, served by the backend (not indexed)
├── pdf/<topic>/<name>.pdf    originals, source of truth
├── md/<topic>/<name>.md      one Markdown file per PDF, same tree and same name (indexed)
└── _to_review/               duplicates and superseded files (not indexed)
```

Topics: `architecture`, `design-patterns`, `frontend`, `java`, `kafka`, `security`, `spring`.

## Pairing rule

`pdf/<topic>/<name>.pdf` ⇄ `md/<topic>/<name>.md`. The two trees must stay identical apart from the extension. `index.json` lists every pair with its original (pre-reorganisation) file name.

## Naming rules

- Folder and file names: lowercase, words separated by hyphens, no spaces, no accents, no copy suffixes such as `(8)`.
- The name keeps the meaning (and language) of the document title, e.g. `immutabilite`, `communication-entre-les-services`.
- Never edit the content of a PDF. If a PDF changes, replace it and regenerate its Markdown.

## Markdown files

Each Markdown file starts with a front matter block:

| Field | Meaning |
|---|---|
| `title`, `topic` | Title of the document and its topic folder |
| `source_pdf` | PDF path relative to `library/` |
| `review_needed` | `true` when a human should compare the file with the PDF |
| `review_pages_transcribed_from_images` | PDF pages whose code or text was read from screenshots (no OCR) |
| `review_pages_diagrams` | PDF pages with diagrams, replaced by a one-line `[diagram: ...]` placeholder |
| `review_pages_indentation_rebuilt` | PDF pages whose code indentation was rebuilt from the text positions in the PDF |
| `review_notes` | What to check on that file |

Content rules: the original wording, headings, lists, tables and fenced code blocks (with the language) are kept; nothing is summarised or added. Diagrams become `[diagram: ...]`; code that could not be read becomes `[code unreadable: needs manual check]`.

## What is indexed for RAG

The backend (`GitHubContentService.listAllMarkdownFiles`) reads every `*.md` under `library/md/` (recursively) and under `portfolio/`. Everything else is ignored: PDFs, the CV, `README.md`, `index.json` and `_to_review/`. PDFs are kept as originals only, so the same content is never indexed twice. Do not put other `.md` files under `library/md/`.

## Adding a document

1. Add the PDF as `pdf/<topic>/<name>.pdf` (naming rules above).
2. Create `md/<topic>/<name>.md` with the front matter and the converted content.
3. Add an entry to `index.json`.
4. Push, then re-index (see below).

## Re-indexing

The backend assigns random IDs to chunks, so indexing the same file twice creates duplicates. After renaming or changing documents: clear the Qdrant collection (or delete and let the backend recreate it), restart the backend (the indexer runs only once per process), then call `POST /api/rag/admin/reindex`.
