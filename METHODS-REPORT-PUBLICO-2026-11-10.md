# Alchemistry 2027 Antiviral Blind Challenge — Methods Report

**Author:** Igor Eduardo
**Affiliation:** Independent Researcher
**Hugging Face username:** `igoreduardoai`
**Submission alias:** `smina-vina-baseline`
**Track:** Regression Prediction
**Method class:** Other / hybrid
**Submission date:** 9 November 2026 (America/Chicago)
**File:** `alchemistry2027_denv2.csv` (54 rows; SHA-256 `5bea9a86678c2e7388a702c93dd20d03e89cab9941e09c40b5ed2479606804cd`)

This report describes the submission already uploaded to the Alchemistry 2027 Antiviral Blind Challenge (Regression Prediction track). It is intended to serve as the public methods document required for ranking (interim deadline 15 January 2027; methods / final deadline 25 March 2027). No proprietary data were used. A public repository is not linked in this document; code is available on request.

## 1. Objective

Predict an absolute binding free energy (kcal/mol) and an uncertainty for each of 54 blinded compounds against the dengue virus serotype 2 (DENV-2) NS2B-NS3 protease.

The method is rigid-receptor docking with the smina / Vina scoring function. It is **not** a trained QSAR or GNN model, and it is **not** an alchemical free energy calculation (no FEP, TI, or perturbation network). The smina `minimizedAffinity` values are reported directly as predicted ΔG, with an explicit calibration caveat: Vina is not a free-energy estimator. Scores belong on the ML/Other leaderboard, not on the physics / FEP board. No pose archive and no edge (relative) CSV were submitted.

## 2. Data

- **Target:** Dengue-2 NS2B-NS3 protease.
- **Reference structure:** the co-crystal PDB provided by the challenge (`alchemistry-2027-challenge_reference_complex.pdb`). Two chains: NS2B (A), NS3 (B). One co-crystallized ligand (residue `LIG`, chain B, residue number 201) plus one catalytic zinc (ZN).
- **Ligands:** 54 CXSMILES from the official blinded test set (Hugging Face dataset `openadmet/alchemistry-2027-antiviral-blind-challenge`). The series is a congeneric set; experimental potencies were withheld until close and were not used.
- **Proprietary data:** none. All inputs are public. Measured potencies were not used to train, fit, or rescore this submission. There is no training set; the challenge is fully prospective.

## 3. Pipeline overview

1. Receptor preparation with PDBFixer (local, macOS ARM64): waters, catalytic zinc, and the co-crystallized ligand removed; protonation at pH 7.4.
2. Ligand preparation with RDKit 2024.03 (local, macOS ARM64): CXSMILES extensions stripped, standardize, add hydrogens, generate conformers.
3. Docking with smina on Modal cloud, CPU only (Linux x86_64).
4. Predicted ΔG = best (most negative) smina `minimizedAffinity` across returned poses and starting conformers, with no empirical correction.
5. Uncertainty = sample standard deviation of `minimizedAffinity` across returned poses for that compound (score-spread, not a calibrated thermodynamic uncertainty).
6. CSV assembly on Modal (RDKit + Python) in the official schema.

## 4. Receptor preparation

PDBFixer (OpenMM 8.x): removed waters, ZN, and the co-crystallized ligand; replaced non-standard residues; added missing atoms; protonated at pH 7.4 using the Amber19 force field. Original residue numbering was preserved. The exact OpenMM / PDBFixer patch version used in this step was not separately recorded.

## 5. Ligand preparation

The 54 CXSMILES were taken from the blinded test set. CXSMILES extensions (`|...|` suffixes) were stripped. Each string was parsed with RDKit 2024.03, the largest fragment was selected, hydrogens were added, and 5 conformers were generated with ETKDGv3 (`seed=42`) and optimized with UFF (`maxIters=2000`). All 54 compounds were processed successfully.

## 6. Docking

| Item | Value |
|---|---|
| Engine | smina (static Linux build; Vina scoring function) |
| Box | 22 Å cube centred at (−41.2, 20.0, 1.6) — heavy-atom centroid of the co-crystallized ligand in the reference PDB |
| exhaustiveness | 8 |
| num_modes | 10 |
| seed | 42 |
| cpu | 8 |
| Execution | Modal cloud, CPU only (8 CPUs, 8 GiB RAM); no GPU |
| Wall time | ~18 minutes |

