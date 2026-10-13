---
name: ingest-slide-deck
description: Use when turning lecture slides or a slide-deck PDF into notes, a summary or study material.
compatibility: Uses pdftotext and pdftoppm from poppler.
metadata:
  status: "candidate"
  topic: "documents"
  created: "2026-10-13"
  updated: "2026-10-13"
  verified: "2026-10-13"
  origin: "own"
  approval: "none"
---

# Turning a slide deck into notes

Text extraction gives the words on a slide. It does not give what the slide says, when the slide says it with arrows, bars or position.

## Steps

1. **Read the assignment or exam brief first,** if there is one. Its deadlines and scope decide how deep the rest goes.
2. **Extract the text of every deck:** `pdftotext -layout deck.pdf deck.txt`. Drop blank lines and lines repeated on every slide, such as titles and footers. Slides that build up step by step repeat most of their text.
3. **Use the text for wording and for printed numbers only.** It cannot show which way an arrow points, how many series a chart has, which box a label belongs to, or what is marked in colour.
4. **Mark the slides that argue with geometry:** diagrams, charts, tables, flows, any slide with a figure and little text, and any slide the task depends on. Render those pages as images and read the images: `pdftoppm -f <n> -l <n> -r 80 -png deck.pdf page`.
5. **Read in order.** Decks often ask a question on one slide and answer it on the next. Attribute that answer to the deck.
6. **Keep provenance in the notes.** Mark what the deck says and what you added or inferred differently. Afterwards check both directions: none of your reasoning is presented as the lecturer's, and none of the lecturer's statements is marked as yours.
7. **If the user has made this kind of document before, ask for it** and follow its format before proposing one.
8. **Say what was not covered:** image-only slides you did not view, colour markers, anything unreadable.

## Verify

- Every number in the notes was checked against the slide it came from.
- Every relationship you describe (cause, order, direction, comparison) came from a slide read as an image.
- Pick five statements at random and find each on its slide.
- The number of decks and pages processed matches what is in the folder.

## Pitfalls

- A loop or flow written backwards, because the text came out in reading order and not in arrow order.
- A chart summarised from its axis labels, with whole series missing.
- A sentence about a figure that the figure does not support, written from the words around it.
- Page numbers from extraction are positions in the PDF. They differ from printed slide numbers when slides build up.
- A converted markdown copy of a paper or deck garbles tables. For a number that matters, open the original.
