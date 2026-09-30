# PH-ECG

**Partially Separable Dual-Branch ECG Representations of Pulmonary Pressure Load and RV Injury: A Causal-Inspired Pressure–RV Deviation Analysis**

This repository currently provides data sources, access requirements, and preparation notes. Training and analysis code will be added in a subsequent update.

## Data sources and access

| Dataset | Study use | Source |
| --- | --- | --- |
| EchoNext, version 1.1.1 | Development, validation, and independent testing using ECG waveforms and echocardiography-derived labels | [PhysioNet EchoNext](https://physionet.org/content/echonext/1.1.1/) |
| MIMIC-IV-ECG, version 1.0 | External-validation ECG waveforms | [PhysioNet MIMIC-IV-ECG 1.0](https://physionet.org/content/mimic-iv-ecg/1.0/) |
| MIMIC-IV-Echo, version 1.0.1 | External-validation structured echocardiographic measurements | [PhysioNet MIMIC-IV-Echo 1.0.1](https://physionet.org/content/mimic-iv-echo/1.0.1/) |

EchoNext is a separate Columbia/Allen dataset, not a MIMIC-IV module. The study uses EchoNext version 1.1.1.

EchoNext requires registration and acceptance of its applicable data-use agreement. MIMIC datasets require PhysioNet credentialing, the specified research training, and acceptance of the relevant dataset-specific agreements. Follow the requirements on each source page.

## Data licenses

Dataset use is governed by the original PhysioNet licenses and agreements:

- [EchoNext access, license, and agreement](https://physionet.org/content/echonext/1.1.1/)
- [MIMIC-IV-ECG 1.0 access, license, and agreement](https://physionet.org/content/mimic-iv-ecg/1.0/)
- [MIMIC-IV-Echo 1.0.1 access, license, and agreement](https://physionet.org/content/mimic-iv-echo/1.0.1/)

This repository does not grant access to or relicense those datasets. Obtain authorized copies directly from PhysioNet. Raw ECG waveforms, patient-level metadata, patient-level labels, and patient-level predictions are not hosted here.

## EchoNext preparation

The development resource contains 100,000 12-lead ECG records with paired echocardiography-derived labels. Each ECG is 10 seconds at 250 Hz, giving 2,500 samples per lead.

The study retained the supplied patient-level training, validation, and test splits. The training cohort used only the latest ECG per patient; records marked `no_split` were excluded. The resulting cohorts comprised 26,218 training, 4,626 validation, and 5,442 test patients.

The local loader uses:

- `echonext/echonext_metadata_100k.csv`
- `echonext/EchoNext_train_waveforms.npy`
- `echonext/EchoNext_val_waveforms.npy`
- `echonext/EchoNext_test_waveforms.npy`

The metadata filename may require a local rename to match the loader. Preserve original metadata row indices when filtering so that labels remain aligned with the corresponding split-specific waveform arrays. Inputs are converted to float32 and standardized separately within each lead.

TRV is the continuous pressure-load training target; PASP is used for clinical-scale interpretation and auxiliary tasks. RV systolic function is represented by four grades, and LVEF is a post-hoc control phenotype.

## MIMIC-IV external preparation

The study uses MIMIC-IV-ECG waveforms and MIMIC-IV-Echo structured measurements. Echocardiographic DICOM images are not required.

ECG and echocardiographic studies are paired within each patient using deidentified timestamps and an absolute interval of at most 24 hours. The pair with the smallest interval is retained; ties are resolved using the smaller ECG study identifier. The external cohort contains 31,292 patients.

ECGs are resampled from 500 to 250 Hz with anti-aliasing and reordered to the development lead order: I, II, III, aVR, aVL, aVF, V1–V6. Inputs undergo the same within-lead standardization.

Nonpositive TRV values are treated as missing. External reference PASP is derived from recorded TRV as `4 × TRV² + 10` mmHg. Model-predicted TRV is mapped to PASP with the development mapping `4.07 × TRV² + 8.1` mmHg. RV function is mapped to grades 0–3; unassessable or ungraded descriptions are treated as missing. Grade ≥2 defines moderate-or-worse dysfunction.

## Dataset citations

Use the version-specific citation and required original publications shown on each PhysioNet page.

- MIMIC-IV-ECG 1.0: DOI [10.13026/4nqg-sb35](https://doi.org/10.13026/4nqg-sb35).
- MIMIC-IV-Echo 1.0.1: DOI [10.13026/307c-mr50](https://doi.org/10.13026/307c-mr50).
- EchoNext 1.1.1: P. Elias and J. Finer, PhysioNet, 2026. DOI [10.13026/7cfw-d091](https://doi.org/10.13026/7cfw-d091).

