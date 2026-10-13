---
name: verify-rendered-document
description: Use when you produced or changed a PDF, slide deck, Word file or other rendered document and are about to say it is done.
compatibility: Uses pdftoppm and pdftotext from poppler, and LibreOffice (soffice) for Office formats. Any tool that renders pages to images will do.
metadata:
  status: "candidate"
  topic: "documents"
  created: "2026-10-13"
  updated: "2026-10-13"
  verified: "2026-10-13"
  origin: "own"
  approval: "none"
---

# Look at the rendered document before calling it done

A build that succeeds says the tool finished. It does not say the pages are right. Every defect below was once shipped by a check that never opened the output.

## Steps

1. **Build with the tool the reader will open or print with,** when that is known. A file printed from Word is paginated by Word, and LibreOffice may break the pages elsewhere. If you cannot use the reader's tool, say that page numbers and page breaks are unconfirmed.
2. **Get a PDF.** For Office formats: `soffice --headless --convert-to pdf <file>`.
3. **Render pages to images and look at them.** `pdftoppm -r 60 -png out.pdf page` writes one image per page. Look at every page of a short document. For a long one: the first and last page, every page with a table, a figure or a change of section, and a sample of the rest.
4. **Check these on the images, not in the source:**
   - headings are rendered as headings, with no literal `#` or stray markup
   - nothing runs off the page, the slide or a table cell, and no text sits behind a footer
   - every figure and table is present and readable
   - page numbers, headers and footers exist where the document promises them
   - special characters and math survived, with no boxes and no gaps
   - a table of contents or an index points to the right pages
5. **Check the text layer as well:** `pdftotext -layout out.pdf -`, then search for a few phrases that must be there and a few that must be gone.
6. **Read the build log** for missing characters, overfull boxes and unresolved references. Some tools delete the log after a successful build; keep it while debugging.
7. **Regenerate every exported format** after the last edit to the source, so the copies do not disagree with it.
8. **Fix, rebuild, look again.** Render once more at the end, even if the last change was small.

## Verify

- You looked at page images made after the final build, and you can say which pages.
- The counts are as expected: pages or slides, figures, tables.
- The report says what was checked by eye and what was not.

## Pitfalls

- Finding a heading's words in the extracted text passes while the heading is printed as a literal `# Title` inside a paragraph.
- Extracted text has page indices whether or not a page number is printed. To confirm page numbers, look at the image.
- A build that exits successfully can still have dropped glyphs or pushed content off the page.
- Text extraction cannot see overlap or overflow.
- Cutting content to save pages before measuring what drives the page count. Forced page breaks often account for more than the text does.
- Deleting the template's sample slide or page the wrong way can leave a file the viewer cannot open. Open the final file in a viewer, not only in the library that wrote it.
