---
name: seqout-single-cell-long-read
description: Use when someone wants single-cell or long-read (PacBio, Oxford Nanopore) public data, or wants to know how good it is: "human lung 10x 5' studies with raw FASTQ and cell counts", "Nanopore direct RNA in Arabidopsis", "median UMI per cell by year and chemistry", "Perturb-seq studies". Requires the seqout MCP server.
author: Saket Lab
license: MIT
metadata:
  version: "0.1.0"
  category: bioinformatics
  tags: [seqout, single-cell, long-read, nanopore, pacbio, 10x, qc]
---

# Single-cell and long-read data with seqout

## Overview

seqout labels single-cell studies from three independent kinds of evidence: a parsed cell-by-gene matrix, a barcode-whitelist hit in the reads, and a read-length structure typical of single-cell chemistry. It also flags long-read studies and reports their chemistry. This skill finds those studies and describes their quality.

## When to use

- Planning an atlas or a benchmark: which studies, which chemistry, how many cells, is raw data available?
- Checking one study's single-cell quality.
- Finding PacBio or Nanopore data by organism, assay or instrument.

## Workflow: single cell

1. Orient with `get_single_cell_overview` or `get_singlecell_summary` (totals, top tissues and organisms).
2. `get_singlecell_facets` lists the valid filter values. Call it before filtering.
3. `get_singlecell_projects` returns one row per study. Filters: `chemistry`, `organism`, `tissue`, `cell_or_nucleus`, `modality`, `assay_l1`, `year`, `has_matrix`, `has_fastq`, `has_sra`, `is_long_read`, `perturbation_method`, `intervention_kind`, and `q` (title or accession substring). Sort with `sort` and `order`. `total` is the filtered study count.
4. For one study: `get_project_single_cell_status` (what the label rests on), `get_project_single_cell_summary` (cells, read-derived scan and microbe summary) and `get_project_single_cell` (per-sample cell and gene counts with read-derived species, sex and assay calls). Page `get_project_single_cell` with `limit` / `offset`. It has no `total`, so compare against `n_samples_detailed`.
5. `list_single_cell_studies` selects studies by evidence strength (`min_evidence` 1 to 3, optional `require_matrix`).
6. Quality trends: `get_sc_quality` gives the 25th, 50th and 75th percentile of per-matrix median UMI counts and detected genes by release year and technology (10x 3' and 5' are split). `get_sc_quality_samples` gives the individual matrices behind those quantiles as a stratified subsample.

To get samples instead of studies, use `search_enriched_samples` with `has_matrix` or `single_cell_only` (see `seqout-build-cohort`).

## Workflow: long read

- `search_all(long_read=true)` restricts any keyword search to studies with a long-read experiment.
- `get_longread_summary` (totals by technology and hybrid status), `get_longread_facets` (valid filter values) and `get_longread_projects` (one row per study, mirrors folded together).
- `get_project_longread_chemistry(accession)` lists every PacBio and Nanopore run in a study with its chemistry call.
- `long_read_only` false means the study also used a short-read platform. NULL means no experiment rows exist to tell (GEO-only entries). PacBio Onso is a short-read instrument and is excluded.

## Reading the numbers

- `cells` is a matrix column count. When `unfiltered` is set the columns are 10x barcodes, not cells, so never sum them. The corpus-level cell totals already exclude studies whose only matrix is unfiltered.
- `cell_or_nucleus` is read-derived (intron fraction and mitochondrial percentage) and is null when the study was never scanned or the call is ambiguous.
- `perturbation_method` (Perturb-seq, CROP-seq, ECCITE-seq, Mosaic-seq, sci-Plex, CRISPR-detect) is a method name found in the study's own text. It is not a confirmed design, and null does not mean the study has none.
- A read-derived flag needs unitigs to align across a sizeable fraction of a panel reference. There is no UMI or replicate confirmation.
- NULL is not false. A null flag or `measurable` false means nothing could be measured. An empty list means measured and nothing found.
- `chemistry_confidence` is `exact` (BAM header), `declared` (Nanopore only, from the depositor's protocol text), `bucket` (inferred from the instrument model, labelled "estimated" in some views) or `unknown`. Report the tier next to the call.
- `get_sc_quality` drops thin cells (a minimum number of matrices and studies per point, returned as `min_matrices` and `min_studies`) and uses filtered matrix columns. `evidence` says whether the chemistry was read from FASTQs or declared by the submitter.

## Example requests

- "Which studies have human lung 10x 5' single-cell data with raw FASTQ, and how many cells does each have?"
- "How has median UMI count per cell changed by year and 10x chemistry?"
- "Which organisms other than human and mouse have Nanopore direct RNA studies?"

## Explaining results

Give the study table (accession, tissue, chemistry, cells, matrix and FASTQ availability), say which evidence supports each single-cell label, and state whether cell counts are real cells or barcodes.

## Troubleshooting

- 404 on `get_project_single_cell`: the study is not in the read-derived set. Use `get_project_enriched` and the raw metadata instead.
- 503 from the QC tools: the statistics are being prepared, retry later.
- A study you expect is missing from facets: it may be labelled only by submitter text, so check with `seqout-find-datasets`.

## References

- 10x Genomics chemistries: https://www.10xgenomics.com
