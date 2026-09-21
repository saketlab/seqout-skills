---
name: seqout-build-cohort
description: Use when someone wants samples, not just studies, that match biological criteria across many public projects, e.g. "female human liver samples over 50 with cirrhosis", "cervical scRNA-seq samples with HPV", "all pancreatic islet samples", or an independent cohort to validate a finding. Requires the seqout MCP server.
author: Saket Lab
license: MIT
metadata:
  version: "0.1.0"
  category: bioinformatics
  tags: [seqout, cohort, samples, meta-analysis, ontology]
---

# Build a sample cohort with seqout

## Overview

Most searches return studies. A cohort question needs samples: "liver samples from women over 50 with cirrhosis" cuts across many studies and depends on per-sample fields such as age and sex. `search_enriched_samples` answers with samples, using seqout's ontology-normalized sample metadata.

Do not approximate it with `search_structured` plus opening each study. Those filters only prove that one matching sample exists somewhere in a study.

## When to use

- Validation cohorts, meta-analyses, "how many samples exist for X".
- Any question that mentions age, sex, treatment, cell type, strain or infection.

## Workflow

### 1. Turn the criteria into filters

Filter semantics change the cohort, so choose by type:

- Tissue, disease, cell_type, assay, phenotype, treatment, development_stage, strain, cell_line, ethnicity and similar fields match as case-insensitive substrings. A short stem (`cervi`, `papilloma`) works better than a long phrase.
- Organism, sex, taxid and study_accession match exactly.
- `disease_ontology_id`, `tissue_ontology_id`, `cell_type_ontology_id`, `assay_ontology_id` and `development_stage_ontology_id` take ontology IDs (for example MONDO:0005148) and include descendants by default. Get a MONDO id with `get_disease_aliases` and check a term with `ontology_term`. Set `include_descendants=false` for the exact term only.
- `age_min_years` / `age_max_years` drop every sample without a recorded age. Report that the cohort is the subset that reported age.
- `min_cell_count` and related filters drop unmeasured samples, and the measured subset covers about 85k samples.
- For single-cell data, `has_matrix` means a parsed cell-by-gene matrix. `single_cell_only` is a stricter read-derived barcode call. State which one you used.
- For infection or microbes, use `microbe="papillomavirus"` or `microbe_class="pathogen"`. See `seqout-microbe-signals` before reporting.
- A filter set to `false` means "do not filter", not "absent".

At least one filter is required. If the query returns a 504, narrow it with organism or tissue and skip the retry.

### 2. Run and size it

Page with `limit` / `offset` until `next_offset` is null, or stop at a size the user needs and say so. Report the real `total`, the number of studies contributing and the largest contributors. `get_enriched_crosstab` and `get_enriched_column` give quick breakdowns of what the cohort contains.

### 3. Check the cohort before recommending it

Group rows by study and look for problems that break a meta-analysis:

- Mixed platforms, library strategies or layouts. Pull per-study details with `bulk_project_samples_metadata` for several projects at once.
- Duplicated samples. A GEO series and its SRA study describe the same samples, so deduplicate through `get_project_xref` before counting.
- Confounding by study. If condition and study line up (all cases from one study, all controls from another), say the design cannot separate them.
- Enriched, unverified metadata. Ontology-normalized fields are model-extracted and can be wrong. Spot-check a few rows against raw metadata via `get_project_metadata_rows` and `get_project_enriched` (`low_confidence_tags`).
- Ethnicity is the submitter's verbatim string, present on a minority of studies, and says nothing about inferred genetic ancestry.
- Exclusions. Structured search cannot express NOT, so filter excluded groups from the returned rows yourself and say what you removed.

### 4. Deliver

Provide a cohort summary (n samples, n studies, organism, assay, tissue and disease mix), the exact filter set for reproducibility, the caveats above that apply, and a sample-level table or accession list. Offer `seqout-download-prepare` for retrieval.

## Example requests

- "List human liver samples annotated with hepatocellular carcinoma with their tissue, cell type and disease IDs."
- "I have bulk RNA-seq from lupus PBMCs. Find other human blood datasets from lupus patients for validation."
- "How many pancreatic islet samples exist, and in how many studies?"

## Explaining results

Describe who is in the cohort in plain terms, whether it is big and balanced enough for the question, and what could bias it. For technical readers, add the filter set, the per-study table with platform and batch structure, and the columns needed to build a design matrix.

## Troubleshooting

- Cohort far smaller than expected: an age, cell-count or microbe filter dropped unmeasured samples. Remove it and compare.
- Cohort far larger than expected: substring filters match broadly (`liver` also matches `liver, left lobe`), or an ontology ID pulled in descendants.
- Timeouts (504): add organism or tissue.

## References

- Ontologies used for enrichment: UBERON (tissue), CL (cell type), MONDO (disease), EFO (assay and other terms).
