# Phage-RBP-Structure-Retrieval-Curation

This repository contains a pipeline for constructing a curated benchmarking dataset of **phage receptor-binding proteins (RBPs)** and their experimentally solved **3D structures**.  
The goal is to support downstream machine learning models that predict interactions between bacteriophage RBPs and bacterial surface proteins.

The full workflow is implemented in **`main.ipynb`**, which performs two major tasks:

---

## 1. High-confidence identification of phage RBPs

The pipeline begins with the input table **`pharbp_export.csv`**, which contains phage receptor-binding protein candidates.

I define a **high-confidence RBP** based on two criteria:

1. **Evidence Type Count ≥ 3**  
   (DepoScope, RBPdetect, RBPdetect2, UniProt/TrEMBL, etc.)

2. **All evidence probabilities ≥ 0.95**  

Entries satisfying both conditions are retained and form the **filtered RBP set**.

---

## 2. Retrieval and curation of structural templates

For each high-confidence RBP:

### 2.1 Extract the protein sequence
The corresponding sequence is retrieved from **`pharbp_sequences.fasta`**.

### 2.2 Identify structural homologs
Two independent search tools are used:

- **RCSB PDB Sequence Search API**
- **MMseqs2** (local search against PDB70)

These tools return candidate PDB structures that align to the query RBP.

### 2.3 Local sequence alignment and filtering
Each candidate PDB chain is locally aligned to the RBP sequence.  
We compute:

- **Sequence identity**
- **Sequence coverage**

We retain only structures satisfying:

- **Identity ≥ 40%**  
- **Coverage ≥ 70%**

### 2.4 Classification of structures into two categories

Each retained PDB entry is classified based on its CIF metadata:

- The **source organism** of each polymer chain is retrieved.
- Each organism is mapped to its **NCBI superkingdom** (virus or bacteria).
- Classification:

| Category | Meaning |
|----------|---------|
| **phage_only** | Structure contains *only phage-derived protein chains* |
| **phage_bacteria** | Structure contains at least one *phage* chain and one *bacterial* chain |


---

## 3. Final output

A final summary file **`final_templates.csv`** is generated, detailing for each curated RBP:

- CDS ID  
- Matched PDB ID  
- Sequence identity  
- Sequence coverage  
- Structural classification (`phage_only` or `phage_bacteria`)
