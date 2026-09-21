---
name: seqout-papers-authors
description: Use when someone wants to go from a paper to its datasets (PMID or DOI), from an author or lab to the data they deposited, get citation details or BibTeX for a study, or see which investigators work on a topic or in a country. Requires the seqout MCP server.
author: Saket Lab
license: MIT
metadata:
  version: "0.1.0"
  category: bioinformatics
  tags: [seqout, publications, citations, authors, bibtex, pubmed]
---

# Papers, authors and citations with seqout

## Overview

seqout links studies to PubMed records, so you can move in both directions: paper to data, and data to paper. It can also list who deposits data on a topic. Links exist only where a study is tied to a PubMed publication, so a study without a paper will not appear in any of these results.

## When to use

- "Which datasets belong to PMID 12345678?" or to a DOI.
- "What has this lab or author deposited?"
- "How do I cite GSE123456?"
- "Who works on CRISPR screens in India?"

## Workflow

- Paper to datasets: `find_publication` with a PMID or DOI.
- Author to datasets: `search_author_projects` returns project rows (search-card format) plus the author's institutes. Prefer it for "datasets by <name>".
- Author to papers: `search_by_author` returns one row per PMID (title, journal, DOI, citations) with the linked accessions.
- Dataset to paper and citation: `get_project_cite`. `type=original` returns the earliest linked PMID and `type=all` returns every linked publication. `format=bibtex` returns plain BibTeX.
- Related studies: `get_project_similar` for GSE or SRP accessions.
- Search results already carry `publications`, `pmid`, `journal`, `doi`, `authors`, `citation_count` and `center_name`. Use `sortby=citations|journal|year` on `search_all` or `search_structured` to rank by them.
- Investigators behind a topic: `get_search_investigators`. It ranks people for any search. `senior_only=true` ranks last authors (usually principal investigators) and false counts every author. Never pull `get_country_pis` and filter by topic yourself.
- Investigators by country with no topic: `get_country_pis`, for "who deposits the most from <country>".
- Valid filter values: `list_journals` and `list_centers`.

## Things to know

- Author and paper tools only find studies linked to PubMed. Recent or unpublished studies are missing by design.
- An author name can match several people. Show the institutes and ask before assuming one.
- Citation counts come from the linked PubMed data and may lag the journal.
- Give BibTeX as returned. Do not rewrite the fields.

## Example requests

- "Which datasets belong to the paper with PMID [PMID], and what else has its senior author deposited?"
- "Give me BibTeX for the paper behind GSE123456."
- "Who are the most active investigators for spatial transcriptomics in Germany?"

## Explaining results

List paper title, journal, year, DOI, citation count and linked accessions in one table. Say when a lookup found nothing because no PubMed link exists, and that this does not mean no data exists. Suggest `seqout-find-datasets` for a keyword search in that case.

## Troubleshooting

- Empty result for a known paper: the study may not be linked to its paper in the index. Search by title keywords instead.
- Many hits for a common name: filter by institute or year.

## References

- PubMed: https://pubmed.ncbi.nlm.nih.gov
