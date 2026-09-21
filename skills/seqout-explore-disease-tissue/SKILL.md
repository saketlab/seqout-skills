---
name: seqout-explore-disease-tissue
description: Use when someone asks what public data exists for a disease, a rare disease or a tissue ("what data exists on Rett syndrome", "fatty liver datasets with FASTQ", "pancreatic islet studies"), or wants ontology terms, synonyms and what the corpus holds by tissue, cell type or assay. Requires the seqout MCP server.
author: Saket Lab
license: MIT
metadata:
  version: "0.1.0"
  category: bioinformatics
  tags: [seqout, disease, tissue, ontology, mondo, uberon, rare-disease]
---

# Explore a disease or tissue with seqout

## Overview

A keyword search misses studies whose samples were annotated with a narrower term, such as "hepatic steatosis" when you asked for "fatty liver". seqout resolves disease names through MONDO and tissue names through UBERON, including descendant terms, then rolls up the studies. This skill uses those rollups.

## When to use

- "What public data exists on <disease or tissue>?"
- Surveying a rare disease, including alternative names and which studies have FASTQ.
- Checking what terms and synonyms submitters use, or what fraction of the corpus is annotated.

## Workflow

### Disease

1. `get_disease_aliases(q)` turns a name into MONDO ids and lists alternative names. It returns `total` and `truncated`, so raise `limit` (max 100) or narrow `q` when truncated.
2. Choose the collection for the rollup:
   - `collection=rare` (NIH GARD, exact MONDO cross-reference) or `collection=nord` (NORD, matched by name, a looser and larger set). Say which you used. They are not interchangeable.
   - A free-text disease name or MONDO id, for example `collection="fatty liver disease"`. The server matches the full MONDO ontology (exact label first, else substring) plus descendants. `resolution` and `matched_labels` in the summary show what matched. A 404 means nothing in MONDO matches, so try a broader or differently spelled term.
3. `get_disease_summary(collection)` for totals, `get_disease_facets(collection)` for valid filter values (call it before projects), then `get_disease_projects(collection, ...)` for one row per study.
4. When the user wants the samples, pass the MONDO ids to `search_enriched_samples(disease_ontology_id=...)` (see `seqout-build-cohort`).

### Tissue

`get_tissue_summary(term)`, `get_tissue_facets(term)` and `get_tissue_projects(term, ...)` work the same way with UBERON, so "blood" also reaches "whole blood" and "peripheral blood mononuclear cell". A 404 means nothing in UBERON matches.

### Ontology terms and corpus profile

- `ontology_term(name)` shows a term's identifiers (UBERON, CL, MONDO, EFO, HGNC, MeSH, Cellosaurus), its synonym cluster and its direct children. Look it up by name. It does not resolve an ID such as MONDO:0005148. Pass `children=false` when you only need identifiers and synonyms. A 404 means no node carries that exact name, so try another spelling.
- `get_enriched_column(column=...)` gives top values for tissue, cell_type, disease or assay. `get_enriched_crosstab(group=..., breakdown=...)` cross-tabulates, for example organism by tissue. `get_enriched_coverage` and `get_enriched_stats` show how much of the corpus is annotated.

## Reading rare-disease rows

- `scope` defaults to `human_primary` (human material that is neither an immortalized line nor a derived model). `patient_derived_model`, `cell_line` and `all` show what that excludes. Name the scope behind every count.
- `n_samples` counts samples matching this collection, not the study total. Controls and out-of-scope samples are excluded.
- A null cell count means unmeasured, not zero. Blank ancestry means unrecorded.
- `has_fastq` and `has_sra` are NULL when the study is absent from the download-links table. That means unknown, not unavailable.
- `ancestries` is the submitter's ethnicity string on about 13% of studies. It is not inferred genetic ancestry.
- `inheritance` comes from HPO, Orphanet and GARD. "Not applicable" means the disease has no Mendelian pattern, not that nobody looked.
- Enriched metadata is model-extracted. Treat totals as a lower bound on what exists.

## Example requests

- "Which public datasets exist for Rett syndrome, what other names does it go by, and which have FASTQ?"
- "What are the child terms of 'T cell', and which synonyms do submitters use for CD8+ T cells?"
- "Which tissues are most common in mouse studies, and how much of the corpus is enriched?"

## Explaining results

Say plainly what was searched (which collection or term, which scope) and how many studies and samples matched. Recommend a few studies and say why. Mention that counts reflect annotated samples, so real coverage is probably larger.

## Troubleshooting

- 404 on a disease or tissue: try the synonym from `get_disease_aliases`, a broader term, or another spelling.
- Filter rejected on a collection: call the matching `_facets` tool for valid values.
- Too few studies: also run `seqout-find-datasets`, since studies without enrichment only appear in keyword search.

## References

- MONDO (disease): https://mondo.monarchinitiative.org, UBERON (anatomy): https://obophenotype.github.io/uberon
