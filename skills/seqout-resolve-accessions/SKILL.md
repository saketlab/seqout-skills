---
name: seqout-resolve-accessions
description: Use when someone gives an accession (GSE, GSM, SRP, SRX, SRR, PRJNA, E-MTAB, E-GEAD, CRA, HRA, DRP, ...) and wants to know what it is, its title and abstract, its parent study, its equivalent in another archive, or its samples and runs. Converts GSE to SRP to PRJNA and back. Requires the seqout MCP server.
author: Saket Lab
license: MIT
metadata:
  version: "0.1.0"
  category: bioinformatics
  tags: [seqout, accession, geo, sra, bioproject, arrayexpress]
---

# Resolve and convert accessions with seqout

## Overview

One dataset often has several names: a GEO series (GSE), an SRA study (SRP), a BioProject (PRJNA) and sometimes an ArrayExpress or DDBJ entry. seqout classifies any accession, finds the parent study for child records (samples, experiments, runs) and lists the equivalents in other archives.

## When to use

- "What is GSE123456?" or "what does SRR1234567 belong to?"
- "Which SRA project matches this GSE?" or the reverse.
- "List the samples, experiments or runs of this study."

## Workflow

### 1. Classify

Call `classify_accession` when the type is unclear. It matches patterns only and is cheap.

### 2. Route by accession type

| Accession | Meaning | Start with |
|---|---|---|
| GSE | GEO series | `get_project`, `get_geo_series_samples` |
| GSM | GEO sample | `get_sample_detail`, then `resolve_accession_project` for the parent GSE |
| SRP / ERP / DRP | Study | `get_project`, `get_study_experiments`, `get_study_samples`, `get_study_runs` |
| SRX / ERX / DRX | Experiment | `get_experiment`, `get_experiment_runs` |
| SRR / ERR / DRR | Run | `get_run_checksums`, `get_run` |
| SRS / ERS / DRS / SAM... | Sample or BioSample | `get_sample_detail` |
| E-XXXX-NNN | ArrayExpress experiment | `get_project`, `get_ae_experiment_samples` |
| PRJNA / PRJEB / PRJDA | BioProject | `resolve_prj_project`, then the study tools |
| SRA###### / ERA###### / DRA###### | Submission (a filing, not a study) | `resolve_submission_studies` |
| CRA / HRA, CRX / HRX, HRS, PRJCA | GSA (China National Center for Bioinformation) | `get_project`, `get_study_runs` on the CRA/HRA study |
| E-GEAD-N | DDBJ GEA expression archive | treat like a GEO series |

Child accessions (GSM, SRS, SRX, SRR) go through `resolve_accession_project` to find the parent project.

### 3. Convert between archives

- `get_project_xref` lists every equivalent at once. Use it when the user wants to see all names.
- `convert_accession` answers "the SRA study for this GSE" (`db=` narrows to one archive). `convert_accessions` is the batch form.
- `get_project_metadata` returns a fast title and description when nothing more is needed.

### 4. Report

State what the accession is, the parent project, the equivalents in other archives, and the counts of samples, experiments and runs when the user asked for them. Say which tool the answer came from when two archives disagree.

## Things to know

- DRP, DRX, DRS and DRR are DDBJ records and behave like their SRA counterparts. Their FASTQ links point at ddbj.nig.ac.jp.
- E-GEAD-N is not ArrayExpress despite the leading "E-". GEA holds no reads. Sequencing entries link to a DRA study through `alias` in `get_project`. Microarray entries have no FASTQ.
- CRR and HRR (GSA) runs have no per-run tool. Read them from `get_study_runs` on their CRA/HRA study. GSA-Human (HRA, HRR, HRS) data is usually access-controlled.
- `get_study_runs` previews the first 500 runs. For a specific run on a larger study use `find_study_runs`.
- `find_study_experiments` and `find_geo_series_samples` only confirm that specific child accessions exist. They stop at 500 rows with no total, so never treat them as a full listing. An empty result does not prove the accession is real.
- A GEO or ArrayExpress study has reads only when a linked SRA/ENA accession exists.

## Example requests

- "Which SRA project matches GSE123456?"
- "What is SRX1234567 and which study does it belong to?"
- "Convert these twenty GSE numbers to SRP accessions."

## Troubleshooting

- 404 or empty: check the prefix and digits, then try `classify_accession`. A GSM or SRR may be too new to be indexed. `get_last_updated` shows index freshness.
- No SRA link for a GSE: microarray and some processed-only series have none. Say so instead of guessing.

## References

- Accession conventions: https://seqout.org/mcp
