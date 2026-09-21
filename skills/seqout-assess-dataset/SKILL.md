---
name: seqout-assess-dataset
description: Use when someone has a specific study (GSE, SRP, PRJNA, E-MTAB, CRA, ...) and asks whether it is usable, what it contains, how good its metadata is, whether it is worth reanalyzing, or whether it has single-cell, long-read or microbial signal. Requires the seqout MCP server.
author: Saket Lab
license: MIT
metadata:
  version: "0.1.0"
  category: bioinformatics
  tags: [seqout, metadata, quality, reuse, enrichment]
---

# Assess a dataset with seqout

## Overview

Answer "can I reuse this study, and what should I watch for?" from seqout's layered metadata: what the submitter wrote, what seqout normalized from it, and what seqout measured in the reads themselves.

## When to use

- Before committing time to a public dataset.
- To check whether groups, replicates and batches can be recovered from the metadata.
- To compare two candidate studies.

## Workflow

### 1. Resolve the accession

Use `seqout-resolve-accessions` when the type is unclear. Child accessions go through `resolve_accession_project` and PRJ accessions through `resolve_prj_project`.

### 2. Gather, in this order

1. `get_project`: title, abstract, organisms, dates, centre.
2. `get_project_cite` or the publication fields: is there a linked paper, and how often is it cited?
3. `get_project_metadata_rows`: the raw archive metadata. Check which `sample_attribute:*` columns are filled, whether groups and replicates are recoverable, and the layout, platform and instrument.
4. `get_project_enriched`: normalized tissue, cell type, disease and assay with ontology IDs. A 404 means the study has no enrichment, so continue with raw metadata.
5. `get_project_single_cell_status`, then `get_project_single_cell`: only when the data looks single-cell. A 404 is common and means the study is absent from the read-derived set.
6. `get_project_longread_chemistry`: only for PacBio or Nanopore runs.
7. `get_project_xref` and `get_supplementary_files`: which raw reads exist (through a linked SRA/ENA study) and which processed files exist.
8. `get_project_similar`: neighbouring studies for context.

Skip steps that clearly do not apply.

### 3. Read each layer with its limits

- The archive text is what the submitter wrote. The enriched layer is a model-extracted normalization of that text and can be wrong. Quote `low_confidence_tags` when present, and name the layer each claim came from.
- In read-derived calls, `null` or `measurable` false means unmeasured. An empty list means measured and nothing found. Only the empty list supports an absence call.
- For single-cell counts, `cells` is a matrix column count. When `unfiltered` is set the columns are barcodes, so do not sum them or report them as cells. Compare against `n_samples_detailed` to see whether the sample list was complete.
- Microbial flags are evidence, not a diagnosis. See `seqout-microbe-signals`.
- Long-read `chemistry_confidence` is `exact` (BAM header), `declared` (depositor text), `bucket` (inferred from the instrument model alone, labelled "estimated" in some views) or `unknown`. Report the tier next to the call.
- GSA-Human (HRA, HRR, HRS) data is usually access-controlled.

### 4. Deliver a verdict

Summarize in five lines: what the study profiled, sample structure (groups, replicates, batches), data availability (raw and processed), metadata quality, and the specific caveats you found. End with a plain reusability judgment: ready to use, usable after cleaning (say what), or needs the original paper or authors.

## Example requests

- "Is GSE123456 usable for a differential expression reanalysis?"
- "Does this study have raw FASTQs, or only a counts matrix?"
- "Compare these two candidate datasets and pick one."

## Explaining results

Lead with the verdict and what the samples are (tissue, condition, how many per group). Describe each caveat as what it could do to a conclusion. For technical readers, add the column list, platform and instrument, layout and the tool fields behind each claim.

## Troubleshooting

- Enriched call returns 404: the study is not enriched. Judge from raw rows and say so.
- Group labels missing from metadata rows: read the abstract and linked paper, and flag the design as unverified.
- Sample counts disagree between tools: report both and name the source of each.

## References

- seqout MCP documentation: https://seqout.org/mcp
