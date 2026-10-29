# Cross-dataset phishing URL detection

Reproducibility materials for the original phishing URL detection study using PhiUSIIL as the source dataset and URL-Phish and LegitPhish as external evaluation datasets.

This package covers the original paper only. Feature-intervention, combined-method and router extension experiments are outside its scope.

**Release status:** publication package in preparation. The portable main notebook has passed syntax and basic path checks, but a complete real-data run from the assembled publication folder has not yet been verified. Existing results belong to the recorded historical run. A public data archive, repository release DOI, final manuscript citation and licensing details have not yet been added.

## Start here

To reproduce the analysis from the approved feature files and preparation records, open:

[`notebooks/URL_Phishing_Pipeline_URL_Grouped.ipynb`](notebooks/URL_Phishing_Pipeline_URL_Grouped.ipynb)

Run its cells in order after configuring the project root. **Do not run the extraction or duplication notebooks first.** They contain historical development and one-time correction steps that are not required when using the finalized input files.

The public notebook has cleared outputs and execution counts. Keep the executed research notebook and saved numerical outputs as separate evidence. Clearing notebook outputs does not delete result files.

## Package layout

Paths below are relative to the folder containing this README. Filenames are case-sensitive on Linux and Colab.

| Path | Purpose |
|---|---|
| `notebooks/URL_Phishing_Pipeline_URL_Grouped.ipynb` | Main analysis and reporting workflow |
| `notebooks/Recomputed_Feature.ipynb` | Historical feature-extraction/development notebook; optional provenance material |
| `notebooks/Duplication_Fix.ipynb` | Historical correction, overlap-audit and partition-preparation notebook |
| `data/` | Six required finalized raw/unified CSVs |
| `results/revision/url_grouped_preparation_20261020T224241295695Z/` | Approved source partitions and target selections; required input |
| `results/revision/full_20261020T231329309567Z/` | Historical reference analysis outputs, if distributed with the package |
| `results/revision/full_<UTC timestamp>/` | Newly generated full-run outputs |
| `docs/` | Dataset guide and additional reproducibility documentation |

The historical full-results directory is reference evidence, not a prerequisite for training the main analysis from the six CSVs and approved preparation records. Large result payloads may be distributed separately, provided their location and checksums are documented.

## Required datasets

Place the following six files in `data/`. Use the approved versions without editing their contents, row order, spelling or column order.

| Dataset | Raw CSV | Unified feature CSV | Raw label meanings |
|---|---|---|---|
| PhiUSIIL | `phiusiil.csv` | `phiusiil_unified.csv` | `0` legitimate; `1` phishing |
| URL-Phish | `URL-Phish.csv` | `urlphish_unified.csv` | `0` legitimate; `1` phishing |
| LegitPhish | `LegitPhish Dataset.csv` | `legitphish_unified.csv` | `1` legitimate; `0` phishing |

**Every unified CSV uses `0 = legitimate`, `1 = phishing`.** The main notebook does not invert unified labels or regenerate features. Raw CSVs support identity, label and row-alignment checks.

Expected full-file counts:

| Dataset | Total rows | Legitimate | Phishing |
|---|---:|---:|---:|
| PhiUSIIL | 235,795 | 134,850 | 100,945 |
| URL-Phish | 116,600 | 100,000 | 16,600 |
| Corrected LegitPhish | 101,218 | 37,540 | 63,678 |

One LegitPhish row with a missing native label was removed from the raw and unified files together. The approved full-file counts above already reflect that correction. Do not remove another row or apply the historical correction again.

Do not substitute backup CSVs, `legitphish_shared_aligned.csv`, earlier development-disjoint CSVs or `phiusiil - Original.csv`. Those are not inputs to this finalized analysis.

### Data availability and permissions

The data distribution location and source dataset citations must be supplied before this repository is released. Until those links are added, this README is not a complete public data-download route.

Third-party datasets retain their original terms. Do not assume that the repository's code license permits redistribution of their raw or derived data. If a dataset cannot be redistributed, provide its official source, the exact source version/checksum and a verified preparation route instead.

