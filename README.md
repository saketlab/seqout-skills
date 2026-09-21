# seqout-skills

Claude Code plugin for [seqout.org](https://seqout.org). It connects the seqout MCP server and adds ten skills for finding, checking and retrieving public sequencing data from GEO, SRA, ENA, DDBJ, ArrayExpress and GSA. The server is read-only and needs no account or API key.

## Install

```
/plugin marketplace add saketlab/seqout-skills
/plugin install seqout@seqout
```

The plugin registers the MCP server at `https://seqout.org/api/mcp`. Local testing: `claude --plugin-dir ~/github/seqout-skills`.

## Skills

| Skill | Use it to |
|---|---|
| `seqout-find-datasets` | Turn a research question into a deduplicated list of studies. |
| `seqout-resolve-accessions` | Identify any accession and convert between GSE, SRP, PRJNA and other archive names. |
| `seqout-assess-dataset` | Judge whether one study is reusable and what to watch for. |
| `seqout-build-cohort` | Build a sample-level cohort across studies and check it for batch and duplicate problems. |
| `seqout-download-prepare` | Get manifests, checksums, scripts and sample sheets for a study. |
| `seqout-explore-disease-tissue` | See what data exists for a disease, rare disease or tissue, with ontology terms and synonyms. |
| `seqout-single-cell-long-read` | Find single-cell and long-read studies and read their quality metrics. |
| `seqout-microbe-signals` | Look for viruses, bacteria and contamination in read-derived detections, with the right caveats. |
| `seqout-papers-authors` | Go from a paper or author to datasets, and get citations and BibTeX. |
| `seqout-archive-stats` | Answer growth, organism, platform, archive-overlap and country questions. |

Start with `seqout-find-datasets` for a research question, or `seqout-resolve-accessions` if you already have an accession.
