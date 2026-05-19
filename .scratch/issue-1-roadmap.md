# Gap-driven roadmap for pre-analyzed pathogen resources (companion to the pangenome initiative)

## Context

We're building per-species pangenome resources (P. falciparum, P. vivax, C. auris, etc.) that let users project features across multiple references within a species. This issue scopes additional "pre-analyzed" resources that BRC could host alongside the pangenomes.

The selection principle is **gap-filling, not duplication**. BRC-A's unique leverage is _per-reference projection + cross-isolate aggregation + Galaxy-workflow launch_ across a catalog of ~thousands of pathogen assemblies. Most existing resources (PlasmoDB, FungiDB, TB-Profiler, MalariaGEN, AlphaFold DB) are anchored to a single canonical reference per species. The candidates below specifically target what those resources don't deliver.

---

## Tier 1 — Direct extensions of the pangenome work

### 1. Liftover chain files between every reference within a species
- **Gap:** No portal hosts UCSC chain files between e.g. Pf 3D7 ↔ Dd2 ↔ HB3 ↔ IT; Af293 ↔ A1163; *C. auris* clade-I/II/III/IV; H37Rv ↔ CDC1551 ↔ Erdman.
- **Why it matters:** Without chains, the "project feature X onto strain Y" promise is brittle for users who don't want to engage with graph pangenomes.
- **Inputs / tooling:** [nf-LO](https://github.com/evotools/nf-LO), [levioSAM2](https://github.com/milkschen/leviosam2), UCSC `liftOver`.
- **Deliverable:** Pairwise chain files + a manifest, downloadable per species.

### 2. Population-genomic signal tracks (BigWig/BigBed) per reference
- **Gap:** Pf8 (33,325 samples), Pv4, Ag1000G, CRyPTIC publish VCFs but **not** iHS, XP-EHH, Fst (by region/year), Tajima's D, π, or country-level allele frequency as ready-to-use browser tracks — and certainly not projected onto every reference in the catalog.
- **Deliverable:** BigWig/BigBed per signal × per reference, generated once via a Galaxy workflow.
- **Inputs:** [MalariaGEN Pf8](https://www.malariagen.net/article/pf8-released-a-major-update-for-the-worlds-largest-malaria-parasite-genomic-data-resource/), Ag1000G, CRyPTIC.

### 3. Harmonized drug-resistance marker catalog for eukaryotic pathogens
- **Gap:** The [WHO Mtb mutation catalogue v2](https://iris.who.int/handle/10665/374061) sets the gold standard. **No equivalent exists for malaria, fungi, or kinetoplastids.**
- **Scope:** ~20–80 high-confidence loci per species — e.g. *kelch13*, *crt*, *mdr1*, *dhfr*, *dhps* (Pf); *cyp51A*, HapE, Cox10 (*A. fumigatus*); *ERG11*, *FKS1*, *TAC1B* (*C. auris*); *ERG11*, *FKS* (*Cryptococcus*); arsenal genes (*Leishmania*).
- **Deliverable:** VCF + BigBed annotation track per reference, with confidence grading following the WHO Mtb model.
- **Recent input:** [*A. fumigatus* pangenome + accessory genes tied to itraconazole resistance (biorxiv 2026)](https://www.biorxiv.org/content/10.64898/2026.04.14.718449v1).

---

## Tier 2 — Functional layers that exist as supp-tables but not as tracks

### 4. Essentiality / CRISPR-fitness tracks
Convert per-gene fitness scores from primary papers into harmonized BigWig/TSV per reference.
- [Cryptosporidium in-vivo CRISPR screens, Nat Commun 2025](https://www.nature.com/articles/s41467-025-63012-1)
- [Cryptococcus essential + fluconazole-resistance atlas (2025)](https://pubmed.ncbi.nlm.nih.gov/40402997/)
- Existing Toxoplasma genome-wide CRISPR data, Plasmodium piggyBac data.

### 5. Mappability / "dark genome" / GC tracks per assembly
Critical for AT-rich genomes (Pf ~80% AT; new methods like [CUT&Tag for Pf](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12461585/) exist *because* of this). Cheap to compute once with GenMap.

### 6. Subtelomere / TE / repeat annotation tracks
Most virulence/AMR variation in Pf (*var*/*rifin*/*stevor*), *Candida*, *Aspergillus*, *Cryptococcus*, and *Trypanosoma* (VSG) lives in subtelomeres, but standard GFFs don't flag these regions. Template: [Z. tritici pangenome-guided TE library](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10652580/).

### 7. SV / CNV catalogs per reference
Pf8 *gch1*/*crt* CNVs, *C. auris* clade-specific accessory genes/CNVs, *A. fumigatus* aneuploidy-driven azole resistance — none centralized as per-reference BED tracks.

---

## Tier 3 — Cross-omic layers

### 8. Strain-specific predicted structures
AlphaFold DB only covers reference proteomes. AF3/ESMFold makes per-strain proteoform structures feasible — high value where SNPs/indels alter binding pockets (Kelch13, Erg11).

### 9. MHC-I/II epitope tracks for vaccine prioritization
NetMHCpan over common HLAs, projected per reference. Exists piecemeal for SARS-CoV-2/flu/TB; nothing unified for eukaryotic parasites.

### 10. Antigen-diversity hotspot tracks
Per-locus π and dN/dS for *var*/*rifin*/*stevor* (Pf), VSG (*T. brucei*), SAG/SRS (*Toxo*), fungal surface families.

### 11. Organellar variant catalogs
Mt / apicoplast variation is routinely dropped — see [*A. fumigatus* mitogenome × azole, npj AMR 2025](https://www.nature.com/articles/s44259-025-00083-6).

### 12. Cross-species ortholog projection track
Derive a per-reference BED from [OrthoMCL 7 / VEuPathDB](https://orthomcl.org/): "single-copy ortholog across Apicomplexa," "pathogen-specific expansion," etc.

### 13. Host-pathogen PPI tracks
Project the pathogen side of large-scale AF-Multimer interactomes ([Saluri 2024; Li 2025](https://www.biorxiv.org/content/10.1101/2025.06.04.657796v1.full)) onto pathogen genome coordinates.

---

## Tier 4 — Outbreak / surveillance gaps

### 14. Reference-anchored phylogenies for non-viral priority pathogens
UShER/Nextstrain do this for SARS-CoV-2 / flu / mpox. **No equivalent downloadable, regularly-updated tree + tip metadata** for Pf, Mtb, *C. auris*, *Cryptococcus*, *Cryptosporidium*. Ready inputs:
- [12,644-genome *C. auris* analysis (biorxiv 2026)](https://www.biorxiv.org/content/10.64898/2026.02.03.703534v1.full.pdf)
- [Global *C. neoformans* surveillance (biorxiv 2025)](https://www.biorxiv.org/content/10.1101/2025.11.11.687764v1.full)

### 15. *Cryptosporidium* gp60 subtype reference panel
The [2025 gp60 review](https://pmc.ncbi.nlm.nih.gov/articles/PMC12304688/) documents IeA11G3T3 going from rare to dominant in 2 years. Wrap [CryptoGenotyper](https://pubmed.ncbi.nlm.nih.gov/33846277/) as a Galaxy workflow + ship reference subtype panel per *C. hominis*/*C. parvum* reference.

### 16. Vector-pathogen-host overlays
BRC-A is one of the few resources covering both vectors and pathogens — overlay Ag1000G insecticide-resistance allele frequencies on the *An. gambiae* reference *alongside* transmission-relevant loci on companion Pf references.

---

## Tier 5 — Cheap utility tracks

- **17.** Standardized assembly-QC / contamination / ploidy reports per reference (extend the existing v0.10 ploidy QC).
- **18.** Codon-usage tables + heterologous-expression compatibility scores per reference.
- **19.** Cross-clade conservation tracks (phyloP/phastCons-style) — no general parasite/fungal equivalent of the vertebrate phyloP tracks exists.

---

## Recommended quick wins

If we want a first deliverable that visibly demonstrates the gap-filling thesis, I'd start with:

1. **Liftover chains + Pf8/Pv4/Ag1000G signal BigWigs** projected onto every reference — purely derived from existing public data, nobody hosts it.
2. **Curated AMR catalog for eukaryotic pathogens** modeled on WHO Mtb v2 — Pf, *C. auris*, *A. fumigatus*, *Cryptococcus*, *Leishmania*.
3. **Reference-anchored, regularly-updated phylogenies + metadata** for Pf, Mtb, *C. auris*, *Cryptococcus*.

---

## Checklist (proposing as sub-issues)

- [ ] Tier 1.1 — Pairwise liftover chains per species
- [ ] Tier 1.2 — Population-genomic signal tracks per reference
- [ ] Tier 1.3 — Eukaryotic-pathogen AMR catalog (WHO-style)
- [ ] Tier 2.4 — Essentiality / CRISPR fitness tracks
- [ ] Tier 2.5 — Mappability / GC tracks
- [ ] Tier 2.6 — Subtelomere / TE annotation tracks
- [ ] Tier 2.7 — SV / CNV catalogs per reference
- [ ] Tier 3.8 — Per-strain predicted structures
- [ ] Tier 3.9 — MHC epitope tracks
- [ ] Tier 3.10 — Antigen-diversity hotspot tracks
- [ ] Tier 3.11 — Organellar variant catalogs
- [ ] Tier 3.12 — Cross-species ortholog projection track
- [ ] Tier 3.13 — Host-pathogen PPI tracks
- [ ] Tier 4.14 — Reference-anchored phylogenies for non-viral priority pathogens
- [ ] Tier 4.15 — *Cryptosporidium* gp60 subtype reference panel
- [ ] Tier 4.16 — Vector-pathogen-host overlays
- [ ] Tier 5.17–19 — QC, codon usage, conservation tracks
