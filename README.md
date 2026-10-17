# Dynamic Traffic-Aware Web Caching

**Implementing Heuristic Routing to Mitigate AI Scraper Interference**

B.Sc. Computer Science thesis, University of Ibadan · Joseph Alikah · Supervised by Dr. Nancy Woods

This repository holds the LaTeX write-up. The system itself (DTAC) lives at
**[github.com/Ehijoe/dtac](https://github.com/Ehijoe/dtac)**.

---

## The problem

Web caches work because humans are predictable. A small set of popular pages gets most of the
traffic (a Zipf distribution), so keeping those pages close to users saves a lot of work.

AI scrapers break that assumption. They crawl huge numbers of distinct pages once each to build
training sets or feed retrieval systems. To a cache this is a *scan*: a flood of one-off requests
that pushes out the content humans actually come back for. Under LRU, the textbook policy, the
human hit rate drops steadily as scraper traffic grows.

The usual fixes don't fit well:

- **Blocking or CAPTCHAs** add friction for humans, and modern bots get past them anyway.
- **Separate caches for humans and bots** work at CDN scale but waste memory for smaller sites.
- **Scan-resistant algorithms** like SIEVE and S3-FIFO help, but they only see access patterns,
  not *who* is asking.

## The idea

Give the cache an extra piece of information: how likely it is that each request comes from a
bot. Then let that score decide how much each request counts.

## What I built

DTAC (Dynamic Traffic-Aware Cache) is written in Rust and has two parts.

**1. A heuristic router.** It scores every request with a bot probability between 0 and 1,
using only cheap, passive signals:

- **Velocity:** how fast this IP is sending requests (a bit-shifted moving average).
- **"Shy Guys":** an IP that keeps fetching pages but has *never* fetched a favicon, CSS, JS or
  image.
- **Headers:** no `Accept-Language` header.
- **Datacenter origin:** the IP belongs to a cloud or hosting provider (offline ASN lookup).

The signals are added up as log-odds starting from a 2% "probably human" prior, then squashed
through a sigmoid. Per-IP state is held in a fixed-size Space-Saving table, so memory stays
bounded however much traffic arrives.

**2. A score-aware cache engine.**

- Every request adds `1 − bot_probability` to the resource's popularity score, which slowly
  decays. Humans add about a full point; detected bots add almost nothing.
- A miss is stored only as metadata in a **ghost cache**. The payload is admitted once the
  score crosses a threshold, which in practice means a second human request.
- **Eviction** samples 8 random entries and removes the lowest-scoring one, so no global sorted
  list is needed.
- State is split across up to 256 locked shards so concurrent requests don't block each other.

There is also a real reverse proxy (Axum) built on the same code, with Prometheus metrics.

## How I tested it

- **Synthetic traffic** from a seeded Python generator:
  - Humans browse in sessions over a 100,000-page Zipf catalog. 30% are returning visitors who
    never refetch the favicon.
  - Bots scan the catalog uniformly at 10%, 30%, 50% and 75% of total traffic.
  - **Naive** bots have the classic mechanical signatures.
  - **Stealthy** bots imitate browsers: real headers, residential IPs, a few requests per IP,
    and one favicon fetch each.
- **Five algorithms:**
  - DTAC
  - DTAC with the score switched off (an ablation)
  - LRU
  - SIEVE
  - S3-FIFO
- **Calibration** of every constant on a held-out seed, which is never used in the results.
- **350 replays** over five seeds, reported as means with 95% confidence intervals. Algorithms
  are compared per seed (paired differences), since they all see identical human traffic.

## What I found

At a cache of 10,000 entries (10% of the catalog):

| | DTAC | No score | LRU | SIEVE | S3-FIFO |
|---|---|---|---|---|---|
| Human hit rate, 10% naive bots | **58.3%** | 58.0% | 52.1% | 57.1% | 57.3% |
| Human hit rate, 75% naive bots | **58.2%** | 53.9% | 41.1% | 54.3% | 53.1% |
| Cache pollution, 75% naive bots | **0%** | 48% | 79% | 37% | 55% |

- **Against naive bots it works.** The router separated bots from humans perfectly (ROC-AUC
  1.0), with no human flagged at 0.5. DTAC's human hit rate didn't move as bot traffic went
  from 10% to 75%, and bots never got a single item into the cache.
- **The score is what makes the difference.** The no-score ablation degrades like the other
  algorithms. The gap between DTAC and its ablation grows from 0.3 to 4.3 points as bots
  increase.
- **Stealthy bots defeat the heuristics.** The router does no better than chance (AUC 0.485).
  DTAC then behaves exactly like its ablation: it still beats LRU and S3-FIFO and ties SIEVE,
  but the extra protection is gone. It fails gracefully rather than badly.
- **Capacity matters.** At 20% of the catalog, DTAC falls behind SIEVE. Its constants were tuned
  for 10%, and a large cache has less to lose to bots anyway.
- **It's cheap.** Scoring takes 23–39 µs at the 99th percentile. The memory cost is about 28 MiB
  more than LRU, mostly for the ghost cache.
- **The real proxy agrees.** A replay over HTTP through the reverse proxy matched the in-process
  hit rate to within 0.03 points.

## Caveats

- **Synthetic traffic only.** The naive bots were built with exactly the signatures the
  heuristics look for, so perfect detection on them is an upper bound, not a real-world accuracy
  figure.
- **Untested conditions.** Real labelled traffic, shifting popularity, shared/NAT IPs and
  byte-sized caches were all out of scope.

## Where it could go next

- Evaluate on real, labelled traffic logs.
- Add harder-to-fake signals: subnet-level rates, TLS fingerprints, lightweight learned
  classifiers.
- Tune the promotion threshold and decay rate automatically as conditions change.
- Apply the same score weighting on top of SIEVE or S3-FIFO.

---

## Repository layout

| Path | Contents |
|---|---|
| `main.tex` | Preamble, formatting, front matter (title page, abstract, lists) |
| `chapters/` | Ch. 1 Introduction · Ch. 2 Literature Review · Ch. 3 Methodology · Ch. 4 Results · Ch. 5 Discussion, Conclusion and Recommendations · Appendix A |
| `references.bib` | Bibliography (APA via `apacite`) |
| `results/` | Benchmark summary and calibration outputs copied from the implementation repo |
| `Project_README.md` | The implementation repo's own README |
| `figures/` | Image assets (the diagrams themselves are drawn in TikZ) |

## Building

```sh
latexmk -pdf main.tex   # builds main.pdf, including the bibliography
latexmk -c              # removes auxiliary files
```

Needs a TeX Live installation with `pdflatex`, `bibtex`, `apacite`, `newtx`, `pgfplots` and
`booktabs`.
