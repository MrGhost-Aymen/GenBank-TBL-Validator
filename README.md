# GenBank TBL Validator

[![Live Demo](https://img.shields.io/badge/Live%20Demo-biopep.42web.io-00d4aa?style=for-the-badge&logo=googlechrome&logoColor=white)](https://biopep.42web.io/pep1)

A fully client-side, single-file HTML tool for validating NCBI GenBank feature table (`.tbl`) files before submission. Catches the structural and biological errors that cause NCBI rejections, with no server, no install, and no data leaving your browser.

---

## Contents

- [Quick Start](#quick-start)
- [Features](#features)
- [Files](#files)
- [Usage](#usage)
- [Validation Checks](#validation-checks)
- [Genome Statistics](#genome-statistics)
- [Reference Comparison](#reference-comparison)
- [Codon Inspector](#codon-inspector)
- [Supported Organisms](#supported-organisms)
- [Scoring System](#scoring-system)
- [Changelog](#changelog)

---

## Quick Start

```bash
# Clone or download the repository, then open in any modern browser:
open GB_tbl_validator_v2_6.html

# Or serve locally (required for codon translation and FASTA features):
python3 -m http.server 8080
# then visit http://localhost:8080/GB_tbl_validator_v2_6.html
```

> **Note:** Paste-mode (no server) works for all structural validation. FASTA-based checks (start/stop codon validation, internal stop detection, codon usage, GC content) and codon translation require `genetic_codes.json` to be served from the same directory — use a local server.

---

## Features

- **Zero dependencies** — single `.html` file, runs entirely in the browser
- **No data upload** — all parsing and validation happens locally; nothing is sent to any server
- **Multi-table support** — paste or load `.tbl` files containing multiple sequence tables
- **Optional FASTA pairing** — paste FASTA alongside the TBL to unlock sequence-level checks
- **Reference genome comparison** — load a GenBank `.gb` reference to compare annotation coordinates, CDS lengths, and gene content
- **Inverted Repeat (IR) awareness** — detects chloroplast IR regions and suppresses false duplicates
- **Trans-splicing support** — correctly handles mixed-strand genes (e.g. `rps12`, `clpP`)
- **Exportable results** — download issues as TSV, copy genome stats summary to clipboard
- **Dark, professional UI** — colour-coded severity levels, tabbed interface, responsive layout

---

## Files

```
GB_tbl_validator_v2_6.html   # Main application (all-in-one)
genetic_codes.json           # NCBI genetic code tables (required for translation)
README.md                    # This file
```

`genetic_codes.json` must be in the same directory as the HTML file when served locally. The tool will work without it but codon-level checks and the Codon Inspector modal will be disabled.

---

## Usage

### 1. Paste your `.tbl` content

Paste the raw text of one or more GenBank feature tables into the left panel. Each table must begin with a `>Feature <seqid>` header line.

```
>Feature lcl|CP123456
1	10000	gene
			gene	rbcL
1	10000	CDS
			gene	rbcL
			product	ribulose-1,5-bisphosphate carboxylase/oxygenase large subunit
			transl_table	11
```

### 2. (Optional) Paste FASTA

Paste the corresponding nucleotide FASTA sequence into the right panel to enable:
- Start codon validation
- Internal stop codon detection
- Stop codon verification
- Abbreviated stop detection (single T / poly-A tail)
- GC content, GC skew, AT skew
- Codon usage table (RSCU-style)
- Per-CDS GC% chart

### 3. Select organism / genetic code

Choose the appropriate organism type from the dropdown. This controls:
- Which NCBI genetic code table is used for codon validation
- Which expected gene set is checked for completeness

### 4. Click **Validate**

Results appear in four tabs: **Issues**, **Features**, **Stats**, and **Ref Compare** (if a reference is loaded).

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
| `GENE_COMPLETELY_OVERLAPPED` | One feature is entirely contained within another of the same name — NCBI will reject with "gene completely overlapped by other genes" |
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
| `TRNA_UNUSUAL_SIZE` | tRNA length outside expected 60–100 bp range |
| `CDS_NOT_DIV3` | CDS effective length is not divisible by 3 |
| `SUSPICIOUS_CDS_LENGTH` | Complete CDS is shorter than 100 bp |
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
| `CIRCULAR_WRAPAROUND` | Feature spans cross the genome origin (span ends near seq-end, next span starts at or near position 1) |
| `POSSIBLY_MISSING_GENES` | One or more expected genes for the selected organism type are absent from the annotation |

---

## Genome Statistics

The **Stats** tab displays a full annotation summary, computed from the TBL and (when available) the FASTA:

- Sequence length, GC%, AT%, GC skew, AT skew
- Gene / CDS / tRNA / rRNA counts
- CDS metrics: total bp, average/min/max length, strand distribution, trans-spliced count
- Coding density, abbreviated stop count, partial feature counts
- Intergenic gap analysis (count, total bp, average, largest gap)
- tRNA completeness badge (22 standard tRNAs) with per-amino-acid grid
- Start and stop codon usage distribution (bar charts)
- Per-CDS GC% chart with genome-average reference line
- Codon usage table (RSCU-style, grouped by amino acid)
- Duplicate coordinate detection

Results can be exported as a TSV file or copied as a formatted plain-text summary.

---

## Reference Comparison

Load a GenBank flat file (`.gb` / `.gbk`) as a reference genome to compare your annotation against a known-good submission. The **Ref Compare** tab shows, for each gene:

- Presence/absence in user vs reference
- CDS length difference (bp and %)
- Protein length comparison (when FASTA is loaded)
- Exon count comparison (with trans-splicing awareness)
- Strand agreement
- Translation table agreement

Rows are colour-coded: critical discrepancies (>15% length difference) in red, warnings (>5%) in amber, exact matches in green. Missing or extra genes are flagged separately.

---

## Codon Inspector

Click the **codons** button on any CDS row in the Features tab to open the Codon Inspector modal. It shows:

- Full codon-by-codon translation grid with colour-coded start, stop, and unknown codons
- Amino acid sequence bar with position highlighting
- Warnings for internal stops, unknown codons, abbreviated stops, and missing terminal stop
- Exon breakdown for trans-spliced genes

Requires `genetic_codes.json` and a loaded FASTA sequence.

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

The selected organism also determines the expected gene set used by `POSSIBLY_MISSING_GENES`. The plant plastid set covers 74 genes including the full photosystem, ribosomal protein, ATP synthase, NADH dehydrogenase, and RNA polymerase complement. Gene name aliases (e.g. `clpP1` → `clpP`) are resolved automatically.

---

## Scoring System

Each validation run produces a quality score (0–100) displayed at the top of the Issues tab. Deductions are severity-weighted:

- Each `CRITICAL` issue: −10 pts
- Each `WARNING` issue: −3 pts
- `INFO` issues: no deduction

A score ≥ 90 is considered submission-ready. IR-aware suppression ensures that expected IR duplicates do not contribute false deductions.

---

## Changelog

### v2.6
- **Fix:** Restored `↔` bidirectional arrow in `CDS_CDS_OVERLAP` messages (was corrupted to `?`)
- **New check:** `CIRCULAR_WRAPAROUND` — detects features that span the circular genome origin (e.g. `rpl2` wrapping from end of sequence back to position 1)
- **Parser:** `exception` qualifier is now parsed and stored on features (e.g. `/exception trans-splicing` on `rps12`), enabling future suppression logic
- **Expected genes:** `bacterial_plant_plastid` set expanded from 43 → 74 genes; added full ribosomal protein complement (`rps2`–`rps19`, `rpl2`–`rpl36`), complete photosystem subunits, and ATP synthase subunits; removed `infA` and `cemA` (frequently absent in angiosperms)
- **Score engine:** Gene alias normalisation (`clpP1` → `clpP`, `pafI` → `ycf3`, etc.) prevents false "possibly missing" deductions for alternative nomenclature

### v2.5
- Added `GENE_COMPLETELY_OVERLAPPED` critical check (the primary cause of NCBI rejection for chloroplast tRNA annotations)
- Added IR duplicate detection and suppression across all checks (`DUPLICATE_GENE_NAME`, `STRAND_MISMATCH`, `CDS_CDS_OVERLAP`)
- Added `INVERTED_REPEAT_DETECTED` info message listing IR gene pairs
- Added `CIRCULAR_WRAPAROUND` groundwork (coordinate parser handles wrap-around spans)
- Codon Inspector modal with full translation grid
- Reference genome comparison tab
- RSCU-style codon usage table
- Per-CDS GC% chart

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

---

## License

MIT — free to use, modify, and distribute. Attribution appreciated but not required.
