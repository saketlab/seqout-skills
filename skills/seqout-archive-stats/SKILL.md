---
name: seqout-archive-stats
description: Use when someone asks about the shape of public sequencing data rather than one study: growth over time, which organisms or platforms dominate, how archives overlap, how fresh the index is, or what a country contributes. Examples: "how has Nanopore data grown", "top organisms by experiments", "what does India deposit". Requires the seqout MCP server.
author: Saket Lab
license: MIT
metadata:
  version: "0.1.0"
  category: bioinformatics
  tags: [seqout, statistics, trends, platforms, organisms, countries]
---

# Archive statistics and trends with seqout

## Overview

Questions about totals, growth and contributors are answered from precomputed statistics, which return quickly and need no search terms.

## When to use

- Growth of projects, experiments or data volume over time.
- Which organisms, platforms or instruments dominate, and how that changed.
- How the archives overlap and how current the index is.
- A country's contribution, or a map of where data comes from.

## Workflow

- Size and freshness: `get_source_totals` (projects and samples per source, approximate) and `get_last_updated` (overall or per source: geo, sra, arrayexpress, ena, gsa, dra, gea).
- Archive overlap: `get_project_overlap` shows how many projects each archive holds and how they intersect, treating all cross-archive equivalents as one project.
- Growth: `get_growth_stats` gives monthly projects, experiments or data volume (`mode=projects|experiments|bases`). In `bases`, ENA reports nucleotide bases while SRA reports FASTQ and SRA bytes, so do not add the two. `get_organism_growth` follows one organism by its scientific name.
- Organisms: `get_organism_totals(limit=...)` ranks organisms by experiments with the source split. `search_organisms` autocompletes names. `get_common_names_by_scientific_names` resolves a list of common names in one call. Avoid listing every organism, since it returns a very large set.
- Platforms: `get_platform_totals`, `get_platform_growth`, `get_platform_instruments` and `get_platform_filters`.
- Annotation coverage: `get_enriched_coverage` shows how many projects and samples have enriched metadata per archive, organism and assay.
- Counts by field for a search: `search_structured(count_by=year|source|organism|country|library_strategy|instrument_model)`.
- Countries (ISO two-letter codes such as IN or US): `get_country_summary`, `get_country_facets`, `get_country_projects`, `get_country_accessions` (a plain accession list) and `get_country_points` (map points). Global view: `get_global_contributions` with `get_global_contribution_filters` for valid values.
- Investigators for a country or topic: see `seqout-papers-authors`.

## Things to know

- Country comes from the submitting institution and covers most archives, but not ArrayExpress.
- Counts are precomputed and refreshed periodically, and source totals are approximate. Quote `get_last_updated` next to any trend.
- Growth by month reflects release date in the archive, not when the experiment was run.
- Some statistics return 503 while being prepared. Retry later.

## Example requests

- "How has Oxford Nanopore data grown by month, and which organisms drive it?"
- "Top ten organisms by number of experiments, with common names."
- "What does India deposit, by assay and archive?"

## Explaining results

Give the number, the unit (projects, experiments, samples or bytes), the archives it covers and the date of the index. Offer a small table or a description of the trend. Do not draw conclusions about why a trend exists unless a source supports it.

## Troubleshooting

- Numbers differ from the archive's own website: totals here are deduplicated across archives and approximate.
- A country returns nothing: check that the code is two letters and that the country has geocoded submissions.

## References

- seqout: https://seqout.org
