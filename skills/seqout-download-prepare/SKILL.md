---
name: seqout-download-prepare
description: Use when someone wants to download or reanalyze data from a public study: FASTQs, SRA files, checksums, processed or supplementary files, a sample sheet, or an input list for nf-core/fetchngs. Produces a sized, verifiable retrieval plan and never starts a large download. Requires the seqout MCP server.
author: Saket Lab
license: MIT
metadata:
  version: "0.1.0"
  category: bioinformatics
  tags: [seqout, fastq, sra, download, checksums, fetchngs]
---

# Prepare downloads with seqout

## Overview

Take an accession to a verified, ready-to-run retrieval plan: what to download, how large it is, how to check it, and how to describe the samples. seqout returns links, sizes, checksums and scripts as text. It downloads nothing and changes nothing.

## When to use

- "Get me the FASTQs for this study" or "download the count matrix".
- "Give me a sample sheet", "checksums" or an nf-core/fetchngs input.

## Workflow

### 1. Resolve to a study

Use `classify_accession`, then `resolve_accession_project` (child accessions), `resolve_prj_project` (PRJ) or `resolve_submission_studies` (SRA/ERA/DRA submissions). Study-level download tools need SRP, ERP or DRP (or CRA/HRA for GSA).

### 2. Decide what is needed

- Raw reads: FASTQ lives at run level. A GSE or E-MTAB has reads only through a linked SRA/ENA study (`get_project_xref`, or `alias` for GEA). Say so when no link exists.
- Processed files (counts matrices, peaks): `get_supplementary_files`, and `download_supplementary_script` for a curl script. This is the only source for most microarray studies.
- Original BAMs: `get_study_bams`.
- Metadata only: `get_project_metadata_rows`.

### 3. Size it first

Call `get_study_runs` and report `total_runs`, `paired_runs`, `single_runs` and `total_fastq_bytes`. State the total size before producing a script, and ask first when it is large. To take a subset, use `find_study_runs`, which searches every run and caps at 500 with `capped` set. Never page `get_study_runs` for that.

### 4. Produce the retrieval artifact

- To inspect or transform data, prefer the JSON tools (manifest, checksums, metadata rows). Produce a script or plain accession list only when the user asks for one.
- Structured manifest: `get_study_download_manifest(study, mode="fastq")`, paged with `next_cursor` until the wanted files are covered. Each file carries `url`, `bytes`, `md5` and `download_command`.
- Ready-to-run script: `download_study_script(study, mode)` with mode `fastq` (default), `sra`, `sra_lite` (smaller, quality scores are lossy), `s3` or `gcs`.
- Single run: `get_run_checksums`.
- A `PAIRED` run with one FASTQ file (`fastq_file_count=1` or `fastq_is_interleaved=true`) is not a full R1/R2 pair. The manifest and script modes handle the conversion when `fastq_conversion_available` is true. Otherwise fetch normalized SRA first (SRA Lite if that is missing) and convert with `sracha get --split split-files`. Report `fastq_conversion_required` and `expected_fastq_paths`.
- Checksums: point to `md5` in the manifest and give a `md5sum -c` command for verification after download.

### 5. Sample sheet and pipeline handoff

Build the sample sheet from `get_project_metadata_rows`, one row per run for SRA/ENA and one per sample for GEO and ArrayExpress. Keep condition and replicate columns, and say which `sample_attribute:*` columns look usable as groups. For nf-core/fetchngs, supply a one-column list of run accessions. For nf-core/rnaseq and similar pipelines, map metadata columns to the pipeline's sample sheet fields and state your assumptions (strandedness is rarely recorded).

## Guardrails

- Confirm free disk space and the destination directory with the user before any script is run.
- GSA CRR and HRR runs have no per-run tool. Read them from `get_study_runs` on the CRA/HRA study. GSA-Human data is typically access-controlled and may have no links.
- DDBJ FASTQ links point at ddbj.nig.ac.jp.
- Whole-search CSV exports and bulk metadata downloads are REST-only. Send the user to https://seqout.org/api-docs or the web UI.

## Example requests

- "Which SRA project matches GSE123456? Give me a script that downloads its FASTQ files with checksums."
- "How big is PRJNA000000 in FASTQ, and is there a processed counts matrix instead?"
- "Make a sample sheet for this study with tissue, treatment, layout and FASTQ links."

## Explaining results

Explain FASTQ versus processed files in one line, recommend the smaller route that answers the question (often the processed matrix), and give one copy-paste command block with the size stated. For technical readers, add the manifest, checksum command, sample sheet, fetchngs input and the layout and instrument fields the pipeline needs.

## Troubleshooting

- No FASTQ for a GEO study: look for a linked SRA study with `get_project_xref`. Microarray studies legitimately have none.
- Only one FASTQ for a paired run: it is interleaved, see step 4.
- Very large study: offer a subset via `find_study_runs`, or the processed files.

## References

- nf-core/fetchngs: https://nf-co.re/fetchngs
