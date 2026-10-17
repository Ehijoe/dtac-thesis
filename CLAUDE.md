# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A LaTeX B.Sc. thesis (University of Ibadan, Dept. of Computer Science): *Dynamic Traffic-Aware Web Caching: Implementing Heuristic Routing to Mitigate AI Scraper Interference*. The system described (DTAC: a log-odds heuristic router feeding a bot score into a cache engine with ghost-cache admission, decayed per-resource scores and a sampled eviction tournament, written in Rust) is implemented in a separate repository; this repo holds the write-up plus copies of that repo's README and benchmark outputs.

## Build

The build uses `pdflatex` + `bibtex` via latexmk (see `main.fdb_latexmk`):

```sh
latexmk -pdf main.tex      # full build incl. bibliography reruns
latexmk -c                 # clean aux files (keeps PDF)
```

All build outputs (`*.aux`, `*.bbl`, `*.log`, `*.pdf`, etc.) are gitignored. Check `main.log` for undefined references/citations after edits.

## Structure

- `main.tex` holds the preamble, all formatting rules, and all front matter inline (title page, certification, dedication, acknowledgement, abbreviations, ToC/LoF, abstract), then `\include`s chapters from `chapters/`.
- All chapters are active: 04 (results), 05 (discussion, conclusion and recommendations, incl. limitations and future work) and `appendix_a` (configuration, calibration sweeps, supplementary tables, reproduction commands).
- `references.bib` is a single BibTeX database. Keys follow `author_firstword_year` (e.g. `zhang_rethinking_2026`), which looks Zotero/Better BibTeX-generated.
- Figures live in `figures/` and are referenced as `figures/<Name>.png`.

## Implementation sources

The thesis text must agree with these copies from the implementation repo. They are the source of truth for every number and design detail in the write-up:

- `Project_README.md`: the implementation's own README (architecture, generator, benchmark method, results, known limitations, and a "Thesis vs. implementation" table listing where the chapters used to disagree with the code).
- `results/summary.md` / `results/summary.csv`: aggregated benchmark results over seeds 1–5 (mean ± 95% CI and paired differences). The CSV has a few metrics the markdown omits (e.g. `t_total_p99_us`, `bot_triggered_share`, overall BHR).
- `results/tuning.md` / `results/tuning.json`: calibration of the router and engine constants on held-out seed 0.
- `results/raw/` and `results/tuning_raw/` are per-run JSON files, gitignored. Ignore them.

## Conventions

- **Formatting follows the university's thesis style spec.** The `% --- 1.x ...` comments in `main.tex` refer to sections of that spec: 12pt Times (`newtxtext`/`newtxmath`), A4 with a 3.7cm left margin, double spacing, uppercase centered 14pt chapter headings formatted as "CHAPTER X: TITLE", and front matter in roman page numbers. Hyperlinks are deliberately black for print. Hyphenation is globally disabled (`hyphenat[none]`). Don't change these without being asked.
- **Citations use `apacite` (APA).** Use `\cite{}` for parenthetical citations and `\citeA{}` for narrative ones ("\citeA{kim_scrapers_2025} define...").
- Front-matter chapters use `\chapter*` plus a manual `\addcontentsline{toc}{chapter}{...}`.
- Label prefixes: `chap:`, `fig:`, `tab:`, `eq:`.
- Tables use `booktabs` (`\toprule`/`\midrule`/`\bottomrule`). Charts are inline `pgfplots`; the preamble defines a shared `resultaxis` style and one greyscale style per algorithm (`algdtac`, `algnoscore`, `alglru`, `algsieve`, `algsfifo`). Reuse them so all charts look the same.
- Results tables report mean ± 95% CI half-width over seeds 1–5, and paired differences as `dtac − X`. Copy numbers from `results/summary.md`/`summary.csv`, never from memory.
- New abbreviations go in the alphabetical `longtable` in `main.tex`'s "List of Abbreviations" (a `longtable` because the list spans two pages).
- Title-page text (title, name, matric number, date) is hardcoded inside the `titlepage` environment. The `\title`, `\author`, `\matriculation`, `\degree`, and `\submissionmonth` macros are defined but the title page doesn't use them, so edits must go into the `titlepage` block itself.
