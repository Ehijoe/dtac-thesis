# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A LaTeX B.Sc. thesis (University of Ibadan, Dept. of Computer Science): *Dynamic Traffic-Aware Web Caching: Implementing Heuristic Routing to Mitigate AI Scraper Interference*. The system described (DTAC, a heuristic proxy + modified-LRU cache written in Rust) is implemented elsewhere; this repo holds only the write-up.

## Build

The build uses `pdflatex` + `bibtex` via latexmk (see `main.fdb_latexmk`):

```sh
latexmk -pdf main.tex      # full build incl. bibliography reruns
latexmk -c                 # clean aux files (keeps PDF)
```

All build outputs (`*.aux`, `*.bbl`, `*.log`, `*.pdf`, etc.) are gitignored. Check `main.log` for undefined references/citations after edits.

## Structure

- `main.tex` holds the preamble, all formatting rules, and all front matter inline (title page, certification, dedication, acknowledgement, abbreviations, ToC/LoF, abstract), then `\include`s chapters from `chapters/`.
- Chapters 04 (results), 05 (discussion), and `appendix_a` are placeholders, and their `\include` lines in `main.tex` are commented out. Uncomment them when the chapters get content. `\listoftables` is also commented out until the first table exists.
- `references.bib` is a single BibTeX database. Keys follow `author_firstword_year` (e.g. `zhang_rethinking_2026`), which looks Zotero/Better BibTeX-generated.
- Figures live in `figures/` and are referenced as `figures/<Name>.png`.

## Conventions

- **Formatting follows the university's thesis style spec.** The `% --- 1.x ...` comments in `main.tex` refer to sections of that spec: 12pt Times (`newtxtext`/`newtxmath`), A4 with a 3.7cm left margin, double spacing, uppercase centered 14pt chapter headings formatted as "CHAPTER X: TITLE", and front matter in roman page numbers. Hyperlinks are deliberately black for print. Hyphenation is globally disabled (`hyphenat[none]`). Don't change these without being asked.
- **Citations use `apacite` (APA).** Use `\cite{}` for parenthetical citations and `\citeA{}` for narrative ones ("\citeA{kim_scrapers_2025} define...").
- Front-matter chapters use `\chapter*` plus a manual `\addcontentsline{toc}{chapter}{...}`.
- Label prefixes: `chap:`, `fig:`.
- New abbreviations go in the alphabetical table in `main.tex`'s "List of Abbreviations".
- Title-page text (title, name, matric number, date) is hardcoded inside the `titlepage` environment. The `\title`, `\author`, `\matriculation`, `\degree`, and `\submissionmonth` macros are defined but the title page doesn't use them, so edits must go into the `titlepage` block itself.
