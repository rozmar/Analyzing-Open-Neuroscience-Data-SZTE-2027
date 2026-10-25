# Course data: how it is built

[`00_Build_Course_Data.ipynb`](../00_Build_Course_Data.ipynb) - Built with Claude
Turns the original public releases of two Allen Institute Patch-seq
datasets into a handful of small, clean tables that the course notebooks load in seconds.
Anyone can rerun it and get the same files; every source, checksum and package version is
written to `provenance.json`.

| dataset | cells | ephys | morphology | transcriptome |
|---|---|---|---|---|
| Mouse V1 GABAergic interneurons, Gouwens, Sorensen et al. 2020 *Cell* | 4,435 (4,284 in the published analysis) | 4,284 | 510 | 4,435 |
| Human neocortex GABAergic interneurons, Lee, Dalley et al. 2023 *Science* | 778 | 704 | 140 | 778 |

## Run it

Open `Modul 2/00_Build_Course_Data.ipynb` and run all cells (~1 min, ~120 MB download, ~2 GB RAM peak).

* **Colab:** the first cell clones this repository and installs `anndata` and `rdata`.
* **Locally:** run `uv sync` once in the repository root and choose the `.venv` kernel.

The *Settings* cell has the options: `ONLY = "mouse"` or `"human"`, `SKIP_EXPRESSION`,
`SKIP_DANDI`, `OFFLINE` (use the `data/downloads/` cache only), `MOUSE_COUNTS = "nemo"` (use the
original NeMO CSV instead of the Allen parquet copy), `MOUSE_SPECIMENS` (another specimen export,
see below).

Always read `data/validation_report.txt` after a run: it lists every warning and the
counts/join rates the build achieved.

## What comes out (`data` folder)

| file | content |
|---|---|
| `cells.csv` | one row per cell, both species: ids, donor, area, layer, t-type, subclass, MET-type, modality flags, NWB/SWC links |
| `{mouse,human}_ephys_features.csv` | precomputed electrophysiology features (Allen) |
| `{mouse,human}_morph_features.csv` | precomputed morphology features (Allen) |
| `{mouse,human}_expression_selected.parquet` | log2(CPM+1) of ~185 genes (all voltage-gated ion channels + markers), pandas only |
| `mouse_expression.h5ad` | all 45,768 genes, raw counts, sparse (AnnData) |
| `human_expression.h5ad` | all 50,281 genes, log2(CPM+1) as published, sparse (AnnData) |
| `genes_selected.csv` | the selected genes, their mouse/human symbols and why they were chosen |
| `DATA_DICTIONARY.md` | every column of every table, with the original column name |
| `provenance.json`, `validation_report.txt` | sources, checksums, versions; checks |

Everything joins on `cell_id` (`mouse_601506507`, `human_868110130`).

## Where every number comes from

| key in `SOURCES` | source | status |
|---|---|---|
| `mouse_metadata` | Allen "Download cell metadata", brain-map.org > Multimodal Characterization of Individual Neurons | official |
| `mouse_file_manifest` | Allen raw-data manifest (NWB on DANDI, SWC on Brain Image Library, checksums) | official |
| `mouse_specimens` | Brain Knowledge Platform specimen table, project `1HEYEW7GMUKWIQW37BO` | **manual export**, see below |
| `mouse_counts_allen_parquet` | Allen EducationAndEngagement GitHub release `patchseq-data-v1` (parquet copy of the NeMO matrix) | official (Allen), default |
| `mouse_counts_nemo` | NeMO `20200513_Mouse_PatchSeq_Release_count.v2` | official original, optional |
| `human_*`, `vgic_genes` | `AllenInstitute/human_patchseq_gaba`, pinned to commit `5eb357a` | official |
| NWB files | DANDI API: dandiset 000020 (mouse, version 0.210913.1639), 000636 (human, latest published) | official |

**The one manual step.** The mouse t-types, MET-types and precomputed ephys/morphology
features live in the Brain Knowledge Platform specimen table, which has no stable download URL.
Export it once: open <https://knowledge.brain-map.org/data/1HEYEW7GMUKWIQW37BO/specimens>,
use the gear icon to show all columns, *Download Data > Metadata table*, unzip, and save the CSV
as `Modul 2/data/SpecimenMetadata.csv` (or set another path in
`MOUSE_SPECIMENS`). This file has already been downloaded and on this repository.