### Unified feature schema

The unified CSV header must contain these 15 feature columns in the following order, followed by `label`:

```text
URLLength, DomainLength, IsDomainIP, TLD, TLDLength,
NoOfSubDomain, NoOfLettersInURL, LetterRatioInURL,
NoOfDegitsInURL, DegitRatioInURL, NoOfEqualsInURL,
NoOfQMarkInURL, NoOfAmpersandInURL,
NoOfOtherSpecialCharsInURL, IsHTTPS, label
```

`TLD` is categorical; the other 14 features are numeric. The spellings `Degits` and `DegitRatioInURL` are intentional compatibility names. URL identifiers, row identifiers and labels are not model features.

Matching labels and URL lengths support row alignment but do not independently prove every feature's extraction provenance. Exact upstream definitions and fallback behaviour require their own provenance documentation.

## Required preparation records

The following six files must remain together inside:

`results/revision/url_grouped_preparation_20261020T224241295695Z/`

- `source_partitions.npz`
- `source_partition_record.json`
- `urlphish_selection.npz`
- `urlphish_selection_record.json`
- `legitphish_selection.npz`
- `legitphish_selection_record.json`

The main notebook checks the preparation files against embedded SHA256 values, then verifies the referenced dataset contents. Keep the preparation JSONs byte-for-byte unchanged, even if they contain the original absolute Drive paths. The portable notebook maps approved dataset filenames to the current root and still checks their content hashes.

Do not edit recorded paths inside those JSONs, bypass a hash mismatch or regenerate partitions solely to make a relocated copy run. A mismatch means the input versions need investigation.

### Saved source partitions

Repeated exact URLs remain in the same source partition. URL comparison strips surrounding whitespace only; it does not canonicalize URLs generally.

| Partition | Rows | Legitimate | Phishing |
|---|---:|---:|---:|
| Source training | 150,931 | 86,294 | 64,637 |
| Source validation | 37,716 | 21,569 | 16,147 |
| Source test | 47,148 | 26,987 | 20,161 |

No exact URL is shared between those three partitions. Load the saved indices rather than redrawing partitions under a different library version.

### Selected external cohorts

Target URLs matching source training **or validation** URLs are excluded using the saved selections.

| Target | Selected rows | Legitimate | Phishing | Distinct URLs |
|---|---:|---:|---:|---:|
| URL-Phish | 116,126 | 99,526 | 16,600 | 114,764 |
| LegitPhish | 71,767 | 8,089 | 63,678 | 71,421 |

These cohorts still contain source-test URL matches: 130 URL-Phish rows and 7,484 LegitPhish rows. They are disjoint from source development URLs, not independent of the entire source dataset. Exact URL grouping also does not establish domain independence.

MMD and entropy use a separately restricted source-test reference excluding URLs in either selected target and retaining one representative per URL. Do not confuse that reference population with the main source-test classifier population.

## Environment

The main analysis requires Python and these packages:

`numpy`, `pandas`, `scipy`, `scikit-learn`, `lightgbm`, `matplotlib`, `joblib`, `threadpoolctl`.

Local notebook execution also needs Jupyter, JupyterLab or another compatible notebook interface. The historical extraction notebook additionally uses packages such as `tldextract` and, in its separate neural experiment, PyTorch; those experiments are not prerequisites for running the main analysis from approved unified CSVs.

Recorded working Colab environment:

| Component | Recorded version |
|---|---|
| Python | 3.13.16 |
| NumPy | 2.1.3 |
| pandas | 2.2.3 |
| SciPy | 1.16.3 |
| scikit-learn | 1.6.1 |
| LightGBM | 4.6.0 |
| Matplotlib | 3.10.0 |
| joblib | 1.6.0 |
| threadpoolctl | 3.7.0 |

Use the archived run's environment record as the authority for the results being reproduced. The notebook also contains reference package pins that can differ from the actual loaded versions. Its configured full run records versions without automatically installing them; that is not proof of numerical equivalence across environments.

