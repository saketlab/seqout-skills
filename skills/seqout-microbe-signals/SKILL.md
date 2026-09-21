---
name: seqout-microbe-signals
description: Use when someone asks about microbes, viruses, infection or contamination in public sequencing data: "samples with HPV", "which organisms show up in human gut samples", "which tissues carry SARS-CoV-2", "is this study contaminated with Mycoplasma". Reports read-derived detections with the caveats they need. Requires the seqout MCP server.
author: Saket Lab
license: MIT
metadata:
  version: "0.1.0"
  category: bioinformatics
  tags: [seqout, microbiome, virus, contamination, hpv, pentimento]
---

# Microbial signals in public data with seqout

## Overview

seqout scans the sequencing reads of many public studies (its scan is called Pentimento) and records which sex, assay, chemistry and microbial reference sequences the reads match. That lets you ask infection and contamination questions of data whose submitters never described them.

A detection means reads match a reference sequence. It does not diagnose infection, and the wording rules below exist for that reason.

## When to use

- Finding samples that carry a virus or bacterium (for example HPV in cervical single-cell data).
- Asking which organisms are common in a tissue and which are likely contaminants.
- Screening one study for cell-culture contamination or reagent background.

## Workflow

1. Corpus and tissue level: `get_pentimento_overview` (sex, assay and microbe totals, top microbes) and `get_tissue_microbes` (detection rate per tissue and organism, over samples that were actually quantified). Cells need at least 2 contributing studies and a tissue with at least 200 screened samples.
2. Sample level: `search_enriched_samples` with a `microbe*` filter (see `seqout-build-cohort`). Any `microbe*` filter both restricts the cohort to samples with a matching detection and attaches per-sample quantification: `microbe_n_detections`, `microbe_reads`, `microbe_kmer_mass`, `microbe_max_breadth_frac` and a `microbes` array.
   - `microbe` is a substring on the organism name, so `papillomavirus` catches every HPV type.
   - `microbe_class="pathogen"` covers "any bacterial infection". Tighten with `microbe_min_breadth`, `microbe_min_kmer_mass` and `microbe_validated_only`.
   - Route infection questions here, not to `has_viral_reads` or `has_bacterial_reads`. Those read a precomputed column on a small subset and understate the cohort by more than 10x.
   - `microbe_tier` sets the breadth floor: `high_breadth` (strict, confident calls), `low_breadth` (default) or `any` (no floor, mostly trace k-mer noise, use only when asked).
3. One study: `get_project_single_cell` for per-sample presence flags, or `search_enriched_samples(study_accession=..., microbe=...)` for per-organism reads, k-mer mass and breadth. Its aggregates cover every detection, even when the `microbes` array is truncated at 50 rows.
4. Full detail: `download_pentimento_detections_csv(accession)` returns one CSV row per sample per organism. A screened sample with nothing found gets one row with an empty organism, so absence is distinguishable from never scanned. `all_rows=true` includes trace and background hits.

## Wording the result

- Write "reference signal consistent with X", never "infected with X".
- Breadth is a fraction of the genome. Viral genomes are about 250x smaller than bacterial ones, so the same percentage is much weaker evidence for a virus. Never compare a viral and a bacterial breadth number directly.
- Low-breadth bacterial rows are only modestly enriched over reagent and skin-flora background, and high-breadth rows more so. Report the tier with every count.
- Culture contaminants (such as Mycoplasma) and vector-derived E. coli signal dominate raw bacterial counts. Report `class`, `is_background` and `is_endogenous` beside every rate. Those organisms indicate contamination or host genome, not infection.
- `spike_in_control` (PhiX), `negative_control` and `bacterial_background` are controls or reagent flora. Keep them out of viral and bacterial totals.
- A flag means unitigs aligned across a sizeable share of a panel reference, with no raw-read, UMI or replicate confirmation.
- NULL means unmeasured, not absent. An empty list means measured and nothing found. Only the empty list supports an absence call.
- `kmer_mass` is a relative proxy within a run. Do not report it as an absolute count.

## Example requests

- "List cervical single-cell RNA-seq samples with HPV quantification."
- "Which organisms show up most often in human gut samples, and which are likely contaminants?"
- "Which tissues carry SARS-CoV-2 reads, and at what rate?"

## Explaining results

State the tier and filters used, the number of samples and studies, and the caveat in one plain sentence. For a biologist, that is usually "these samples contain sequences matching X, which is consistent with but does not prove its presence".

## Troubleshooting

- Cohort is far smaller than expected: the study or sample was never scanned, so this is a floor.
- 504 on a broad microbe query: narrow with organism or tissue and skip the retry.
- 404 from a detections download: the study is not in the scanned set.

## References

- seqout Pentimento overview: https://seqout.org/mcp
