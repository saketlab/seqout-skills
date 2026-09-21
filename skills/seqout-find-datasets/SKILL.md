---
name: seqout-find-datasets
description: Use when someone wants public sequencing or expression data for a research question, e.g. "find scRNA-seq of human pancreas in type 2 diabetes", "ChIP-seq for CTCF in mouse", "RNA-seq of zebrafish heart regeneration since 2021", "what data exists on cirrhosis". Searches GEO, SRA, ENA, DDBJ, ArrayExpress and GSA together. Requires the seqout MCP server.
author: Saket Lab
license: MIT
metadata:
  version: "0.1.0"
  category: bioinformatics
  tags: [seqout, geo, sra, search, public-data]
---

# Find public datasets with seqout

## Overview

Public sequencing data is spread over several archives that each use their own accession formats. seqout searches them together and links the records that describe the same study, so one question returns one list. This skill turns a research question into a ranked, deduplicated list of studies.

## When to use

- You have a question ("what public data exists on X?") and no accession yet.
- You want studies filtered by organism, assay, instrument, country, journal or date.
- You want to reproduce a search someone else described.

For a single accession you already have, use `seqout-resolve-accessions`. For samples rather than studies, use `seqout-build-cohort`.

## Workflow

### 1. Pin down the question

Extract organism, assay (RNA-seq, ATAC-seq, scRNA-seq, WGS, ChIP-seq), tissue or disease, and any date or platform limit. Ask only for what is missing and would change the search. Default to all archives and all years.

### 2. Pick the search tool

| The request looks like | Use |
|---|---|
| A full sentence | `search_nl` (`dry_run=true` first if the sentence is ambiguous) |
| Explicit filters (organism, assay, instrument, year, country, journal) | `search_structured` |
| Keywords or a boolean query | `search_all` |
| Sample properties, and the samples themselves are wanted | `search_enriched_samples`, see `seqout-build-cohort` |
| "What exists on <disease or tissue>" | `seqout-explore-disease-tissue` |
| Single-cell or long-read | `seqout-single-cell-long-read` |
| A paper, author or lab | `seqout-papers-authors` |

Look up valid filter values with `list_library_strategies`, `list_instrument_models`, `list_journals`, `list_centers` and `search_organisms` before guessing them. Both organism and library strategy accept several values, so "human and mouse RNA-seq" is one search.

### 3. Search discipline

- Reuse rows already in the conversation before searching again. Filter and sort them locally.
- Continue with the returned cursor. A changed `q` starts a new search. Relevance searches page with `cursor_rank` + `cursor_acc`, and `sortby` searches with `cursor_sort` + `cursor_acc`.
- Read `parsed.warnings` from `search_nl` and repeat every warning to the user. Exclusions such as "excluding mouse" are detected but not applied, so filter the rows yourself.
- Structured search cannot reach DRA-exclusive and GEA records. Use `search_all(db=dra)` or `search_all(db=gea)` when the user needs them.
- Some filters skip archives: `platform` skips GEO and ArrayExpress, `instrument_model` skips ArrayExpress, ENA and GSA, `country` skips ArrayExpress, and `multi_platform` covers GEO and SRA only. Tell the user when a filter narrowed the archives searched.
- `sample_tissue`, `sample_disease` and `sample_cell_type` only match studies that have enriched sample metadata, so a missing study may simply lack enrichment.
- `published_after` / `published_before` filter the release date. `pub_date_after` / `pub_date_before` filter the linked paper's date and drop studies without a paper.
- For counts instead of rows, use `search_structured(count_by=year|source|organism|country|library_strategy|instrument_model)`.
- Too few results on a plausible query: try `search_suggest` for misspellings. Use `search_expansion` when the user asks why a keyword search looks too broad or too narrow.

### 4. Deduplicate across archives

A GEO series and its SRA study describe one dataset. Use `get_project_xref` or `convert_accession`, and list each dataset once with its equivalent accessions in one row.

### 5. Report

Give the true `total`, the filters actually applied and the archives they covered. Show a compact table:

| Accession(s) | Title | Organism | Assay | Samples | Year | Paper / citations |

Rank by relevance unless the user asked for citations, year or journal (`sortby=citations|journal|year`). Offer the next step: assess a study (`seqout-assess-dataset`), build a cohort (`seqout-build-cohort`) or prepare downloads (`seqout-download-prepare`).

## Example requests

- "Human single-cell RNA-seq of pancreatic islets since 2022. Summarize the top three."
- "Mouse hippocampus RNA-seq after chronic stress. List accessions with organism and platform."
- "Which organisms other than human and mouse have ATAC-seq data?"

## Explaining results

Write for the person asking. Describe each hit in one plain sentence (what was profiled, in what, whether a paper exists), and explain terms such as "library strategy" or "accession" on first use. Recommend two or three studies and say why. When the user is clearly a bioinformatician, add the exact filter set so the search can be repeated and an accession list to paste.

## Troubleshooting

- Zero hits on a specific phrase: broaden to a stem, drop one filter at a time, or try `search_suggest`.
- Results look off-topic: `search_expansion` shows which synonyms reached the search.
- A known dataset is missing: it may be DRA or GEA only, or the filter you used skips its archive.

## References

- seqout: https://seqout.org, MCP documentation: https://seqout.org/mcp
