# Integrate per-species pangenome resources (graph + chains + Pv4 signal tracks + HyPhy selection) into the catalog and UI

## Goal

Add a new top-level resource type — a per-species **pangenome bundle** — to BRC-Analytics. Each bundle is a set of references for one species with derived cross-isolate datasets:

- Pangenome graph (GFA)
- Pairwise liftover chain files between member references
- Cohort VCF projections (e.g., MalariaGEN Pv4 → every Pv reference)
- Single-copy ortholog table + per-ortholog CDS alignments
- Population-genomic signal tracks (iHS, F<sub>ST</sub>, π, Tajima's D, allele frequency by country) as BigWig/BigBed
- HyPhy/Datamonkey selection-analysis outputs (BUSTED[S], FEL, MEME, aBSREL, RELAX, Contrast-FEL, MK α) as BigBed + JSON

First target: **`plasmodium-vivax-v1`** — 9 vivax-clade species, anchored on PvP01, signal tracks derived from MalariaGEN Pv4 (1,895 samples; ~1,210 monoclonal after F<sub>WS</sub>>0.95 filter). Pf8 follows once the Pv pipeline is shaken out.

## Mockup

PangenomeView for `plasmodium-vivax-v1` — `.scratch/pangenome-mockup.html` in branch `claude/add-pathogen-pangenomes-TgXTa`. Rendered:

![Pangenome view mockup — Plasmodium vivax v1](https://raw.githubusercontent.com/nekrut/brc-analytics/claude/add-pathogen-pangenomes-TgXTa/.scratch/pangenome-mockup.png)

Mockup shows: hero strip with summary stats, tab bar (Graph / References / Signal tracks / Selection / Orthologs / Provenance), an ASCII placeholder genome-browser preview around *pvdbp* showing how popgen + selection tracks stack, the top selection-hits table, the member-references table with pairwise chain downloads, and the Provenance + Re-use panels.

## Visualization target: UCSC

Primary genome-browser visualization piggy-backs on the **existing BRC UCSC track hub** (`https://hgdownload.soe.ucsc.edu/hubs/BRC/assemblyList.json`) and the existing UCSC integration in this codebase:

- `assemblies.json` already carries `ucscBrowserUrl` per assembly (`catalog/build/ts/build-assemblies.ts:92`).
- `app/components/Entity/components/.../RelatedTracksStep/hooks/UseUCSCTracks/` already discovers UCSC tracks for an assembly and renders them in a `GenomeBrowser` component inside the workflow configurator.
- `OutbreakResourceType` already has `HGPHYLOPLACE` (UCSC tool) → adding pangenome tracks to the hub fits the existing pattern.

This means **no new in-house browser component**. All pangenome-derived tracks register as UCSC composite tracks; the assembly page picks them up automatically; PangenomeView just builds parameterized `hgTracks` URLs.

## Data model

New entity `Pangenome` (LinkML, in `catalog/py_package/catalog_build/schema/pangenomes.yaml`):

```yaml
Pangenome:
  attributes:
    id:                     # "plasmodium-vivax-v1"
    species_taxonomy_id:
    version:                # "2026-01"
    reference_anchor:       # canonical accession (PvP01)
    member_assemblies:      # list of accessions, must exist in assemblies.yml
    graph:                  # PangenomeArtifact (GFA1)
    chain_files:            # list of ChainFile
    population_signal_tracks:   # list of SignalTrack
    selection_tracks:       # list of SelectionTrack
    ortholog_table:         # PangenomeArtifact
    cds_alignments:         # PangenomeArtifact

SignalTrack:
  attributes:
    assembly_accession:
    signal:                 # SignalType enum: IHS|XP_EHH|FST|TAJIMAS_D|PI|ALLELE_FREQUENCY
    format:                 # TrackFormat: BIGWIG|BIGBED|VCF_TABIX
    url:                    # bigDataUrl on datacache
    stratification:         # {partition_by, values}
    provenance:             # {source_dataset, source_version, sample_count, filter, workflow_url}
    ucsc_track_id:
    ucsc_visibility:        # hide|dense|squish|pack|full
    ucsc_composite_parent:
    ucsc_color:

SelectionTrack:
  attributes:
    assembly_accession:
    method:                 # SelectionMethod: BUSTED|BUSTED_S|MEME|FEL|ABSREL|RELAX|CONTRAST_FEL|FUBAR|FADE|PRIME|MK_ALPHA
    url:
    format:
```

New enums in `catalog/py_package/catalog_build/schema/enums/`:
- `signal_type.yaml`
- `selection_method.yaml`
- `track_format.yaml`

Schema entrypoint (`schema.yaml`) gains `- ./pangenomes` import.

New source YAML: `catalog/source/pangenomes.yml`. Hand-edited entries list `id`, `species_taxonomy_id`, `reference_anchor`, `member_assemblies`. Track lists (`population_signal_tracks`, `selection_tracks`, `chain_files`) are **auto-populated** by the build step from a manifest on datacache, not hand-curated.

Extend `OutbreakResourceType` enum with:
- `PANGENOME`
- `UCSC_TRACK_HUB`
- `SELECTION_ANALYSIS`

## Build pipeline

In `catalog/build/py/build_files_from_ncbi.py`, add:

```python
PANGENOMES_PATH = "catalog/source/pangenomes.yml"
PANGENOMES_OUTPUT_PATH = "catalog/output/pangenomes.json"
ASSEMBLY_ARTIFACTS_OUTPUT_PATH = "catalog/output/assembly-artifacts.json"
```

New step in `catalog/py_package/catalog_build/` (`build_pangenomes`) that:

1. Loads `pangenomes.yml`.
2. For each pangenome, fetches `https://datacache.galaxyproject.org/brc/data/pangenomes/<id>/manifest.json` and populates `chain_files` / `population_signal_tracks` / `selection_tracks` from files actually present.
3. Cross-validates `member_assemblies` against `assemblies.yml`, writes problems to `catalog/output/qc-report.data.md`.
4. Emits `pangenomes.json` + `.d.ts`.
5. Emits a reverse index `assembly-artifacts.json` — keyed by accession → `{pangenome_id, chains[], signal_tracks[], selection_tracks[]}`. Frontend reads this on assembly pages without scanning all pangenomes.

New step `publish_ucsc_hub`: emits per-assembly `trackDb.txt` stanzas (composite tracks grouped by `popgen` and `selection`) and writes them to the BRC hub layout. Example stanza:

```
track brc_popgen_pv_v1
compositeTrack on
shortLabel Pv4 popgen
longLabel  MalariaGEN Pv4 population-genomic signals (n=1,210 monoclonal)
type bigWig
group brc_pangenome
visibility full

    track brc_popgen_pv_v1_ihs
    parent brc_popgen_pv_v1 on
    shortLabel iHS
    longLabel  iHS · Pv4 monoclonal
    bigDataUrl https://datacache.galaxyproject.org/brc/data/pangenomes/plasmodium-vivax-v1/signals/per_reference/GCA_900093555.2/ihs.bw
    type bigWig
    color 31,111,235
    autoScale on
```

`subGroups` matrix selector handles AF-by-country/year stratification.

Wire both new steps into `npm run build-brc-db`.

## Frontend

### New view: `app/views/PangenomeView/`

Mirrors `PriorityPathogenView/`. Tabs:
- **Graph** — embedded thumbnail/static visualization + "Open in UCSC Browser" button (built URL with `hubUrl`, `position`, default visibility per track).
- **References** — table of member assemblies with chain-file download cells; per-row "View this reference in UCSC" link.
- **Signal tracks** — list grouped by signal type; download + UCSC-track-id chips.
- **Selection** — sortable table: BUSTED q, RELAX K, aBSREL+ branches, MEME-positive site count, MK α. Filter by gene-family tag. Drill-down opens HyPhy JSON in HyPhy-Vision.
- **Orthologs & alignments** — searchable ortholog browser with per-orthogroup FASTA download.
- **Provenance** — reproducibility metadata (source cohort, sample filter, workflow versions, generation date).

Routes: `pages/data/pangenomes/index.tsx` and `pages/data/pangenomes/[pangenomeId].tsx`.

### Assembly page extension

Add `app/views/EntityView/assembly/components/PangenomeResources/` slotted into the existing `Side` section. Reads `assembly-artifacts.json`. Shows:
- "Member of pangenome: …" link
- Available signal tracks (chips)
- Available selection tracks
- Chain files to/from this assembly
- "Open in UCSC with pangenome tracks" button

The existing `RelatedTracksStep` already discovers UCSC tracks for an assembly — once pangenome tracks are added to the hub, they show up there with no code change.

### Outbreak pages

`PriorityPathogenView` already renders `OutbreakResource[]`. With the new resource types, a single entry per outbreak markdown gets the pangenome surfaced:

```yaml
- title: "BRC pangenome: Plasmodium vivax v1"
  url: "/data/pangenomes/plasmodium-vivax-v1"
  type: PANGENOME
- title: "UCSC Genome Browser — Pv pangenome v1 tracks"
  url: "https://genome.ucsc.edu/cgi-bin/hgTracks?genome=GCA_900093555.2&hubUrl=https://hgdownload.soe.ucsc.edu/hubs/BRC/hub.txt"
  type: UCSC_TRACK_HUB
```

Eventually `catalog/source/outbreaks/plasmodium_vivax.md` lands as a new outbreak page.

### Sunburst

Decorate species nodes in the homepage sunburst (added v0.11) with a small badge when a pangenome is available, keyed by `species_taxonomy_id`. Drives discovery.

## Workflow integration

Two new IWC-registered workflows surface in AnalyzeView via the existing `workflows.yml`/`iwc-manifest-to-workflows-yaml` machinery:

1. **`pangenome-population-signals`** — input: cohort VCF + chain files + species reference set → output: per-reference BigWigs + a manifest JSON consumed by `publish_ucsc_hub`. Category: new `POPULATION_GENOMICS`.
2. **`hyphy-selection-screen`** — input: CDS alignments + species tree + branch partition → output: per-gene HyPhy JSON + selection.bb. Category: new `SELECTION_ANALYSIS`.

Two new entries in `workflow_categories.yml`:

```yaml
- category: "POPULATION_GENOMICS"
  name: "Population genomics"
  description: "Compute iHS, F_ST, Tajima's D, π from cohort VCFs and project to references."
  show_coming_soon: false

- category: "SELECTION_ANALYSIS"
  name: "Selection analysis"
  description: "HyPhy / Datamonkey CDS-alignment workflows (BUSTED, FEL, aBSREL, RELAX, MK α)."
  show_coming_soon: false
```

Two new entries in `workflow_parameter_variable.yaml` so the workflow stepper can offer pangenome inputs at launch time:
- `PANGENOME_GRAPH_URL`
- `CHAIN_FILE_URL`

## Datacache layout

Mirrors the existing `/brc/data/genomes/<accession>/...` convention:

```
datacache.galaxyproject.org/brc/data/pangenomes/<pangenome-id>/
├── manifest.json
├── graph.gfa.gz
├── chains/<refA>__to__<refB>.chain.gz
├── orthologs/orthogroups.tsv.gz
├── cds_alignments/<orthogroup>.fa.gz
├── selection/
│   ├── busted_summary.tsv.gz
│   ├── per_reference/<accession>/selection.bb
│   └── hyphy_json/<orthogroup>.json.gz
└── signals/
    └── per_reference/<accession>/
        ├── ihs.bw
        ├── fst_<comparison>.bw
        ├── pi.bw
        ├── tajimas_d.bw
        └── af_by_country.bb
```

UCSC hub:

```
brc-hub/
├── hub.txt
├── genomes.txt
└── <ucsc_db>/
    └── trackDb.txt          # auto-generated; bigDataUrls point at datacache
```

## Suggested PR sequence

| PR | Scope | User-visible? | Risk |
|----|-------|--------------|------|
| **1** | Schema + enums + empty `pangenomes.yml` + `build_pangenomes` emitting `pangenomes.json` & `assembly-artifacts.json`. No UI. | No | Very low — purely additive. |
| **2** | `publish_ucsc_hub` step; `trackDb.txt` generation; data ingest of `plasmodium-vivax-v1` artifacts to datacache; outbreak markdown link to UCSC. | Tracks visible in UCSC for Pv references | Low — text manifests only. |
| **3** | `PangenomeView` route + tabs; `PangenomeResources` section in assembly entity page; sunburst badge. | Full UI lit up | Medium. |
| **4** | New `workflow_categories` (POPULATION_GENOMICS, SELECTION_ANALYSIS); IWC workflow registration; stepper support for new parameter variables. | Workflows launchable | Medium — Galaxy integration. |

Each PR mergeable independently.

## Open questions

1. **Pangenome QC artifacts** — surface in `qc-report.data.md` (current pattern) or a separate per-pangenome JSON on datacache rendered in the Provenance tab? Lean: latter, with summary lines in the existing report.
2. **Versioning** — tie pangenome version to (a) source-cohort version (Pv4 → Pv5 forces new `id`) or (b) graph version? Lean: include both as separate fields so they can evolve independently.
3. **`assembly-artifacts.json` shape** — single file or per-species shards? Lean: single file until it exceeds ~5 MB.
4. **WHO Mtb catalogue v2 license** — CC-BY-NC-SA 3.0 IGO. Confirm non-commercial constraint is OK before re-hosting Mtb-derived tracks downstream.
5. **JBrowse2 as secondary?** — UCSC is primary; JBrowse2 stays out of scope unless there's a specific need (e.g., a track type UCSC can't render).

## Companion strategic context

Two strategic issues drafted for the `BRC-research` repo describe the broader gap-filling roadmap and the primary-dataset ingest map (full markdown in `.scratch/issue-1-roadmap.md` and `.scratch/issue-2-ingest-map.md` on this branch). This `brc-analytics` issue covers the implementation slice for the first deliverable in that roadmap.
