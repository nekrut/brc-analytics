# Primary-dataset ingest map for the gap-driven pathogen-resource roadmap

Companion to #<roadmap-issue>. Maps each proposed BRC resource to **specific, already-public primary datasets** that can be ingested, with URLs, formats, licensing notes, and a one-line ingest plan. The goal is to make it concrete what we'd download, how big it is, and what the first Galaxy workflow looks like.

Format per row: **Source dataset → format/scope → license → ingest plan.**

---

## Tier 1 — Pangenome companion resources

### 1.1 Liftover chains between references within a species

Generated, not ingested — but the **inputs** are the assemblies already in `catalog/source/assemblies.yml`.

- **Tool:** [nf-LO](https://github.com/evotools/nf-LO) (Nextflow, containerized; works for any pair of assemblies) or [levioSAM2](https://github.com/milkschen/leviosam2) for VCF-aware lifts.
- **Per-species scope:** N×(N-1)/2 chain files for N references; for Pf (~10 commonly used refs) that's ~45 chains. Trivial storage.
- **Ingest plan:** One Galaxy workflow `assemblies-pairs → minimap2 → chainNet → .chain.gz`, parameterized over `assemblies.yml`. Output: `chains/<species>/<refA>__to__<refB>.chain.gz` + a `manifest.tsv`.

### 1.2 Population-genomic signal tracks per reference

| Source | Format | Scope | License | Ingest |
|---|---|---|---|---|
| [MalariaGEN Pf8](https://www.malariagen.net/data_package/open-dataset-plasmodium-falciparum-v80/) | VCF + Zarr | 33,325 Pf samples, ~10.8M SNPs, 12M sites | CC-BY 4.0 | Pull VCF/Zarr → scikit-allel/hapne → BigWig per reference via liftover chains from 1.1 |
| [MalariaGEN Pv4](https://www.malariagen.net/resource/34/) | VCF + Zarr | ~1,895 Pv samples | CC-BY 4.0 | Same pipeline as Pf8 |
| [Ag1000G phase 3](https://www.malariagen.net/data_package/open-dataset-anopheles-gambiae-v30/) | VCF + Zarr | ~3,000 *An. gambiae*/*An. coluzzii* | CC-BY 4.0 | Same pipeline; project onto AgamP4 and any other An. references |
| [CRyPTIC Mtb 30k](http://ftp.ebi.ac.uk/pub/databases/cryptic/) | VCF + phenotype | ~15,000 Mtb genomes with MIC for 13 drugs | Public-domain via EBI | Aggregate AF + drug-MIC association tracks per H37Rv coords + lift to other Mtb refs |

**First deliverable:** Per-reference BigWig of iHS, XP-EHH, Fst (country-level), Tajima's D, π, plus country-binned allele frequency for AMR-relevant genes.

### 1.3 Harmonized eukaryotic-pathogen AMR catalog (WHO-style)

| Pathogen | Primary inputs | Notes |
|---|---|---|
| *P. falciparum* | [WWARN parasite clearance & Kelch13 surveyor](https://www.wwarn.org/tracking-resistance/k13-tracker), [Pf8 CNVs in *gch1*/*crt*](https://www.malariagen.net/article/pf8-released-a-major-update-for-the-worlds-largest-malaria-parasite-genomic-data-resource/), [MalariaGEN drug-resistance variant catalog](https://www.malariagen.net/resource/35/) | Kelch13, crt, mdr1, mdr2, dhfr, dhps, plasmepsin2/3, ap2-mu, pfubp1 |
| *A. fumigatus* | [A. fumigatus pangenome biorxiv 2026 (>1,000 isolates)](https://www.biorxiv.org/content/10.64898/2026.04.14.718449v1), [mitogenome × azole, npj AMR 2025](https://www.nature.com/articles/s44259-025-00083-6) | cyp51A TR34/TR46/L98H, HapE, Cox10, hmg1; accessory genes from supp tables |
| *C. auris* | [Global 12,644-genome C. auris dataset, biorxiv 2026](https://www.biorxiv.org/content/10.64898/2026.02.03.703534v1.full.pdf), [CDC Updated Genomic Epi (EID 2026)](https://wwwnc.cdc.gov/eid/article/32/5/25-0760_article), [associations w/ antifungal susceptibility (PMC10821368)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10821368/) | ERG11 (Y132F, K143R), FKS1 (S639F/Y/P), TAC1B, MRR1, ERG3 |
| *Cryptococcus neoformans* | [Global C. neoformans surveillance biorxiv 2025](https://www.biorxiv.org/content/10.1101/2025.11.11.687764v1.full), [Essential + fluconazole-resistance atlas (2025)](https://pubmed.ncbi.nlm.nih.gov/40402997/) | ERG11, AFR1, MDR1 |
| *Leishmania* spp. | [Imamura et al. genome diversity (eLife 2016 + updates)](https://elifesciences.org/articles/12613); MIL/SbV resistance loci literature | MRPA, AQP1, MIL-T |
| *M. tuberculosis* (already covered) | [WHO Mtb catalogue v2 (2023)](https://iris.who.int/handle/10665/374061) + [TB-Profiler db](https://github.com/jodyphelan/tbdb) | Adopt directly; track per-reference projection |

**Deliverable:** Per-reference VCF + BigBed; tiered confidence column following WHO's Group 1–5 schema.

---

## Tier 2 — Functional layers from supp-tables

### 2.4 Essentiality / CRISPR-fitness tracks

| Source | Scope | Format | Ingest |
|---|---|---|---|
| [Toxo genome-wide CRISPR (Sidik et al. 2016, Cell)](https://www.cell.com/cell/fulltext/S0092-8674(16)31070-4) + downstream studies | Per-gene fitness scores, RH and ME49 | Supp tables (xlsx) | Parse → BigBed per gene_id; lift to other Toxo refs |
| [Cryptosporidium in-vivo CRISPR screens, Nat Commun 2025](https://www.nature.com/articles/s41467-025-63012-1) | Pyrimidine salvage and other gene fitness | Supp tables | Same as above |
| [Cryptococcus essential + fluconazole atlas (Nat Microbiol 2025)](https://pubmed.ncbi.nlm.nih.gov/40402997/) | Genome-wide CRISPRi | Supp tables / GEO | Same |
| [Plasmodium piggyBac essentiality (Zhang et al. 2018, Science)](https://www.science.org/doi/10.1126/science.aap7847) | MIS score, mutability index, per gene | Supp + [PlasmoGEM](https://plasmogem.umu.se/pbgem/) | Parse → per-gene BigBed |
| [Mtb TraSH / Tn-seq compilation (DeJesus et al.)](https://github.com/mad-lab/transit) | Per-gene essentiality | TRANSIT outputs | Project on H37Rv → lift to other refs |

### 2.5 Mappability / GC / dark-genome tracks

- **Tool:** [GenMap](https://github.com/cpockrandt/genmap), [Umap/Bismap](https://bismap.hoffmanlab.org/) (concept), `bedtools nuc`.
- **Inputs:** Each FASTA in `assemblies.yml`.
- **Deliverable:** `mappability_k50.bw`, `gc_w100.bw`, `low_complexity.bed.gz` per reference. One workflow, runs over the whole catalog.

### 2.6 Subtelomere / TE / repeat annotation

| Source | Notes |
|---|---|
| [Dfam 3.8](https://www.dfam.org/) | TE HMMs across eukaryotes; includes fungi |
| [RepeatModeler2](https://github.com/Dfam-consortium/RepeatModeler) | De-novo libraries per assembly |
| [TEclass2](https://github.com/IOB-Muenster/TEclass2) | Classifies novel families |
| [PfDB6 var/rifin/stevor annotations (Otto et al.)](https://wellcomeopenresearch.org/articles/3-52) | Subtelomere virulence-family coords |
| [VSG-seq pipeline (Mugnier lab)](https://github.com/mugnierlab/find_polycistrons.py) | Trypanosoma surface-antigen repertoire |
| Template: [Z. tritici pangenome-guided TE library, PMC10652580](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10652580/) | Methodology to replicate per species |

### 2.7 SV / CNV catalogs

| Source | Scope |
|---|---|
| [Pf8 CNV calls in *gch1* and *crt*](https://www.malariagen.net/article/pf8-released-a-major-update-for-the-worlds-largest-malaria-parasite-genomic-data-resource/) | Per-sample copy number, with metadata |
| [C. auris clade-CNV paper (Munoz et al., PLOS ONE 2019 + updates)](https://pmc.ncbi.nlm.nih.gov/articles/PMC6785063/) | Per-clade conserved CNVs |
| [A. fumigatus aneuploidy × azole (mBio 2023)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10112202/) | Karyotype variation |

**Deliverable:** Per-reference BED of recurrent CNVs + sample-level CNV matrix.

---

## Tier 3 — Cross-omic layers

### 3.8 Strain-specific predicted structures

- **Inputs:** Each assembly's annotated proteome (GFF + FASTA from VEuPathDB/NCBI).
- **Tools:** [ColabFold/AF3](https://github.com/sokrypton/ColabFold), [ESMFold](https://github.com/facebookresearch/esm).
- **Deliverable:** Per-strain `.pdb` + pLDDT for each protein, indexed by gene_id; downloadable per-reference tarballs.
- **Note:** Skip proteins identical to AlphaFold-DB entries; only re-fold proteoforms that differ from the canonical reference. This makes the storage tractable.

### 3.9 MHC-I/II epitope tracks

| Source | Scope |
|---|---|
| [NetMHCpan-4.1 / NetMHCIIpan-4.3](https://services.healthtech.dtu.dk/services/NetMHCpan-4.1/) | Pan-HLA binding predictions |
| [IEDB Tools API](https://tools.iedb.org/main/tools-api/) | Programmatic batch |
| [HLA allele frequencies, allelefrequencies.net](http://www.allelefrequencies.net/) | Population weighting |

**Deliverable:** Per-reference BigBed of predicted binders with HLA weights for malaria-endemic, TB-endemic, and global populations.

### 3.10 Antigen-diversity hotspot tracks

- **Inputs:** Pf8 / Pv4 / CRyPTIC VCFs + per-gene CDS coords.
- **Tool:** Standard π / dN/dS sliding window ([SLAC, FUBAR via HyPhy](https://www.hyphy.org/)).
- **Deliverable:** BigWig of π and dN/dS per reference, with var/rifin/stevor (Pf), VSG (T. brucei), SAG/SRS (Toxo) tracks prominently surfaced.

### 3.11 Organellar variant catalogs

| Source |
|---|
| [Pf8 organellar VCF](https://www.malariagen.net/data_package/open-dataset-plasmodium-falciparum-v80/) (mitochondrial + apicoplast subsets) |
| [A. fumigatus mitogenome × azole study, npj AMR 2025](https://www.nature.com/articles/s44259-025-00083-6) |
| [Mtb mitochondrial-equivalent: 16S/23S rRNA variants from CRyPTIC](http://ftp.ebi.ac.uk/pub/databases/cryptic/) |

### 3.12 Cross-species ortholog projection track

- **Source:** [OrthoMCL 7](https://orthomcl.org/) (8.6M proteins, 716 proteomes, bi-monthly updates).
- **Deliverable:** Per-reference BED encoding ortholog group ID, single-copy-ortholog flag, lineage-specific expansion flag. Derived once per OrthoMCL release.

### 3.13 Host-pathogen PPI tracks

| Source | Scope |
|---|---|
| [HPIDB 3.0](https://hpidb.igbb.msstate.edu/) | Experimental host-pathogen PPIs |
| [Saluri et al. FoldDock-predicted pairs, 2024](https://www.biorxiv.org/content/10.1101/2024.04.05.588311v1) (and successors) | ~8,000 pathogen-human PPI pairs |
| [Li et al. AF-Multimer viral-human PPIs, 2025](https://www.biorxiv.org/content/10.1101/2025.06.04.657796v1.full) | >11,000 viral-human pairs |

**Deliverable:** Per-reference BigBed marking pathogen-protein residues predicted to interface with host proteins, with host UniProt ID + AF confidence.

---

## Tier 4 — Outbreak / surveillance

### 4.14 Reference-anchored phylogenies for non-viral priority pathogens

| Pathogen | Primary input | Tooling |
|---|---|---|
| *P. falciparum* | Pf8 VCF (33,325 samples) | [IQ-TREE 2](http://www.iqtree.org/) + [Augur/Auspice](https://nextstrain.github.io/auspice/) |
| *M. tuberculosis* | [CRyPTIC](http://ftp.ebi.ac.uk/pub/databases/cryptic/) + [TB-Profiler-collated](https://github.com/jodyphelan/tbdb) | [UShER](https://usher-wiki.readthedocs.io/) (mutation-annotated tree) |
| *C. auris* | [12,644-genome biorxiv 2026](https://www.biorxiv.org/content/10.64898/2026.02.03.703534v1.full.pdf) | IQ-TREE + Auspice |
| *C. neoformans* | [Global surveillance biorxiv 2025](https://www.biorxiv.org/content/10.1101/2025.11.11.687764v1.full) | Same |
| *Cryptosporidium* | [CryptoNet](https://www.cdc.gov/parasites/crypto/cryptonet.html) + published outbreak genomes | Same |

**Deliverable:** Newick + JSON metadata bundle per outbreak species, auto-rebuilt monthly. Hosted alongside existing `outbreaks/*.md`.

### 4.15 *Cryptosporidium* gp60 subtype reference panel

- **Tool:** [CryptoGenotyper](https://github.com/phac-nml/CryptoGenotyper) — wrap as Galaxy tool.
- **Reference data:** [2025 gp60 review (PMC12304688)](https://pmc.ncbi.nlm.nih.gov/articles/PMC12304688/) provides curated subtype panel.
- **Deliverable:** Reference subtype FASTA panel + Galaxy workflow + per-reference BED of gp60 locus on each *C. hominis*/*C. parvum* assembly.

### 4.16 Vector-pathogen-host overlay

- **Inputs:** Ag1000G phase 3, Pf8, human Duffy/HbS/G6PD allele frequencies (gnomAD + [HbVar](http://globin.bx.psu.edu/hbvar/)).
- **Deliverable:** A unified browser view: insecticide-resistance loci (kdr, GSTe2, Cyp6P) on AgamP4 + transmission-relevant Pf loci on Pf 3D7 + co-located host-genetic variants, all reachable via existing outbreak-page UI.

---

## Tier 5 — Cheap utility tracks

| Resource | Source / tooling |
|---|---|
| 5.17 Assembly QC + ploidy | Extend the v0.10 ploidy check (`#366`) into a public QC manifest |
| 5.18 Codon usage | [Codon Usage Database](http://www.kazusa.or.jp/codon/) + [coRdon](https://bioconductor.org/packages/coRdon/) per assembly |
| 5.19 Conservation tracks | [phyloP/phastCons](http://compgen.cshl.edu/phast/) over OrthoMCL alignments → BigWig per reference |

---

## Licensing / redistribution notes

- All MalariaGEN datasets: **CC-BY 4.0**, redistributable with attribution.
- CRyPTIC: public-domain via EBI; cite the consortium.
- WHO Mtb catalogue v2: CC-BY-NC-SA 3.0 IGO; **non-commercial** — confirm BRC's commercial-use posture before re-hosting.
- TB-Profiler db: GPL-3 — OK.
- WWARN K13 tracker: data-sharing policy requires attribution; check redistribution.
- OrthoMCL / VEuPathDB: open with attribution.
- AlphaFold DB: CC-BY 4.0 (DeepMind/EMBL-EBI).

---

## Suggested first PR

Pick one of these as a proof-of-concept:

- **(A)** Tier 1.1 + 1.2 for *P. falciparum* only: ship chain files between 10 commonly-used Pf references + Pf8-derived iHS/Fst/π BigWigs projected onto each. One Galaxy workflow, three weeks of work, immediately visible value.
- **(B)** Tier 1.3 for *C. auris*: harmonized AMR marker VCF + BigBed across clade-I/II/III/IV references. Smallest scope, highest clinical visibility.

Recommend (A) — it sets up infrastructure (chains, projection pipeline, browser tracks) that every downstream tier reuses.
