# GenBank TBL Validator

[![Live Demo](https://img.shields.io/badge/Live%20Demo-biopep.42web.io-00d4aa?style=for-the-badge&logo=googlechrome&logoColor=white)](https://biopep.42web.io/pep1)

A fully client-side, single-file HTML tool for validating NCBI GenBank feature table (`.tbl`) files before submission. Catches the structural and biological errors that cause NCBI rejections, with no server, no install, and no data leaving your browser.

> **Note:** To benefit from Ai assistant and explanation use the link above 
---

## Contents

- [Quick Start](#quick-start)
- [Features](#features)
- [Files](#files)
- [Usage](#usage)
- [Validation Checks](#validation-checks)
- [Genome Statistics](#genome-statistics)
- [Circular Genome Map](#circular-genome-map)
- [Reference Comparison](#reference-comparison)
- [Codon Inspector](#codon-inspector)
- [Session Persistence](#session-persistence)
- [Supported Organisms](#supported-organisms)
- [Scoring System](#scoring-system)
- [Changelog](#changelog)

---

## Quick Start

```bash
# Clone or download the repository, then open in any modern browser:
open GB_tbl_validator_v2_39.html

# Or serve locally (required for codon translation and FASTA features):
python3 -m http.server 8080
# then visit http://localhost:8080/GB_tbl_validator_v2_39.html
```

> **Note:** Uploading files is the supported input method — see [Usage](#usage) below. FASTA-based checks (start/stop codon validation, internal stop detection, codon usage/RSCU, GC content) and codon translation require `genetic_codes.json` to be served from the same directory — use a local server, or the file will still validate structurally with these checks disabled.

---

## Features

- **Zero dependencies** — single `.html` file, runs entirely in the browser
- **No data upload to any server** — all parsing and validation happens locally in your browser; the tool requires you to *upload* your `.tbl` (and, for full feature coverage, `.fasta` and optionally `.gb`) files, but nothing ever leaves your machine
- **Multi-table support** — upload `.tbl` files containing multiple sequence tables
- **Optional FASTA pairing** — upload FASTA alongside the TBL to unlock sequence-level checks
- **Reference genome comparison** — load a GenBank `.gb` reference to compare annotation coordinates, CDS lengths, and gene content
- **Same-species / cross-species comparison mode** — switch between strict and permissive thresholds when comparing against a reference from a different taxon
- **Inverted Repeat (IR) awareness** — detects chloroplast IR regions and suppresses false duplicates
- **Trans-splicing support** — correctly handles mixed-strand and IR-duplicated genes (e.g. `rps12`, `clpP`)
- **Add / edit features interactively** — insert new gene, CDS, tRNA, rRNA, repeat_region, or misc_feature entries directly in the Features tab without editing the raw TBL
- **Gene search and cross-tab navigation** — filter features by name/type/product; jump from any issue or reference comparison row directly to the matching feature
- **Circular genome map** — OGDraw-style SVG map with strand/function colour modes, GC content ring, declared LSC/SSC/IRA/IRB region ring, and transcription-direction arrows; exportable as SVG or PNG
- **Publication-style RSCU charts** — grouped bar chart, stacked per-AA bar chart (matching the format used in organelle-genome papers), and heatmap; codon-to-amino-acid families are derived dynamically from the genome's actual declared genetic code (not a hardcoded standard-code table), so RSCU is correct even for non-standard codes; configurable RSCU threshold (default 1.0; one-click preset to 1.6 per Sharp & Li 1987)
- **Session persistence** — the last validated `.tbl`, FASTA, reference file, and organism selection are remembered locally (IndexedDB) so reopening the tool doesn't require re-uploading everything; fully local, clearable anytime
- **Citation helper** — one-click plain-text and BibTeX citation, auto-dated
- **Exportable results** — download issues as TSV, copy genome stats summary to clipboard, export validation report as standalone HTML/PDF
- **Dark, professional UI** — colour-coded severity levels, tabbed interface, responsive layout, light/dark theme toggle

---

## Files

```
GB_tbl_validator_v2_39.html  # Main application (all-in-one)
genetic_codes.json           # NCBI genetic code tables (required for translation, RSCU, and the circular map's genetic-code-aware features)
README.md                    # This file
```

`genetic_codes.json` must be in the same directory as the HTML file when served locally. The tool will work without it but codon-level checks, RSCU charts, and the Codon Inspector modal will be disabled.

---

## Usage

### 1. Upload your `.tbl` file (and, ideally, `.fasta` and `.gb`)

Drop your files onto the upload box, or click it to browse — you can select multiple files at once (`.tbl` + `.fasta` + `.gb` together). The `.tbl` file must contain a `>Feature <seqid>` header line.

```
>Feature lcl|CP123456
1	10000	gene
			gene	rbcL
1	10000	CDS
			gene	rbcL
			product	ribulose-1,5-bisphosphate carboxylase/oxygenase large subunit
			transl_table	11
```

Only the `.tbl` file is strictly required to run validation, but uploading the matching `.fasta` alongside it is strongly recommended — without it, an entire category of checks (start/stop codon validation, internal stops, GC content, RSCU) can't run, since there's no sequence to check them against.

### 2. What the FASTA unlocks

Uploading the corresponding nucleotide FASTA alongside your `.tbl` enables:
- Start codon validation
- Internal stop codon detection
- Stop codon verification
- Abbreviated stop detection (single T / poly-A tail)
- GC content, GC skew, AT skew
- Codon usage table with RSCU charts
- Per-CDS GC% chart
- The GC-content ring on the circular genome map

### 3. Select organism / genetic code

Choose the appropriate organism type from the dropdown. This controls:
- Which NCBI genetic code table is used for codon validation and translation
- Which codon-to-amino-acid families RSCU is computed against
- Which expected gene set is checked for completeness

### 4. (Optional) Load a reference GenBank file

Load a `.gb` / `.gbk` reference file to enable the **Ref Compare** tab. Choose **Same species** mode for close relatives or **Cross-species** mode when comparing against a reference from a different taxon — cross-species mode suppresses false-critical length/structure differences while keeping genuine errors (wrong genetic code, absent genes) flagged.

### 5. Click **Validate**

Results appear across the **Issues**, **Features**, **Auto-Fix**, **Ref Compare** (if a reference is loaded), **Stats**, **Map**, and **JSON Export** tabs.

---

## Validation Checks

Issues are classified into three severity levels:

| Level | Colour | Meaning |
|-------|--------|---------|
| `CRITICAL` | Red | NCBI will reject the submission |
| `WARNING` | Amber | Likely error; review before submitting |
| `INFO` | Green | Informational; may be intentional |

### Critical checks

| Code | Description |
|------|-------------|
| `NO_COORDINATES` | Feature has no coordinate spans |
| `GENE_NAME_IS_NUMBER` | A raw coordinate was pasted into the `gene` qualifier |
| `MIXED_STRAND_IN_FEATURE` | Feature has spans on both strands without `trans_splicing` qualifier |
| `FORBIDDEN_QUALIFIER` | `protein_id` or `transcript_id` present in `.tbl` (not allowed) |
| `CDS_WITHOUT_GENE` | CDS has no associated `gene` feature — NCBI rejects this |
| `RNA_WITHOUT_GENE` | tRNA or rRNA has no associated `gene` feature |
| `GENE_WITHOUT_CHILD` | `gene` feature has no CDS or RNA child |
| `STRAND_MISMATCH` | Gene and its CDS/RNA are on opposite strands (non-IR) |
| `ILLEGAL_CDS_RNA_OVERLAP` | CDS overlaps a tRNA or rRNA by more than 3 bp |
| `GENE_COMPLETELY_OVERLAPPED` | One feature is entirely contained within another of the same name — NCBI will reject with "gene completely overlapped by other genes". Trans-spliced genes sharing a single exon across IR copies are correctly downgraded to INFO. |
| `INTERNAL_STOP_CODON` | FASTA-verified: CDS contains an in-frame stop codon (frameshift, wrong genetic code, or bad exon boundary) |

### Warning checks

| Code | Description |
|------|-------------|
| `GENE_NAME_WHITESPACE` | Gene name contains spaces |
| `GENE_NAME_NUMBER_SUFFIX` | Gene name ends with a number (auto-numbering artefact) |
| `QUALIFIER_ON_GENE` | `product`, `codon_start`, or `protein_id` placed on a `gene` feature instead of CDS |
| `INVALID_QUALIFIER` | BLAST/tool artefact qualifier present (e.g. `blast_score`, `e_value`) |
| `CDS_OUTSIDE_GENE` | CDS extends beyond the boundary of its parent gene feature |
| `GENE_CDS_NAME_MISMATCH` | Overlapping gene and CDS have different gene names |
| `DUPLICATE_GENE_NAME` | Same gene name appears multiple times with different coordinates (IR pairs suppressed) |
| `CDS_CDS_OVERLAP` | Two CDS features overlap (non-IR, non-trans-spliced) |
| `TRNA_ANTICODON_PRODUCT_MISMATCH` | The anticodon in the tRNA gene name decodes a different amino acid than the `/product` qualifier (e.g. `trnT-GGU` / `tRNA-Pro` would be flagged). Anticodon→amino-acid mapping is computed by reverse-complementing each anticodon against the standard codon table. |
| `TRNA_UNUSUAL_SIZE` | tRNA length outside expected 60–100 bp range |
| `CDS_NOT_DIV3` | CDS effective length is not divisible by 3 |
| `SUSPICIOUS_CDS_LENGTH` | Complete CDS is shorter than 100 bp (legitimately short genes such as `petN` and `petL` are exempt) |
| `MISSING_PRODUCT` | CDS lacks a `product` qualifier |
| `MISSING_TRANSL_TABLE` | CDS lacks `transl_table` qualifier (NCBI requires this explicitly) |
| `UNKNOWN_TRANSL_TABLE` | `transl_table` value is not a recognised NCBI genetic code |
| `TRANS_SPLICED_SINGLE_SPAN` | `/trans_splicing` present but only one exon span |
| `RRNA_UNUSUAL_SIZE` | rRNA length outside expected range for its subunit type |
| `INVALID_START_CODON` | FASTA-verified: CDS does not start with a valid codon for the selected genetic code |

### Info checks

| Code | Description |
|------|-------------|
| `TRANS_SPLICED_DETECTED` | Mixed-strand trans-spliced gene detected (expected) |
| `INVERTED_REPEAT_DETECTED` | IR region identified; lists duplicated gene pairs |
| `IR_CDS_OVERLAP` | Overlap between IR copy CDS features (expected) |
| `SMALL_CDS_RNA_OVERLAP` | CDS/RNA overlap ≤ 3 bp (within NCBI tolerance) |
| `ABBREVIATED_STOP_CODON` | CDS length mod 3 = 1, consistent with single-T abbreviated stop (poly-A tail completes TAA) |
| `MISSING_CODON_START` | `codon_start` absent (defaults to 1; informational only) |
| `ALTERNATIVE_START_CODON` | FASTA-verified: CDS uses a non-ATG but valid alternative start codon |
| `CIRCULAR_WRAPAROUND` | Feature spans cross the genome origin |
| `POSSIBLY_MISSING_GENES` | One or more expected genes for the selected organism type are absent |
| `GENE_COMPLETELY_OVERLAPPED` (INFO) | Two same-named features share an identical exon but diverge elsewhere — consistent with a trans-spliced gene duplicated across the IR (e.g. `rps12`). Both features are likely correct. |

---

## Genome Statistics

The **Stats** tab displays a full annotation summary, computed from the TBL and (when available) the FASTA:

- Sequence length, GC%, AT%, GC skew, AT skew
- Gene / CDS / tRNA / rRNA counts
- CDS metrics: total bp, average/min/max length, strand distribution, trans-spliced count
- Coding density, abbreviated stop count, partial feature counts
- Intergenic region analysis — measured from individual exon spans (not feature envelopes) so trans-spliced genes do not inflate the count; overlapping features are merged before gap measurement; IR duplicates at identical coordinates are deduplicated. Displayed as "Intergenic regions" to distinguish from sequencing gaps (N-stretches).
- tRNA completeness badge (22 standard tRNAs) with per-amino-acid grid
- Start and stop codon usage distribution
- Per-CDS GC% chart with genome-average reference line
- **Codon usage — three views:**
  - **Grouped Bars** — one bar per codon, grouped by amino acid
  - **Per-AA (stacked)** — one stacked bar per amino acid group; each coloured segment = one codon's RSCU value; total RSCU sum labeled on top; codon-box legend below — matches the layout used in published organelle-genome RSCU figures. Colours follow the 3rd-codon-position convention (U = blue, C = orange, A = green, G = purple).
  - **Heatmap** — colour grid of RSCU values by amino acid and codon
  - **Genetic-code-aware families** — which codons are treated as synonymous with which is derived from the genome's own declared genetic code (via `genetic_codes.json`), not a fixed standard-code assumption. This matters for genomes using a non-standard code (e.g. an animal mitochondrial code), where degeneracy genuinely differs (some amino acids gain or lose codons relative to the standard table).
  - **Configurable RSCU threshold** — input field with one-click presets for 1.0 (no bias) and 1.6 (Sharp & Li 1987 high-preference cutoff used in most plastid publications). The threshold line updates live across all chart views; codons above the threshold are highlighted.

Results can be exported as a TSV file or copied as a formatted plain-text summary.

---

## Circular Genome Map

The **Map** tab renders an OGDraw-style circular genome map as SVG, exportable as standalone SVG or PNG.

- **Ring layout** (center → edge): structural region ring (LSC/SSC/IRA/IRB, plastid-only) → generic gene ring → rRNA ring → tRNA ring → CDS− ring → CDS+ ring → GC content ring
- **Colour modes** — toggle between **Strand/Type** (by feature type and strand) and **Function** (OGDraw-style functional categories: photosystem I/II, cytochrome b6f, ATP synthase, NADH dehydrogenase, ribosomal proteins, etc.), with separate category schemes for plastid vs. mitochondrial genomes
- **Transcription-direction arrows** — a curved arrow sits directly against the CDS+ ring (outside it) showing the + strand reads clockwise, and another sits directly against the CDS− ring (adjacent, on its inner side) showing the − strand reads counterclockwise; explained in the legend, colour-coded to match
- **Structural region ring** — shows declared LSC/SSC/IRA/IRB regions when the `.tbl` contains explicit `repeat_region` features with `rpt_type=inverted` and an "inverted repeat" note; plastid genomes only. Renders independently of the GC ring (the two used to be mutually exclusive; they now share the map without conflict)
- **GC content ring** — outward deviation = higher local GC%
- **Issue markers** — critical/warning triangles placed at the genomic position of the underlying issue
- **Legend** — full colour key plus a plain-language explanation of the direction arrows, included in both the on-screen view and exported files

---

## Reference Comparison

Load a GenBank flat file (`.gb` / `.gbk`) as a reference genome to compare your annotation against a known submission. The **Ref Compare** tab shows, for each gene:

- Presence/absence in user annotation vs reference
- CDS length difference (bp and %)
- Protein length comparison (when FASTA is loaded)
- Exon count comparison (with trans-splicing awareness)
- Strand agreement
- Translation table agreement

When a gene appears under the same name multiple times (IR duplication or unjoined trans-spliced exons), the tool selects the most complete representative on each side — preferring entries with a `/translation` qualifier (reference) or `/product` qualifier (user TBL), then by most exons, then by longest spliced length. This prevents an incompletely-annotated IR duplicate from being compared against the correctly-curated entry.

**Comparison modes** — toggle between:
- **Same species (strict)** — CDS length differences >15% are Critical, >5% are Warning
- **Cross-species (permissive)** — length and exon-count differences are downgraded to Review at most; only absent genes and wrong genetic codes remain Critical

Each gene row in the table carries a **⇢ Features** navigation pill that switches to the Features tab and highlights that gene directly.

Multi-copy genes display a **×N copies** badge; expanding the row lists every copy found on both sides with coordinates, length, and whether each extra copy appears to be a legitimate IR duplicate.

---

## Features Tab

The **Features** tab lists every parsed feature with interactive editing:

- **Click any coordinate** to edit it inline; changes re-validate automatically
- **Add Feature** button (top-right) — opens a panel to insert a new `gene + CDS`, `gene + tRNA`, `gene + rRNA`, `gene only`, `repeat_region`, or `misc_feature`. Supports multi-exon / trans-spliced entries (add extra spans with the + button). The paired `gene` feature is created automatically; `transl_table` is inherited from existing CDS features. Product names auto-fill for known tRNA and rRNA gene names.
- **Gene search** — filter the feature table live by gene name, feature type, or product; match count shown; ✕ to clear
- **Cross-tab navigation** — every issue in the Issues tab and every gene row in Ref Compare carries a clickable gene pill (⇢ gene name) that switches to the Features tab, runs the search, and scrolls to the matching row

---

## Codon Inspector

Click the **codons** button on any CDS row in the Features tab to open the Codon Inspector modal. It shows:

- Full codon-by-codon translation grid with colour-coded start, stop, and unknown codons
- Amino acid sequence bar with position highlighting
- Warnings for internal stops, unknown codons, abbreviated stops, and missing terminal stop
- Exon breakdown for trans-spliced genes

Requires `genetic_codes.json` and a loaded FASTA sequence.

---

## Session Persistence

The tool remembers your last validated session — the `.tbl` text, FASTA sequence, any loaded reference `.gb` file, and the organism/genetic-code selection — using the browser's IndexedDB, so reopening the page doesn't require re-uploading everything.

- Saved automatically after every successful validation
- On reopening, a banner offers **Restore & validate** or **Dismiss**
- A persistent status line under the upload area shows whenever a session is stored, with a one-click **clear it** link
- Everything is stored locally in the browser only — nothing is ever uploaded or sent anywhere
- Uses IndexedDB rather than `localStorage`, since a full genome FASTA (hundreds of KB, more for larger mitogenomes) is a poor fit for `localStorage`'s smaller, synchronous storage model
- Some browsers restrict IndexedDB in private/incognito windows — session persistence will silently no-op in that case (validation itself is unaffected)

---

## Supported Organisms

| Dropdown value | NCBI Genetic Code | Notes |
|----------------|-------------------|-------|
| Bacterial / Plant Plastid | 11 | Chloroplast genomes; includes full IR awareness |
| Vertebrate Mitochondrial | 2 | |
| Invertebrate Mitochondrial | 5 | |
| Yeast Mitochondrial | 3 | |
| Mold / Protozoan Mitochondrial | 4 | |
| Echinoderm / Flatworm Mitochondrial | 9 | |
| Ascidian Mitochondrial | 13 | |
| Trematode Mitochondrial | 21 | |
| Pterobranchia Mitochondrial | 24 | |
| Chlorophycean Mitochondrial | 16 | |
| Alternative Yeast Nuclear | 12 | |
| Standard | 1 | Default fallback |

The selected organism also determines the expected gene set used by `POSSIBLY_MISSING_GENES` and the codon-to-amino-acid families used for RSCU. The plant plastid set covers 75 genes including the full photosystem, ribosomal protein, ATP synthase, NADH dehydrogenase, and RNA polymerase complement. Gene name aliases (e.g. a genome using `clpP1` instead of `clpP`) are resolved automatically; `clpP1` is not separately required since it's an alternate name for the same gene, not a distinct one.

---

## Scoring System

Each validation run produces a quality score (0–100) displayed at the top of the Issues tab. The maximum is always 100 regardless of whether a reference file is loaded.

**Without reference:**

| Category | Max pts |
|----------|---------|
| Gene Completeness | 30 |
| Structural Integrity | 25 |
| CDS Annotation Quality | 20 |
| Translation Validity | 15 |
| Annotation Hygiene | 10 |

**With reference loaded** (Gene Completeness is partially covered by Reference Concordance):

| Category | Max pts |
|----------|---------|
| Gene Completeness | 10 |
| Structural Integrity | 25 |
| CDS Annotation Quality | 20 |
| Translation Validity | 15 |
| Annotation Hygiene | 10 |
| Reference Concordance | 20 |

Deductions are severity-weighted: each `CRITICAL` issue −10 pts, each `WARNING` issue −3 pts, `INFO` issues no deduction. A score ≥ 90 is considered submission-ready. IR-aware suppression ensures that expected IR duplicates do not contribute false deductions. Acknowledged issues can be individually dismissed from the score.

---

## Changelog

### v2.39

- **Removed the "paste directly" text input** — it only ever accepted `.tbl` text with no way to pair a matching FASTA or reference file, so any FASTA-dependent check (start/stop codons, internal stops, GC content, RSCU) silently couldn't run for anything typed or pasted in. File upload (`.tbl` + `.fasta` + optional `.gb`, dropped together or selected at once) is now the only supported input path, so there's no longer a route into the tool that produces an incomplete, checks-disabled validation run without it being obvious why
- Removed the "Load example" button for the same reason — it populated the same text-only, FASTA-less input path
- Updated the empty-input warning and upload-box copy to reflect this

### v2.29 – v2.38

Full development notes for this range live in `SESSION_SUMMARY_v2.16_to_v2.28.md` and `SESSION_SUMMARY_v2.29_to_v2.37.md` in this repo. Highlights:

**Bug fixes:**
- **RSCU codon families were hardcoded to the standard/code-11 table** — silently wrong for genomes using a non-standard genetic code (e.g. vertebrate mitochondrial, where AGA/AGG are stop codons and ATA is Met, not Arg/Ile). RSCU now derives codon-to-amino-acid families dynamically from the genome's own declared genetic code, via `genetic_codes.json`
- **`POSSIBLY_MISSING_GENES` false positives** — the plant-plastid expected-gene list stored names in mixed case (`rbcL`, `psbA`, `matK`...), but the comparison lowercases everything first, so any entry with an internal capital letter could never match regardless of whether the gene was present. Fixed by lowercasing the whole expected-gene table
- **`clpP1` required as a separate gene from `clpP`** — it's just an alternate name for the same gene; removed as a separately-required entry (a genome using either spelling now passes correctly)
- **RSCU threshold line appeared stuck at 1.0** — a permanent "RSCU=1.0" reference line was always drawn at full visual weight, while the adjustable threshold line only appeared if it differed from 1.0 by more than a small margin. The adjustable line now always renders and is clearly labeled; the 1.0 baseline was demoted to a plain gridline
- **3rd-codon-position legend overlapped the codon-legend row** in the per-AA RSCU chart for any amino acid with 4 synonymous codons (a common case). Fixed by anchoring the legend forward from the last row's actual position instead of back-computing from total chart height
- **Ring-axis labels on the genome map rendered as garbled overlapping text** (`CDS+`/`CDS−`/`tRNA`/`rRNA` all at the same height, only ~15-20px apart while each label is ~25px wide). Fixed by staggering them vertically
- **A legend entry got silently truncated to unreadable ellipsis** — a full-sentence legend entry was pushed into the swatch-grid legend, which truncates long text by design. Moved the explanation into its own caption line instead, in both the on-screen and exported legend
- **Structural region ring (LSC/SSC/IRA/IRB) and GC ring couldn't render together** — the region ring used to sit inside the GC ring's own radius band, so it was hard-coded mutually exclusive with it. Moved the region ring to sit under the rRNA ring instead; both now render simultaneously

**Features added:**
- Circular genome map — OGDraw-style SVG map with strand/function colour modes, GC ring, declared-region ring, transcription-direction arrows (positioned directly against the CDS+/CDS− rings they represent, not floating arbitrarily), SVG/PNG export
- Citation button + modal (plain text and BibTeX, auto-dated), next to the theme toggle
- Session persistence via IndexedDB (see [Session Persistence](#session-persistence))

### v2.16 – v2.28

See `SESSION_SUMMARY_v2.16_to_v2.28.md` for full detail. Highlights: 3′-partial CDS scoring fix, AI explain feature with friendly error handling, toast notifications replacing native alerts, standalone HTML/PDF validation report export, keyboard-accessible tab bar, functional-category gene colouring on the genome map (verified against real OGDraw reference images), declared LSC/SSC/IRA/IRB structural region detection, multi-column legend layout.

### v2.12

**New features:**
- **Add Feature panel** — insert new features (gene+CDS, gene+tRNA, gene+rRNA, repeat_region, misc_feature) directly in the Features tab. Supports multi-span/trans-spliced entries; gene feature auto-created; transl_table auto-inherited; product auto-filled for tRNA and rRNA names; inserted at the correct linear coordinate position
- **Gene search** in Features tab — live filter by gene name, type, or product with match count
- **Cross-tab navigation** — clickable gene pills in Issues and Ref Compare tabs jump to the matching feature in Features tab with highlight and scroll
- **Per-AA stacked RSCU bar chart** — publication-style stacked bar view: one bar per amino acid group, each segment = one codon's RSCU, total sum labeled on top, codon-box legend below. Colours follow 3rd-codon-position convention (U=blue, C=orange, A=green, G=purple)
- **RSCU threshold control** — configurable threshold input with quick-set buttons for 1.0 and 1.6 (Sharp & Li 1987); threshold line updates live in all chart views; codons/bars above threshold highlighted in amber
- **Comparison mode toggle** in Ref Compare tab — Same species (strict) vs Cross-species (permissive); permissive mode suppresses false-critical length/structure differences when using a different-taxon reference while keeping genetic-code mismatches and absent genes as Critical
- **rps12 / IR shared-exon pattern** — `GENE_COMPLETELY_OVERLAPPED` for two same-named features that share an identical exon but diverge elsewhere is now correctly downgraded to INFO with an explanatory message, rather than firing as Critical

**Bug fixes:**
- **Root-cause fix: `parseFeatureTable` never set `feature.low`/`feature.high`** — these fields were only initialised by the GenBank `.gb` parser, not the `.tbl` parser. This silently disabled three things for every `.tbl` file: `GENE_COMPLETELY_OVERLAPPED` detection, `DUPLICATE_GENE_NAME` / `DUPLICATE_RNA_GENE_NAME` detection, and intergenic gap statistics (all showed 0 / empty). Fixed by computing `low = min(spans)` and `high = max(spans)` in `flushFeature()`
- **`CDS_OUTSIDE_GENE` was structurally unreachable** — the spatial-overlap fallback always set `matched` before the overshoot warning loop could run, so a CDS that poked past its gene boundary was silently associated with the gene without warning. Fixed by decoupling the overshoot check from the bookkeeping match
- **Score exceeded 100 when reference loaded** — `geneMax` dropped from 30 → 20 when a reference was loaded, but the 20-pt Reference Concordance category was added on top, making the maximum achievable score 110. Fixed: `geneMax` now drops to 10 when a reference is loaded (10+25+20+15+10+20 = 100)
- **Anticodon→amino-acid table was incorrect** — the lookup table had duplicate keys and wrong entries (e.g. `trnT-GGU` flagged as Pro, `trnV-GAC` as Leu). Table rebuilt programmatically by reverse-complementing each anticodon against the standard codon table; 61 entries, stop-codon anticodons correctly excluded
- **`pickCanonicalCds` selected wrong IR copy** — when a gene appeared multiple times (IR duplication), the comparison picked the longest entry as the canonical representative. If the reference file had an incompletely-annotated IR duplicate (no `/translation` qualifier) that happened to be longer, the tool compared against that instead of the curated entry. Fixed: entries with `/translation` (reference) or `/product` (user TBL) are now preferred before considering span count or length
- **Intergenic gap calculation was inflated** — the old code used feature envelopes (`f.low`/`f.high`); a trans-spliced gene like `rps12` (envelope spanning ~28 kb) swallowed all features between its exons, producing phantom gaps or hiding real ones. Fixed: gaps are now measured from individual exon spans; overlapping spans are merged before measurement; IR duplicate spans at identical coordinates are deduplicated. UI label renamed from "Gaps" to "Intergenic regions" to avoid confusion with sequencing gaps (N-stretches)
- **RSCU threshold line appeared behind bars** — reference lines were rendered before bars in the SVG, so tall bars obscured the threshold line. Fixed by rendering reference lines after bars in both the grouped and per-AA charts
- **RSCU threshold did not update on input change** — the build functions were local closures inaccessible to `updateRSCUThreshold`. Fixed by exposing them on `window._rscuBuildGrouped` and `window._rscuBuildPerAA` after first render; both charts now rebuild immediately on threshold change without a full re-render
- **`null addEventListener` errors on load** — two event listeners (`addFeaturePanel` backdrop click, `afp-product` input) referenced elements declared after the `<script>` block. Fixed by deferring registration to `DOMContentLoaded`

---

### v2.6
- **Fix:** Restored `↔` bidirectional arrow in `CDS_CDS_OVERLAP` messages
- **New check:** `CIRCULAR_WRAPAROUND` — detects features that span the circular genome origin
- **Parser:** `exception` qualifier now parsed and stored
- **Expected genes:** `bacterial_plant_plastid` set expanded from 43 → 74 genes
- **Score engine:** Gene alias normalisation prevents false "possibly missing" deductions

### v2.5
- Added `GENE_COMPLETELY_OVERLAPPED` critical check
- Added IR duplicate detection and suppression across all checks
- Added `INVERTED_REPEAT_DETECTED` info message
- Codon Inspector modal with full translation grid
- Reference genome comparison tab
- RSCU-style codon usage table and per-CDS GC% chart

### v2.0 – v2.4
- FASTA integration and sequence-level checks
- Trans-splicing detection and suppression
- Genome statistics tab
- Multi-table (multi-sequence) support
- TSV export and clipboard summary

---

## Known Limitations

- Reference comparison requires a GenBank flat file with `CDS` features and `translation` qualifiers to compute protein lengths
- `CIRCULAR_WRAPAROUND` uses a heuristic (span starts at position ≤ 5 and previous span ends in the top 10% of sequence length); very short sequences may produce false positives
- Abbreviated stop codon detection (`T` + poly-A) is confirmed only when FASTA is loaded; without it, a length-mod-1 CDS is reported as `INFO` rather than confirmed
- The tool does not validate qualifier values against the full INSDC feature table specification (e.g. controlled vocabulary for `product` names)
- Codon-level checks (translation validity, RSCU charts, Codon Inspector) require `genetic_codes.json` to be served from the same directory; opening the file directly via `file://` in some browsers will disable these features
- Session persistence (IndexedDB) may be unavailable or restricted in some private/incognito browser windows; validation itself is unaffected, only the "remember last session" convenience

---

## License

MIT — free to use, modify, and distribute. Attribution appreciated but not required.
