# BENCH.pdf (formerly Ream)

A PDF editor that runs in the browser, meant to replace Adobe Acrobat for everyday work. It's one file, `bench-pdf.html`. Open it in a browser. There's nothing to install or build.

Libraries load from a CDN, so it needs an internet connection:
- pdf.js 3.11.174 (cdnjs) shows the pages
- pdf-lib 1.17.1 (unpkg) writes the saved PDF

It was first made as a claude.ai artifact. When it runs there, saving goes through `window.claude.use('downloads')`. Opened as a file in Chrome or Edge, Save PDF opens a Windows "Save as" window so you can choose the folder and name. It remembers the last folder. Other browsers fall back to a normal download.

## What it does today

- **Open and combine:** open several PDFs, add more to the end, or drop them onto the page. Thumbnails are colored by the file each page came from.
- **Pages:** drag thumbnails to reorder. Rotate left or right, delete, duplicate, insert a blank page, or extract the selected pages into a new PDF.
- **Markup:** highlight, freehand pen, text, sticky note, box, white-out, and stamps (RECEIVED / REVIEWED / APPROVED / PAID / COPY / VOID). Choose from 4 colors and 9 to 24 pt text.
- **Editing:** select, move and delete markup. Undo and redo keep 80 steps.
- **Viewing:** zoom and fit-to-width, light and dark themes, works on a phone.
- **Saving:** builds a new PDF with pdf-lib and never changes the original file.

## How saving works (worth knowing)

Markup is **burned into the page** when you save (pdf-lib `drawRectangle` / `drawText` / `drawSvgPath`). It is not saved as real PDF annotations. So:
- Once saved, nobody can edit or remove the notes, in BENCH.pdf or in Acrobat.
- **White-out only covers text, it doesn't remove it.** The text under the white box is still in the file and can be selected, copied or searched. Don't use it to hide sensitive information until true redaction exists (see below).
- Text uses Helvetica, so characters it doesn't have (emoji, many non-Latin scripts) save as `?`.

## Gaps compared to Acrobat

Roughly in order of how much they matter for everyday use:

1. **True redaction:** actually delete the text and image data under a box. This is what makes white-out safe for sensitive information.
2. **Fill forms:** read the PDF's AcroForm fields, let people type in them, and save the values. pdf-lib supports this.
3. **Sign:** draw, type or upload a signature and place it on a page. Optionally add a date stamp.
4. **Find text:** search the document and jump to each match (pdf.js `getTextContent`).
5. **Editable notes:** save markup as real PDF annotations (Highlight, Ink, FreeText, Text, Square) so other programs can edit them later. Also load annotations that are already in the file.
6. **Text-aware highlight:** snap a highlight to the words you select instead of drawing a rectangle.
7. **Edit existing text:** Acrobat's flagship feature and the hardest one here. The cheap version is to white-out the old text and type over it.
8. **More output options:** compress, split by page ranges, save pages as images, add page numbers, headers and footers, or watermarks.
9. **Images and OCR:** add images to a page, and run OCR on scanned pages (tesseract.js) so they can be searched.
10. **Passwords:** open and add passwords. Today a password-protected file is refused on open.
11. **Works offline:** bundle the libraries instead of loading them from a CDN, and possibly make it an installable PWA.
12. **Unicode text:** embed a font that covers more characters (pdf-lib + fontkit).

## Look and layout (v2, 2026-11-01)
Styled after BENCH Fantasy (see `/mnt/project-files/BENCH THEME/README.md`), with a layout modeled on WeTransfer's sign screen: an icon menu on the left (Read, Organize, Mark up, Search), a panel for the current task, a grey canvas and floating controls. It opens light. The moon/sun button switches to dark and remembers the choice.

## What changed in v2
- **Opens like Adobe.** A PDF opens to page 1 in a normal reading view, with a strip of page thumbnails on the left to jump around.
- **Organize pages** is the outlined button at the top (or the Organize icon). Every page appears in a grid. Click to select, Shift-click to select a run and Ctrl-click to add one, then drag to reorder. A bar at the bottom works on the selection: rotate, duplicate, move to start or end, save as a separate PDF, delete. The Documents list reorders or removes whole files, and "Combine with another PDF" adds files.
- **Clean-up on open.** Blank pages (no text and almost no ink) and exact duplicates (same text and same picture as an earlier page) are removed when a PDF is opened or added. A green note lists what went, and "Put them back" or Undo restores them. You can turn it off with the checkbox in Organize.
- **Search.** A search bar sits at the top (Ctrl+F) and searches every page. Matches are highlighted on the pages and listed in the Search panel. "Select these pages" hands them to Organize, and "Highlight all" turns them into real highlights. Scanned pages with no text can't be searched, and it says so.
- **Worker fix.** The pdf.js worker was loaded as a page script, which forced pdf.js into its slower "fake worker" and printed the console warning. It now runs as a real worker.

## Installed app (GitHub Pages)

This repo is served at https://hdramos29-hub.github.io/bench-pdf/. Open it in Chrome or Edge and click the install icon in the address bar to get a BENCH.pdf window with its own icon. Then go to Windows Settings › Apps › Default apps › `.pdf` and pick BENCH.pdf, so double-clicked PDFs open in it.

`index.html` is the same single-file editor plus a manifest link. `manifest.webmanifest` registers it as a PDF handler, and `sw.js` caches it for offline use (bump `VERSION` when `index.html` changes). PDFs never leave the computer.
