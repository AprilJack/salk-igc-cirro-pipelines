# MethylSeq nf-core 4.2 (On-Demand high resources)

Cirro pipeline configuration for [nf-core/methylseq](https://github.com/nf-core/methylseq)
**4.2.0**, with larger resources for the long-running indexing and alignment steps.

## What is here

| File | Source |
|---|---|
| `process-form.json` | Unchanged from Cirro's public config, [`CirroBio/Cirro-pipelines` `nf-core/methylseq/4.2/`](https://github.com/CirroBio/Cirro-pipelines/tree/main/nf-core/methylseq/4.2) |
| `process-input.json` | Unchanged from Cirro's public config |
| `process-output.json` | Unchanged from Cirro's public config |
| `preprocess.py`, `test_preprocess.py` | Unchanged from Cirro's public config (samplesheet check) |
| `process-compute.config` | Cirro's public retry rule plus the resources below |

The run form is identical to Cirro's shared **MethylSeq (nf-core/methylseq)** pipeline.

## Resource overrides

All on the On-Demand queue (`PW_ONDEMAND_JOB_QUEUE`), so long jobs are not lost to Spot
reclaims. Every other step keeps the pipeline defaults (`conf/base.config`) on Spot.

| Step | CPUs | Memory | Time |
|---|---|---|---|
| `BWAMETH_INDEX` | 24 | 192 GB | 24 h |
| `BWAMETH_ALIGN` | 48 | 192 GB | 3 d |
| `BISMARK_GENOMEPREPARATION` | 16 | 128 GB | 24 h |
| `BISMARK_ALIGN` | 48 | 192 GB | 5 d |
| `BISMARK_DEDUPLICATE` | 8 | 64 GB | 2 d |
| `BISMARK_METHYLATIONEXTRACTOR` | 16 | 64 GB | 2 d |

- `BWAMETH_ALIGN` passes all CPUs to bwa-meth (`-t task.cpus`).
- `BISMARK_ALIGN` sets `--multicore` to min(CPUs / 3, memory / 13 GB) for directional
  libraries (CPUs / 5, memory / 18 GB when non-directional), so 48 CPUs / 192 GB gives
  `--multicore 14`.
- `BWAMETH_INDEX` with `use_mem2` builds a BWA-MEM2 index of the doubled (C->T / G->A)
  genome, which needs roughly 150 GB of RAM for mouse.

## Why

On the public defaults (`process_high`: 12 CPUs / 72 GB on Spot), Bismark alignment of
~350 M read-pair EM-seq mouse samples ran at about 30 minutes per million reads and was
repeatedly interrupted by Spot reclaims before finishing.

## Registering in Cirro

Config source **EXTERNAL**: workflow code `nf-core/methylseq@4.2.0:main.nf`, config from
this repository, branch `main`, folder `nf-core/methylseq/4.2-ondemand`.
