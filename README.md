# protein-metrics
What do proteins have to tell us about themselves?

<a href="https://colab.research.google.com/github/Sofokli5/protein-metrics/blob/main/Enzyme_Metrics.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

# Enzyme Metrics Pipeline: Structural & Thermodynamic Analysis of Protein Complexes

This repository contains a Google Colab notebook (`Enzyme_Metrics.ipynb`) that analyses **AlphaFold 3** models of the CYP17–POR enzyme–reductase complex and measures how a drug changes it.

For every drug, the notebook compares two models:

- **Holo:** CYP17 + POR + Heme + drug
- **Control:** CYP17 + POR + Heme, without a drug (the same Control is used for every drug)

Each run adds one row of metrics for the current drug to a shared CSV file. The last cell reads that file and ranks all drugs analysed so far.

## Input Files

All files are read from one Google Drive folder (`BASE_DIR`, set in cell 2). They are the standard AlphaFold 3 outputs:

| Model | Files |
|---|---|
| Holo | `Holo_CYP17_POR_<LIGAND>_model.cif`, `_confidences.json`, `_summary_confidences.json` |
| Control | `Control_CYP17_POR_Base_model.cif`, `_confidences.json`, `_summary_confidences.json` |

The pipeline expects **chain A = CYP17**, **chain B = POR** and the Heme as residue `HEM`. The drug is identified as any non-protein molecule that is not the Heme.

## How to Run

1. Open the notebook in Colab. Google Drive is mounted from the Colab interface.
2. In cell 2, set `LIGAND` to the drug name used in the file names (e.g. `"Rifampicin"`).
3. Run all cells in order. Cells 1 and 2 must run first in every new session.

Cell 1 installs all dependencies and stops with an error if any of them fails. AutoDock Vina is installed as the official pinned command-line binary (v1.2.7, checksum-verified), so it does not depend on Colab's Python version.

## Pipeline Workflow & Metrics Calculated

| Cell | Step | Tools | Main metrics |
|---|---|---|---|
| 1 | Installation | pip, apt | — |
| 2 | Input files: defines and checks all input files, converts CIF to PDB for the session | OpenBabel | — |
| 3 | Interactive 3D view of the Holo model (optional) | py3Dmol | — |
| 4 | **Model confidence**, Control vs Holo | NumPy, Biopython | pTM, ipTM, ipTM CYP–POR, ranking score, clashes, global and interface PAE, pLDDT per chain and for the drug |
| 5 | **Surface areas**, Control vs Holo | FreeSASA | protein-only Buried Surface Area (BSA) of the CYP–POR interface, **ΔBSA**, buried area and % of the drug |
| 6 | **POR deformation**, Holo vs Control | MDAnalysis | CYP backbone RMSD after superposition, **POR displacement** (POR position relative to CYP), **POR conformational change** (POR shape) |
| 7 | **Where the drug sits** | MDAnalysis | residues within 4 Å of the drug, per chain (`A:PHE 114`), contacts with CYP and POR, minimum distance to the Heme |
| 8 | **Drug binding** | AutoDock Vina, OpenBabel | Vina score of the AlphaFold pose with CYP+POR+Heme and with CYP+Heme only, **POR contribution**, score after local optimization, ligand efficiency |
| 9 | **CYP–POR binding affinity**, Control vs Holo | PRODIGY | predicted ΔG and Kd at 37 °C, interface contacts by type, **ΔΔG** |
| 10 | **Summary**: one row per drug in the shared CSV (`all_drugs_metrics.csv`), with run date and tool versions | pandas | — |
| 11 | **Drug comparison and ranking** | pandas | see below |

For Vina, hydrogens are added at pH 7.4 and the Heme is part of the receptor. The search box is fitted around the drug (`--autobox`).

## Comparing Drugs

Cell 11 reads the shared CSV and works in four steps:

1. **Consistency check:** warns if the Control values differ between rows, i.e. if the drugs were not analysed against the same Control.
2. **Reliability filter:** drugs with ipTM CYP–POR < 0.6, drug pLDDT < 50 or a predicted clash are listed separately and not ranked.
3. **Location:** each drug is classified as *Active site* (touches the Heme), *Interface* (contacts with both CYP and POR), *CYP surface* or *POR surface*.
4. **Ranking** (1 = best):
   - `Effect_Rank`: mean rank over PRODIGY ΔΔG, ΔBSA and Δ ipTM CYP–POR, i.e. the effect on CYP–POR binding.
   - `Binding_Rank`: mean rank over the locally optimized Vina score and the ligand efficiency.

Set `GOAL = "stabilize"` to rank first the drugs that strengthen CYP–POR binding, or `GOAL = "disrupt"` for the opposite. The POR RMSD values are shown for context but are not ranked, because they measure how much POR changes, not in which direction.

## Interpreting the Results

- **Use the values comparatively.** PRODIGY ΔG and Kd, and Vina scores, are best used for Holo vs Control and drug-vs-drug comparisons, not as absolute affinities. Vina scores typically carry errors of about ±2 kcal/mol.
- **AlphaFold confidence is not affinity.** pTM, ipTM, PAE and pLDDT describe how sure the model is about a structure, not how strongly the proteins bind.
- **PRODIGY and BSA see only the proteins.** The drug's effect on CYP–POR binding appears only through how it changes the predicted interface.
- **One model per system.** Each value comes from a single AlphaFold 3 model. Small differences between drugs may be within the variation between AlphaFold seeds. Analysing several seeds of the Control shows how large a difference has to be to be meaningful.
- **Use the same AlphaFold 3 settings** (seeds, number of samples, molecular composition) for every drug, so that their rows stay comparable.

## Purpose & Application

This pipeline is an *in silico* structural screening tool. It combines AlphaFold 3 predictions with physicochemical metrics from several tools, so that researchers can compare and rank drugs by their effect on the assembly of the enzyme–reductase complex before wet-lab validation.

## License

MIT, see [LICENSE](LICENSE).
