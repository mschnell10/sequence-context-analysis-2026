# sequence-context-analysis-2026
This repository houses a python script that is used to read Binary Alignment Map (BAM) files and identify sites of enzymatic cytosine deamination. Specifically, it is used to identify enzymatic sequence context preferences by calculating the frequency at which flanking bases occur at deaminated sites.

# Cytosine Deaminase Sequence Context Preference Analysis Script

Analyzes a BAM file (aligned with Bismark, used here purely as a
mismatch-tolerant aligner) to determine
the local sequence-context preference of a cytosine deaminase enzyme.

## Usage Notes

- The enzyme deaminates cytosine (C) to uracil (U), read as thymine (T)
  during sequencing.
- Every C→T mismatch in a read relative to the reference genome is treated
  as a deamination event.
- Bismark is used to process raw fastq files because it
  aligns reads containing C→T changes without penalizing them as
  mismatches. There is no bisulfite chemistry in the experimental setup.

## Upstream pipeline

The BAM file this script expects should already have been processed
through:

1. Bismark genome preparation (C→T and G→A converted genomes)
2. TrimGalore! read trimming
3. Bismark/Bowtie2 alignment
4. Picard sorting + MarkDuplicates deduplication

## Requirements

- Python 3.9+
- Packages listed in `requirements.txt`:

  ```bash
  pip install -r requirements.txt
  ```

## Inputs

| Input | Description |
|---|---|
| **BAM file** | Sorted, deduplicated Bismark/Bowtie2 alignment (see pipeline above) |
| **Reference FASTA** | The genome the BAM was aligned to; must be indexable by `pyfaidx` |

## Configuration

All user-editable settings are in **Cell 1**, under the `CONFIG` block near
the top of the script. At minimum, set:

```python
BAM_FILE   = r"path/to/your_sample.sorted.dedup.bam"
REF_FASTA  = r"path/to/reference_genome.fa"
OUTPUT_DIR = r"path/to/output_folder"
```

Other adjustable parameters:

| Variable | Default | Meaning |
|---|---|---|
| `FLANK_SIZE` | 3 | Bases of context on each side of the C/G (a 7-mer window) |
| `MIN_BASE_QUALITY` | 20 | Minimum Phred score at the C/G position |
| `MIN_MAPPING_QUAL` | 20 | Minimum read MAPQ |
| `MIN_COVERAGE` | 8 | Minimum reads covering a site for it to be kept |
| `CHROMOSOMES` | `[]` (all) | Restrict analysis to specific chromosome names |
| `GENERATE_COMPOSITION_CORRECTED` | `True` | Also produce genome-composition-corrected heatmap/logo (plots 09b/10b) |

## How to run (in Spyder)

The script is organized into three cells, run in order:

1. **Cell 1** — imports, configuration, and helper function definitions.
   No heavy computation. Always run first.
2. **Cell 2** — scans the BAM and reference FASTA once to collect
   per-site C→T counts and genome base composition. This is the slow
   step for large files. Re-run only if you change the BAM/FASTA paths
   or read-filtering settings in Cell 1.
3. **Cell 3** — computes deamination-weighted position frequencies and
   +1-context conversion rates, generates all plots, and writes all
   Excel/CSV outputs. Fast; safe to re-run freely once Cell 2 has run.

## Outputs

Written to `OUTPUT_DIR`:

**Plots (PDF, editable text/vector)**
- `09_position_frequency_heatmap.pdf` — raw observed base frequency by flanking position
- `09b_position_frequency_heatmap_corrected.pdf` — genome-composition-corrected version (if enabled)
- `10_sequence_logo.pdf` — deamination-weighted sequence logo
- `10b_sequence_logo_corrected.pdf` — composition-corrected version (if enabled)
- `11_dinucleotide_context_conversion.pdf` — % C→T conversion by +1 context (CpG/CpA/CpC/CpT)

**Data tables (CSV)**
- `site_deamination_data.csv` — per-site coverage, deamination counts, and context
- `dinucleotide_context_conversion.csv` — summary table backing plot 11

**Data tables (Excel workbooks, with formulas and notes)**
- `00_reference_genome_base_composition.xlsx`
- `site_vs_control.xlsx` (legacy filename; retained per original analysis naming — no control comparison is performed in this version)
- `09_position_frequency_heatmap_data.xlsx`
- `09b_position_frequency_heatmap_corrected_data.xlsx` (if enabled)
- `10_sequence_logo_data.xlsx`
- `10b_sequence_logo_corrected_data.xlsx` (if enabled)
- `11_dinucleotide_context_conversion_data.xlsx`

Each Excel workbook includes a `Notes` sheet describing its contents, and
distinguishes source data (blue text) from calculated values (black text).

## Notes on interpretation

- Every C→T observation at a sufficiently covered site is counted directly as a deamination event.
- Plots 09/10 report raw observed frequencies; 09b/10b (optional) instead
  report frequencies normalized against the reference genome's own local
  base composition.

## Citation

If you use this script, please cite:

> [Author names], "[Manuscript title]," *RSC Chemical Biology*, [year],
> DOI: [manuscript DOI]

## License

[Add your chosen license here, e.g. MIT — see `LICENSE` file in this repository]