**Do not upgrade or downgrade NumPy inside a running Colab kernel.** A preloaded library is not itself an error. If environment repair is necessary, perform installation separately, restart the runtime and then run the analysis from the beginning. Report installed-versus-loaded version conflicts rather than repeatedly reloading modules.

Full analysis includes repeated calibration, bootstrap intervals, multiple ablations and kernel calculations. Runtime and RAM needs depend on the host. No measured minimum RAM or portable wall-time guarantee is supplied here. The main tabular analysis does not require the optional Char-CNN experiment.

## Running on a local computer

1. Obtain the code, the six approved CSVs and the six preparation files.
2. Arrange them using the relative paths above.
3. Open `notebooks/URL_Phishing_Pipeline_URL_Grouped.ipynb`.
4. In its first code cell, set the project root. For example:

   ```python
   PROJECT_ROOT = r"D:\Phishing_Paper_Reproducibility"
   ```

   On macOS/Linux, use the corresponding absolute folder path. The root is the folder containing `data`, `notebooks` and `results`; it is not the `notebooks` folder itself.

5. Leave `RUN_SYNTHETIC_SELF_TEST = False` for the real analysis.
6. Run the cells in order from a clean kernel.
7. Check the final status and generated run manifest before using any output.

With an empty `PROJECT_ROOT`, the configuration tries the current working directory and its parent. If it cannot identify a unique project root, set the absolute path explicitly. Opening a notebook does not always set its working directory to its own folder.

Local execution does not require mounting Google Drive or importing Colab-specific modules when a local root is selected.

## Running in Google Colab

1. Place the same project structure in your own Google Drive.
2. Open the main notebook in Colab.
3. Set the first cell's root to your own project folder, for example:

   ```python
   PROJECT_ROOT = "/content/drive/MyDrive/Phishing_Paper_Reproducibility"
   ```

4. Execute that cell and authorize Drive mounting when prompted. An already mounted Drive is reused.
5. Confirm that all required CSVs and preparation files exist under that root.
6. Keep synthetic testing disabled and run the remaining cells in order.

Changing the root does not download or copy datasets automatically. Authentication failures must be resolved through the Colab account/session; changing scientific code will not repair Drive authorization.

## What the main notebook runs

| Stage | Main purpose |
|---|---|
| 1 | Validate full feature inputs, raw alignment, saved selections and URL identities |
| 2 | Load source partitions, fit source-only preprocessing and train LR/LightGBM models |
| 3 | Baseline metrics, uncertainty, probability saturation and operating points |
| 4 | Disjoint target adaptation/evaluation calibration comparisons |
| 5 | Repeated distinct-URL label-budget experiments |
| 6 | Distribution-shift diagnostics, single-feature AUROC and pruning |
| 7 | Matched-seed feature-family ablation and paired uncertainty |
| 8 | Numeric-feature MMD with sensitivity and class-conditional analyses |
| 9 | Source and error-focused feature attributions |
| 10 | Source-calibrated entropy, monitoring and selective-prediction analysis |
| 11 | Figures generated from saved numerical outputs |
| 12 | Final validation and completion records |

LightGBM and logistic regression are evaluated on matched populations. Target-label-selected thresholds and other oracle diagnostics must remain clearly distinguished from deployable/source-selected policies.

The reviewed LightGBM configuration has `subsample_freq=0`; row bagging is inactive even though `subsample=0.8`. Do not silently enable it when reproducing the paper's run.

## Outputs and verification

A full run creates a new timestamped directory under `results/revision/`. It saves numerical tables, figures, model objects, predictions, split/group records and diagnostics in the corresponding subfolders, together with:

- `run_manifest.json`: configuration, input records, environment and completed stages.
- `artifact_inventory.csv`: recorded output members and integrity information.
- Run-local protocol and additional analysis records generated by the notebook.

Use the generated tables and figures from the **same completed run**. Do not mix older values with a later run, manually replace an unfavourable number or treat cached notebook displays as analysis inputs.

Before reporting results, confirm:

- All 12 stages completed successfully.
- Full-file and selected-cohort counts match this README.
- Preparation and input integrity checks passed.
- Confusion counts agree with the reported metrics.
- Oracle and non-oracle results remain identified.
- The saved environment and configuration correspond to the intended analysis.
- The manuscript's tables and figures map to that exact run.

An interrupted execution is incomplete. The notebook does not provide a documented general-purpose resume-from-checkpoint procedure. Do not combine partial outputs with a later run. Preserve interrupted outputs separately and start a new run when the runtime state cannot be recovered reliably.

## Optional synthetic execution check

The main notebook exposes `RUN_SYNTHETIC_SELF_TEST`. Setting it to `True` selects generated test inputs and a reduced workload. This checks execution paths only; synthetic scores are not paper results or evidence about the real datasets.

The shared root-configuration cell runs first, so provide an existing project root even when selecting the synthetic mode later. The synthetic paths in the historical implementation use `/tmp`; Windows support for that mode has not been fully validated. Use the real-data configuration instructions above for the publication analysis.

## Historical notebooks: read before executing

### `Recomputed_Feature.ipynb`

This notebook contains extraction development, repeated extraction cells, an exploratory MMD calculation and a separate Char-CNN experiment. Extraction cells write unified CSVs to `data/` and can overwrite existing files. Use a disposable data copy for any extraction comparison. This notebook has not been consolidated into a verified top-to-bottom regeneration recipe.

### `Duplication_Fix.ipynb`

This notebook records one-time missing-label correction, historical overlap audits and partition/selection preparation. Some cells expect superseded results or earlier data versions deliberately excluded from this package. It is not a prerequisite for running the finalized main notebook with supplied preparation files. Do not execute its correction cells against already corrected data or recreate the approved partitions to bypass integrity checks.

These notebooks are retained for traceability. Their presence does not establish that every historical cell produced the final paper artifacts. A clean, independently tested feature/preparation regeneration workflow remains a separate release task.

## Interpretation limits

- Exact URL grouping prevents identical-URL leakage between designated partitions; it does not establish independence among URLs from the same domain.
- Main target cohorts retain the documented source-test overlap.
- MMD is computed on the 14 numeric base features, not the full one-hot TLD space.
- The restricted MMD/entropy reference is not the entire source deployment population.
- Feature-family ablations use explicitly defined masks; some diagnostic families intentionally overlap and are not a partition or an additive causal decomposition.
- AP from margins and AP from probabilities can differ when probabilities saturate or introduce ties. Preserve the reported ranking representation.
- Target calibration and evaluation roles are separated within experiments, but these historical datasets have already informed the research. They are not newly collected confirmation data.
- Bootstrap intervals, seed variability and split-to-split variability answer different questions. Do not describe all of them as confidence intervals for the same quantity.
- Successful execution and matching checksums do not prove that the entire scientific study is free of errors.

## Citation and license

The final paper citation, author metadata, code license, original dataset citations and archived release DOI must be added before publication. No license grant is implied by this draft README. Do not invent a DOI or describe a planned archive as already available.

Cite the specific archived code/data versions used by the manuscript once their release records exist.

## Publication checklist

- [x] Portable publication copies of the three notebooks prepared with cleared outputs.
- [x] Basic syntax and root-path checks performed on the main publication copy.
- [x] Required six datasets and six approved preparation files identified.
- [ ] Verify the assembled package contains the exact approved input bytes.
- [ ] Review the dataset guide against the final selected files and label mappings.
- [ ] Verify upstream extraction provenance and document regeneration limits.
- [ ] Confirm redistribution terms and add official dataset citations/download links.
- [ ] Add a dependency/environment file matched to the archived real run.
- [ ] Run the main publication notebook from a fresh location with real data.
- [ ] Reconcile regenerated results with the manuscript and explain material differences.
- [ ] Verify the intended release contents exclude secondary research and private material.
- [ ] Add the code license and final paper/author citation metadata.
- [ ] Publish a versioned code release and permitted data archive.
- [ ] Add the real repository/data links and release DOI to this README and paper.

Update this checklist only when the corresponding action has evidence. Publishing this README does not itself mark the package as fully reproduced.