The exact smina binary version or commit was not recorded in the local records; the engine is the static Linux build used with the Vina scoring function.

## 7. Predicted ΔG

The `minimizedAffinity` score from smina (Vina scoring function) is reported directly as the predicted binding free energy in kcal/mol, in the official column `denv2_ns2b_ns3_potency_predicted`. For each compound, the best (most negative) score across all returned poses and starting conformers is selected. No empirical correction, linear fit, molecular-weight term, or training on measured potencies was applied.

**Calibration caveat.** The smina / Vina scoring function is not a free-energy estimator. The scores may correlate with binding affinity but are not calibrated to experimental ΔG. This submission is an honest docking baseline on the Other / hybrid class and is intended for the ML/Other leaderboard, not the alchemical free-energy leaderboard.

## 8. Uncertainty estimation

The uncertainty for each compound (`denv2_ns2b_ns3_potency_predicted_uncertainty`) is the sample standard deviation of `minimizedAffinity` across all returned poses (up to 10 modes × 5 starting conformers = 50 poses). This reflects the spread in the scoring landscape, not a calibrated thermodynamic uncertainty or a prediction error.

- Mean uncertainty: 0.72 kcal/mol
- Range: 0.5134–0.8745 kcal/mol

## 9. Submission file and score summary

Official columns: `Molecule_Name`, `CXSMILES (CDD Compatible)`, `denv2_ns2b_ns3_potency_predicted`, `denv2_ns2b_ns3_potency_predicted_uncertainty`. Identifiers are `A_001` through the 54th test compound. All 54 values are finite. No pose zip was packaged (poses are optional and are not scored if omitted). No edge CSV was produced (this is not a relative method).

From the official CSV (SHA-256 `5bea9a86678c2e7388a702c93dd20d03e89cab9941e09c40b5ed2479606804cd`):

- 54/54 compounds scored
- Predicted ΔG range: −9.7749 to −8.2639 kcal/mol
- Mean: −9.15 kcal/mol
- Standard deviation: 0.34 kcal/mol

These figures describe the reported smina scores. They are not experimental potencies, and no correlation with the withheld measurements is claimed.

## 10. Software versions

- RDKit 2024.03
- OpenMM 8.x / PDBFixer (receptor preparation)
- smina (static Linux build, Vina scoring)
- Python 3.11

## 11. Compute and provenance

- Receptor and ligand preparation: local machine (macOS ARM64).
- Docking and CSV assembly: Modal cloud, CPU only (Linux x86_64, smina; RDKit + Python). No GPU. Wall time ~18 minutes (8 CPUs, 8 GiB RAM).
- Numerical scores were generated on 8 November 2026. The methods class was recorded as Other / hybrid on 9 November 2026; the numerical scores were unchanged. The CSV was submitted on 9 November 2026 (America/Chicago).

## 12. Reproducibility

- Ligand conformer generation used `seed=42`. Docking used `seed=42`.
- The submitted CSV is uniquely identified by SHA-256 `5bea9a86678c2e7388a702c93dd20d03e89cab9941e09c40b5ed2479606804cd` (4,418 bytes; 54 data rows plus header).
- Box centre (−41.2, 20.0, 1.6) is the heavy-atom centroid of the reference ligand and can be recomputed from the challenge PDB.
- A public reproduction repository is not linked yet.

## 13. Code availability

Preparation and docking scripts are available on request. No public repository is linked yet. A public URL for this report or for a code repository will be added only after explicit authorization to publish.

## 14. Limitations

- Rigid-receptor docking: no induced fit.
- Vina scoring function: not calibrated to experimental ΔG. Reported values are docking scores used as a ΔG proxy, not alchemical free energies.
- Uncertainties are pose-to-pose score-spread, not calibrated prediction intervals.
- Single receptor conformation from a single co-crystal.
- No alchemical free-energy calculations (FEP/TI); no relative perturbation network.
- No official correlation, MAE, or rank statistic is reported here: experimental potencies remain blinded.
- This is an honest docking baseline on the Other / hybrid class, not a physics-engine entry.
