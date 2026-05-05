# Ollama Model Specs

**Live site:** [https://jlaiii.github.io/ollama-model-specs/](https://jlaiii.github.io/ollama-model-specs/)

A static, single-page catalog of every model in the Ollama library — with per-tag download sizes, hardware requirements, benchmark scores, and side-by-side comparison tools. No build step. No framework. No API key.

---

## What It Is

This is a **static single-page app** that catalogs **229 model families** from [ollama.com/library](https://ollama.com/library), exposing more than **1,950 individually quantized tags** in a searchable, filterable, sortable interface.

For each model, you can instantly see:

- **Estimated download size** (GB) for each quantized tag
- **VRAM requirements** per quantization level (Q2_K through FP16)
- **Benchmark scores** (MMLU, MATH, HumanEval, GPQA, Arena Elo, BBH, IF-Eval, MMLU-Pro, SWE-Bench)
- **Capability chips** (vision, code, math, reasoning, chat, medical, embedding, safety, MoE, small, tool-use, uncensored)
- **Parameter counts** and model architecture hints

The site includes two dedicated tools built into the same page:

- **My Specs** — Enter your system RAM and VRAM, pick a preferred quantization, and see which models you can actually run. Green "XGB VRAM" for runs, red "Needs XGB VRAM" for no-go.
- **Compare** — Select up to 6 models and view their sizes, benchmarks, and capabilities side-by-side in a unified comparison table.

Everything runs client-side in the browser. There is no backend, no database server, and no external API calls after the initial JSON load.

---

## Why It Exists

Ollama does not publish a public catalog API that lists every model family or every quantized tag. The official `/api/tags` endpoint only returns 39 curated models. Individual library pages are client-side rendered and heavily rate-limited — making bulk scraping unreliable.

If you want to know whether a 70B Q4_K_M model will fit on your 24GB GPU, or whether a specific coding model outperforms another on HumanEval, you normally have to hunt across fragmented model cards, Hugging Face repos, and Discord threads.

This project fills that gap. It gives users a **single reference** where they can check specs **before** committing to multi-gigabyte downloads.

---

## What's Included

### Model Coverage

| Metric | Count |
|--------|-------|
| Model families | 229 |
| Quantized tags | 1,950+ |
| Benchmarked models | 90 |
| Capability-backfilled models | 104 |

### Quantization Levels

The site computes sizes and VRAM estimates for 5 quantization formats:

| Quant | Size Multiplier (GB per 1B params) |
|-------|-----------------------------------|
| Q2_K | 0.38 |
| Q4_K_M | 0.61 |
| Q5_K_M | 0.71 |
| Q8_0 | 0.93 |
| FP16 | 2.11 |

Each multiplier is multiplied by a **1.10 overhead factor** to account for GGUF metadata, tokenizer vocabulary, and tensor headers.

### Benchmarks Exposed

For the 90 models where benchmark data is available, the following scores are shown:

- **MMLU** — Massive Multitask Language Understanding
- **MATH** — Mathematical reasoning
- **HumanEval** — Code generation
- **GPQA** — Graduate-level reasoning
- **Arena** — Chatbot Arena Elo rating
- **BBH** — Big-Bench Hard
- **IF** — Instruction following evaluation
- **MMLU-Pro** — Professional-level MMLU
- **SWE** — Software engineering tasks (SWE-Bench)

### Capability Chips

Each model is tagged with zero or more capability chips:

| Chip | Meaning |
|------|---------|
| Chat | General conversational |
| Code | Programming / code generation |
| Vision | Image understanding |
| Math | Mathematical reasoning |
| Reasoning | Chain-of-thought / logic |
| Medical | Biomedical / clinical |
| Embedding | Vector embeddings |
| Safety | Alignment / safety-tuned |
| MoE | Mixture of Experts architecture |
| Small | Small parameter count (under 3B) |
| Tool Use | Function calling / tool use |
| Uncensored | Uncensored / unrestricted variant |

### Interactive Tools

- **My Specs** — Input your system RAM (GB), GPU VRAM (GB), preferred execution mode (CPU/GPU/any), and preferred quantization. The grid refreshes with a compatibility badge on each card.
- **Compare** — Click "Compare" on any card to add it to the compare bar. Select up to 6 models, then click "Compare" to open a side-by-side modal.
- **Details** — Click "Details" on any card to open a modal showing every quantized tag, its download size, all benchmark scores, VRAM estimates, and capability list.

---

## How It Works Technically

### Static Architecture

The entire application is **pure static HTML/CSS/JS**. There is no build step, no bundler, no framework, and no runtime dependency on npm, React, Vue, or any other library.

- `index.html` — One file. Everything.
- `data/database.min.json` — ~277 KB minified JSON served via GitHub Pages.
- All logic is vanilla ES6: `fetch`, `Array.prototype`, `Set` for capability chips, and native DOM APIs.

### Data Pipeline

1. **Scrape** — A curl fetch of `ollama.com/library` HTML extracts all 229 model family names from hrefs like `/library/<name>`.
2. **Merge** — Benchmark JSON from the companion repo `jlaiii/ollama-benchmarks` is merged by exact model name match.
3. **Compute** — Per-tag sizes are derived from quantization formulas (see table above) rather than measured from every individual blob. VRAM is estimated at 1.5× the download size.
4. **Backfill** — 104 models had no capability tags. A Python script inferred them from name heuristics (e.g., `llava` → vision, `coder` → code, `med` → medical).
5. **Generate** — Two artifacts are emitted: `database.json` (pretty, for debugging) and `database.min.json` (single-line, for production).
6. **Patch** — `fix_missing.py` backfills parameter counts for 68 edge-case models using name heuristics to ensure every model has a valid tags array.

### Size Formula

```
size_GB = params_in_billions × quant_multiplier × 1.10_overhead
```

Example: llama3.1 8B Q4_K_M = 8.0 × 0.61 × 1.10 = **4.88 GB** (actual Ollama: ≈ 4.92 GB)

### VRAM Formula

```
vram_GB = size_GB × 1.5
```

This is a conservative rule-of-thumb that covers model weights plus KV-cache overhead for a moderate context length. Actual VRAM usage varies with batch size, context window, and architecture.

### Client-Side Features

All of the following happen in the browser with zero network round-trips after the initial JSON load:

- **Search** — Text input filters across model names, parameter counts, and tag names in real time.
- **Filter** — Multi-select capability chips; intersection filter (models must match ALL active chips).
- **Sort** — 11 sort options: Name A→Z/Z→A, Parameters largest/smallest, Download largest/smallest, MMLU/MATH/HE/Arena/GPQA highest.
- **Compare** — Compare state is a lightweight JS array; up to 6 models.
- **My Specs** — RAM/VRAM thresholds are evaluated client-side against every tag's computed size.

---

## How It Was Made

### Phase 1: Data Collection

The project started with `curl -A 'Mozilla/5.0' https://ollama.com/library | python3 ...` to extract the full list of 229 model family names. The Ollama `/api/tags` endpoint only returns 39 models, so HTML scraping was the only option for full coverage.

### Phase 2: Benchmark Integration

Benchmark data already existed in `jlaiii/ollama-benchmarks`. A Python merge script aligned those scores to the catalog using exact string matching on model family names.

### Phase 3: Size & VRAM Computation

Rather than download every GGUF blob (terabytes of data), sizes were computed from published quantization ratios. A `gen_db.py` script generated per-tag sizes for 5 quants per parameter size. A `patch_tags.py` script refined these with the actual quantization constants.

### Phase 4: Metadata Backfill

Of the 229 models, 104 had empty capability lists. A Python script used regex heuristics on names:

```python
if 'llava' in name or 'vision' in name or 'minicpm-v' in name:
    capabilities.append('vision')
if 'code' in name or 'coder' in name or 'deepcoder' in name:
    capabilities.append('code')
if 'deepseek-r1' in name or 'qwq' in name:
    capabilities.append('math')
```

These heuristics are conservative and spot-checked against known model origins.

### Phase 5: Frontend

The UI was built iteratively in pure HTML/CSS/JS:

1. **Prototype** — Card grid from a sample JSON slice.
2. **Search/Filter** — Input + chip intersection logic.
3. **Sort** — 11-way sort pipeline.
4. **My Specs** — Form panel with compatibility badges.
5. **Compare** — Compare bar + side-by-side modal up to 6 models.
6. **Details Modal** — Per-model drill-down with all tags, sizes, and benchmarks.

### Phase 6: Bug Fixing

Building a complex UI without a framework introduced several challenges:

- **Nested template literal escaping** — After multiple escaping disasters (`&lt;`, `&gt;`, `&amp;` leaking into JS strings and breaking nested backticks), the entire script block was rewritten using only simple string concatenation (`'...'+var+'...'`).
- **compare bar text glitch** — `esc()` double-encoded the `&times;` HTML entity into `&amp;times;`, rendering the literal word "times;" instead of the × symbol. Fixed by using raw Unicode `×` outside the `esc()` scope.
- **MoE sort bug** — `parseFloat('8x22B')` returned 8 instead of 176 for Mixture-of-Experts models. Fixed with a custom `parseParam()` function that multiplies `NxM` notation.
- **Mobile CSS** — Playwright audits at 375px and 390px viewports found iOS auto-zoom, cramped modal dt/dd floats, and infinite compare bar growth. Fixed with mobile-specific media queries, 16px input fonts, max-height on compare bar, and dt/dd un-floating.
- **Benchmark sort bug** — Dropdown values (`mmlu`) didn't match benchmark keys (`MMLU`), causing sort to silently fallback to name sort. Fixed by uppercasing sort values.

---

## How to Use

### Open the Live Site

Navigate to [https://jlaiii.github.io/ollama-model-specs/](https://jlaiii.github.io/ollama-model-specs/)

### Search

Type in the search box to filter across:
- Model family names (e.g., `llama`, `qwen`, `mistral`)
- Tag names (e.g., `q2_k`, `7b`, `fp16`)
- Parameter counts (e.g., `70b`)

Search is case-insensitive and filters in real time as you type.

### Sort

Open the Sort dropdown to reorder results. Available options:

1. Name A→Z
2. Name Z→A
3. Parameters (largest)
4. Parameters (smallest)
5. Download Size (largest)
6. Download Size (smallest)
7. MMLU Score (highest)
8. MATH Score (highest)
9. HumanEval (highest)
10. Arena Elo (highest)
11. GPQA (highest)

### Filter by Capability

Click any capability chip to toggle it. Only models matching **all** active chips are shown. Click again to deselect.

### My Specs

1. Click the **My Specs** button.
2. Enter your system RAM (GB) and GPU VRAM (GB).
3. Select your preferred execution mode (Any / CPU Only / GPU Only).
4. Select your preferred quantization (default: Q4_K_M).

The grid refreshes with a compatibility indicator on each card:
- **Green** — Model runs on your hardware at that quant
- **Red** — Model needs more VRAM than you have

### Compare

1. Click **Compare** on any model card.
2. The compare bar appears at the bottom.
3. Add up to 6 models.
4. Click **Compare** in the bar to open the side-by-side modal.
5. Click the **×** on a model in the compare bar to remove it, or **Clear** to reset.

### Details Modal

Click **Details** on any card to open a modal showing:
- Every quantized tag with its download size
- All benchmark scores
- VRAM estimates for all 5 quants
- Capability chips
- Hardware requirements (min/rec RAM, min/rec VRAM, GPU preference)

---

## Data Accuracy Disclaimer

### Download Sizes

Reported sizes are **estimates**, not exact byte measurements. They are computed from quantization multipliers applied to parameter counts. For popular models, these estimates are typically accurate to within **±0.5 GB**:

| Model | Tag | Estimate | Actual |
|-------|-----|----------|--------|
| llama3.1 | 8b | 4.88 GB | ≈ 4.92 GB |
| deepseek-r1 | 7b | 4.27 GB | ≈ 4.67 GB |
| phi4 | 14b | 8.54 GB | ≈ 9.1 GB |
| mixtral | 47b | 28.67 GB | ≈ 28.67 GB |

Exact sizes would require scraping Ollama's Docker registry manifests — a technique that was partially researched but not batch-applied due to rate-limiting complexities.

### VRAM Estimates

VRAM is calculated at **1.5× the model file size**. This is intentionally rough. Actual VRAM consumption depends on context window length, batch size, and architecture. Use it as a **planning guideline**, not a profiling result.

### Capability Tags

104 models have **heuristic-based** capability tags inferred from model names, not verified against upstream documentation. Always verify capabilities against the upstream model card before deploying in production.

### Benchmarks

Benchmark scores are sourced from `jlaiii/ollama-benchmarks` and reflect the best available public evaluation. Scores may vary depending on evaluation framework and quantization level.

---

## How to Contribute / Run Locally

### Clone

```bash
git clone https://github.com/jlaiii/ollama-model-specs.git
cd ollama-model-specs
```

### Serve Locally

Because the app fetches `data/database.min.json` via `fetch()`, you must serve it over HTTP, not `file://`.

```bash
python3 -m http.server 8000
# Then open http://localhost:8000
```

### Edit the Frontend

All UI code lives in `index.html`. There are no framework components, no webpack config, and no transpilation. Edit the HTML/CSS/JS and refresh.

### Edit the Data

- `data/database.json` / `data/database.min.json` are generated by Python scripts.
- To regenerate, run the merge/compute pipeline and output new JSON files.
- Verify your changes against the live HTML before opening a pull request.

### Pull Requests

PRs are welcome for:
- Correcting benchmark scores
- Adding missing model families
- Improving size/VRAM formulas
- Fixing UI bugs or accessibility issues
- Adding screenshots or documentation

Please keep the frontend dependency-free.

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Data generation | Python 3 (json, re, math) |
| Scraping | curl + Python string parsing |
| Frontend | Vanilla HTML5 / CSS3 / ES6 |
| Styling | Custom CSS, no framework |
| Fonts | Inter + JetBrains Mono (Google Fonts) |
| Hosting | GitHub Pages |

Key architectural constraints:
- **Zero build step** — `index.html` is hand-written and committed as-is.
- **Zero frontend dependencies** — No npm packages shipped to the browser.
- **Zero runtime API calls** — All data is bundled in `database.min.json`.

---

## License

MIT License — feel free to use, modify, and distribute.

Built by [jlaiii](https://github.com/jlaiii).
