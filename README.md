# 🧬 GenBank TBL Validator

A modern, browser-based validator for GenBank 5-column feature tables (`.tbl` format) used in organelle genome submissions to NCBI. Features real-time validation, codon visualization, and BLAST integration.

![Version](https://img.shields.io/badge/version-1.2.0-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)
![Status](https://img.shields.io/badge/status-active-success)

## ✨ Features

### 🔍 Comprehensive Validation
- **Format checks** — Validates 5-column feature table syntax, coordinates, and qualifiers
- **NCBI compliance** — Catches common submission-rejection issues before you submit
- **Gene association** — Verifies CDS/tRNA/rRNA features have proper parent gene features
- **Overlap detection** — Identifies illegal CDS↔RNA overlaps (>3 bp) that NCBI rejects
- **Strand consistency** — Checks that associated features share the same strand
- **Qualifier validation** — Flags forbidden (`protein_id`, `transcript_id`) and invalid qualifiers
- **Gene name errors** — Detects common paste errors (coordinates in gene name field)

### 🧬 Codon Visualization
- **In-frame translation** — Extracts CDS sequences from FASTA and translates using the correct genetic code
- **Codon grid** — Visual representation of every codon with amino acid translation
- **Protein sequence** — Color-coded amino acid string with start/stop highlighting
- **Issue detection** — Identifies internal stop codons (pseudogenes?), unknown codons, missing terminal stops

### 🔬 BLAST Integration
- **One-click BLAST** — Extract CDS sequence and search NCBI BLASTN directly
- **Sequence validation** — Verify annotation accuracy by comparing BLAST results

### 🎨 Modern UI
- **Dark theme** — Cyberpunk-inspired design with animated grid background
- **Responsive** — Works on desktop and tablet
- **Drag & drop** — Upload `.tbl` and `.fasta` files by dragging or browsing
- **Paste support** — Paste feature table text directly
- **Filter & sort** — Filter issues by severity (Critical/Warning/Info)
- **JSON export** — Export validation results as structured JSON

## 🚀 Quick Start

### Option 1: Local Server (Recommended)
```bash
# Clone the repository
git clone https://github.com/MrGhost-Aymen/genbank-tbl-validator.git
cd genbank-tbl-validator

# Start a local server
python -m http.server 8000
# OR
npx serve

# Open in browser
open http://localhost:8000

Option 2: Direct Open

Open index.html directly in your browser. Codon visualization will use built-in genetic codes (organelle-focused). For all 24 genetic codes, use Option 1 with the included genetic_codes.json.
📋 Supported Organism Types
Category	Genetic Code	transl_table
🦋 Invertebrate Mitochondrial	Code 5	5
🐟 Vertebrate Mitochondrial	Code 2	2
⭐ Echinoderm/Flatworm Mito	Code 9	9
🦑 Ascidian Mitochondrial	Code 13	13
🪱 Trematode Mitochondrial	Code 21	21
🪸 Pterobranchia Mitochondrial	Code 24	24
🌿 Plant Mitochondrial	Standard Code 1	1
🍃 Bacterial/Plant Plastid	Code 11	11
🟢 Chlorophycean Mito	Code 16	16
🍄 Yeast Mitochondrial	Code 3	3
🦠 Mold/Protozoan Mito	Code 4	4
🍄 Alt. Yeast Nuclear	Code 12	12
📖 Usage
Basic Validation

    Select your organism type from the dropdown

    Paste your feature table text or upload a .tbl file

    Optionally upload a .fasta file for codon visualization and BLAST

    Click ▶ Validate

    Review issues in the Issues tab, or browse features in the Features tab

Feature Table Format
text

>Feature sequence_id
1	1500	gene
			gene	cox1
1	1500	CDS
			gene	cox1
			product	cytochrome c oxidase subunit I
			transl_table	5

With FASTA File

When a FASTA file is loaded, each CDS feature gets:

    🧬 BLAST — Opens NCBI BLASTN with extracted CDS sequence

    🔬 Codons — Opens codon-by-codon visualization with translation

Example Data

Click Load example to see a working butterfly (Pyrgus ruralis) mitochondrial annotation.
🎯 Validation Checks
Critical Errors (Submission Will Fail)

    ❌ Missing coordinates on features

    ❌ CDS not associated with a gene feature

    ❌ RNA not associated with a gene feature

    ❌ Gene feature with no child (CDS/RNA)

    ❌ Mixed strands within a feature

    ❌ Forbidden qualifiers (protein_id, transcript_id)

    ❌ CDS↔RNA overlap > 3 bp

    ❌ Strand mismatch between gene and its CDS/RNA

    ❌ Gene name is a raw number (paste error)

    ❌ Invalid codon_start value

Warnings (Likely Issues)

    ⚠️ Gene name contains whitespace or number suffixes

    ⚠️ CDS length not divisible by 3 (check codon_start)

    ⚠️ Missing product qualifier

    ⚠️ Missing transl_table

    ⚠️ Unusual tRNA size (< 60 or > 100 bp)

    ⚠️ CDS outside gene boundary

    ⚠️ Unexpected transl_table for selected organism

    ⚠️ Trans-spliced gene with single span

    ⚠️ Duplicate gene names with different coordinates

    ⚠️ Invalid qualifiers (BLAST artefacts)

Info (Suggestions)

    ℹ️ Small CDS↔RNA overlap (within 3 bp tolerance)

    ℹ️ Missing codon_start (defaults to 1)

    ℹ️ Possibly missing expected genes

    ℹ️ Short CDS (< 100 bp, may be partial)

🗂️ File Structure
text

genbank-tbl-validator/
├── genbank_validator.html             # Main application
├── genetic_codes.json      # Complete NCBI genetic codes (optional)
├── README.md               # This file
└── LICENSE                 # MIT License

🔧 Technical Details
Built With

    Vanilla JavaScript — No frameworks, no dependencies

    CSS Custom Properties — Dark theme with consistent design tokens

    Async/Await — Non-blocking genetic code loading

    Tab-Separated Parsing — Handles standard 5-column .tbl format

Browser Support

    Chrome/Edge 90+

    Firefox 88+

    Safari 14+

Known Trans-Spliced Genes

The validator recognizes these trans-spliced genes and allows mixed-strand features:
nad1, nad2, nad5, rps12, nad7, nad4, cox2
📝 NCBI Submission Tips

    Always include transl_table on CDS features

    Gene names should match between gene and CDS/rRNA/tRNA features

    CDS↔tRNA overlaps > 3 bp will be rejected — adjust coordinates

    Never include protein_id or transcript_id in .tbl files

    Partial genes should use < and > in coordinates (e.g., <1)

    Trans-spliced genes should use join() with multiple spans

🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
Areas for Contribution

    Additional validation rules for nuclear genomes

    Support for more feature types (intron, exon, mRNA)

    Batch processing of multiple tables

    Annotation fix suggestions

📄 License

MIT License — see LICENSE file for details.
🙏 Acknowledgments

    NCBI for the Sequin and BankIt submission guidelines

    The organelle genomics community for feedback on validation rules

📬 Contact
  ouamoa@gmail.com
