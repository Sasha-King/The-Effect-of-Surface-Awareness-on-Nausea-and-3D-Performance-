# fNIRS Processing Documentation

The numbers needed are participant-level cognitive-load values for:

- `CL_E_DLPFC_ch1_7_hbdiff_uM`: frontal/DLPFC ROI, channels 1-7
- `CL_V_occipital_ch8_15_hbdiff_uM`: posterior/occipital ROI, channels 8-15

These values are computed as:

```text
HbDiff(t) = HbO(t) - HbR(t)

trial change = mean(HbDiff during final 5 seconds of gameplay)
             - mean(HbDiff during final 5 seconds of preceding baseline)

participant ROI value = average trial change across valid baseline-game pairs
```

Sign convention:

```text
positive value = gameplay > baseline
negative value = gameplay < baseline
```
Units are `uM` after modified Beer-Lambert conversion.

## Software
Installed analysis versions:

- MNE-Python: `1.10.2`
- MNE-NIRS: `0.7.1`

## Final Preprocessing Pipeline

The primary pipeline follows the MNE fNIRS workflow:

1. Load NIRx participant folders with MNE.
2. Keep valid continuous-wave fNIRS channels and remove zero/invalid source-detector distances.
3. Convert raw light intensity to optical density.
4. Compute scalp coupling index (SCI) on optical-density data.
5. Mark channels with `SCI < 0.5` as low-quality.
6. Remove low-quality channels at the participant level only.
7. Retain participants if each ROI has at least 3 usable source-detector pairs.
8. Correct optical-density data for motion using temporal derivative distribution repair (TDDR).
9. Convert optical density to HbO and HbR using the modified Beer-Lambert law.
10. Use age-specific differential pathlength factors for 760 nm and 850 nm.
11. Band-pass filter HbO/HbR from `0.01` to `0.09 Hz`.
12. Create baseline-game trial pairs from trigger annotations.
13. Compute last-5s gameplay minus last-5s baseline contrasts.
14. Average remaining valid channels within each ROI.
15. Average across valid trials to produce one frontal value and one occipital value per participant.


## ROI Definitions

ROI definitions were based on the standardized optode montage.

| ROI | Interpretation | Source-detector region | Cognitive-load variable |
|---|---|---|---|
| Frontal | DLPFC/executive load | channels 1-7 | `CL_E_DLPFC_ch1_7_hbdiff_uM` |
| Posterior | Occipital/visual load | channels 8-15 | `CL_V_occipital_ch8_15_hbdiff_uM` |

Operationally, frontal channels correspond to source-detector pairs within sources/detectors 1-4. Posterior channels correspond to pairs within sources/detectors 5-8.

## Trigger Mapping and Trial Definition

The trigger files use rows of the form:

```text
timestamp;sample;code
```

Final mapping:

| Trigger code | Meaning |
|---:|---|
| `2` | Gameplay/activity period |
| `3` | Visual blank/rest/baseline period |
| `4` | Block boundary/transition |

The trial parser pairs a baseline window (`code 3`) with the immediately following gameplay window (`code 2`). Each valid pair contributes one trial-level last-5s contrast.

Expected design:

```text
8 condition blocks x 3 repeats = 24 baseline-game pairs
```

Final valid trial counts:

- 23 participants have 24 valid baseline-game pairs.
- P23 has 23 valid baseline-game pairs.
- P13 has 24 valid pairs despite an extra unmatched gameplay marker.

Trigger exceptions:

| Participant | Issue | Lines |
|---|---|---|
| P13 | Extra unmatched `code 2` window; strict parser still keeps 24 valid pairs | `fnirsdata/P13_fnirs/2025-11-28_002_lsl.tri`, lines 7-8 |
| P23 | Unmatched/interleaved `code 2` sequence; strict parser keeps 23 valid pairs | `fnirsdata/P23_fnirs/2025-12-05_001_lsl.tri`, lines 26-29 |

## Channel Quality and Participant Retention

Primary channel-quality rule:

```text
good channel: SCI >= 0.5
bad channel:  SCI < 0.5
```

Final MNE-based bad source-detector pairs:

| Participant | Bad pair | ROI | SCI |
|---:|---|---|---:|
| 9 | S5_D5 | posterior | 0.466 |
| 13 | S1_D1 | frontal | 0.451 |
| 13 | S4_D4 | frontal | 0.438 |
| 15 | S4_D4 | frontal | 0.487 |
| 16 | S4_D4 | frontal | 0.315 |
| 17 | S3_D2 | frontal | 0.353 |
| 19 | S1_D2 | frontal | 0.372 |
| 20 | S6_D6 | posterior | 0.232 |
| 20 | S6_D7 | posterior | 0.411 |
| 21 | S5_D5 | posterior | 0.202 |

ROI channel retention after removing bad channels:

- Minimum frontal retention: 5 of 7 channels
- Minimum posterior retention: 6 of 8 channels
- No participant-region has fewer than 3 valid channels
- All 24 participants are retained for the primary analysis

P23 is retained with 23 valid trials. A strict complete-case sensitivity analysis can exclude P23 if exactly 24 trials are required.

## Primary Participant-Level Values

| participant | CL-E DLPFC hbdiff uM | CL-V occipital hbdiff uM |
|---:|---:|---:|
| 1 | 0.300841 | 1.140047 |
| 2 | -0.110874 | 0.620480 |
| 3 | -0.427343 | 0.371922 |
| 4 | -0.840167 | 0.268488 |
| 5 | 0.125777 | 0.044385 |
| 6 | -0.453250 | -0.106605 |
| 7 | -0.178321 | 0.396476 |
| 8 | -0.196154 | 0.033664 |
| 9 | -0.035260 | 0.467601 |
| 10 | -0.040227 | 1.186230 |
| 11 | -0.233758 | 1.159910 |
| 12 | -0.064731 | 0.268100 |
| 13 | -0.383733 | -0.246076 |
| 14 | -0.624638 | 0.342835 |
| 15 | 0.106446 | 0.744521 |
| 16 | -0.306496 | 0.308017 |
| 17 | -0.044800 | 0.141089 |
| 18 | 0.051518 | -0.098260 |
| 19 | -0.091249 | 0.286382 |
| 20 | -0.303882 | -0.556406 |
| 21 | -0.264723 | 0.384869 |
| 22 | -0.143769 | -0.255794 |
| 23 | 0.147288 | 0.838929 |
| 24 | 0.009576 | 0.380722 |

Summary of primary values:

| Variable | Mean | SD | Min | Max |
|---|---:|---:|---:|---:|
| CL-E frontal/DLPFC | -0.166747 | 0.258573 | -0.840167 | 0.300841 |
| CL-V posterior/occipital | 0.338397 | 0.450714 | -0.556406 | 1.186230 |

Interpretation:

- Positive value: higher `HbO - HbR` during gameplay than during baseline.
- Negative value: lower `HbO - HbR` during gameplay than during baseline.
- The value is a within-participant, within-ROI contrast, not an absolute activation level.

## Analysis

Primary analysis: the last-5s contrast because we are measuringh cognitive-load. The GLM is retained as a secondary check that asks whether a model-based full-time-series approach gives a broadly compatible activity-minus-baseline pattern.

## Analysis Decisions

Participant-specific channel removal was chosen instead of common group-level channel removal.

- Bad channels were not the same across participants.
- Removing every channel that was bad for any participant would unnecessarily discard good data.
- Under final MNE SCI, common-channel removal would reduce the frontal ROI to only 3 of 7 channels and posterior ROI to 5 of 8 channels.
- Participant-specific rejection preserves ROI coverage while still removing low-quality measurements.

Final inclusion rule:

```text
Include participant-region estimates if the ROI has at least 3 valid source-detector pairs after SCI screening.
```

All participant-regions pass this rule.

## Stuff to Report

1. No short-separation channels were collected so the pipeline cannot perform short-separation regression to remove superficial physiology. 
2. TDDR was used for motion correction, not Wavelet filtering (look for literature) 
3. SCI threshold `0.5` was used as the primary channel-quality threshold. SCI `0.3` was run as a sensitivity check.
4. P23 has 23 valid trials rather than 24. Include P23 in the primary analysis, and optionally repeat models excluding P23 as a complete-case sensitivity check.
5. HbO and HbR can be inspected separately, but the primary cognitive-load value is the oxygenation-difference contrast: `(HbO - HbR)`.
6. GLM outputs are useful for validation but do not replace the required last-5s contrast.

## References and Guidance Consulted

- MNE fNIRS preprocessing tutorial: https://mne.tools/stable/auto_tutorials/preprocessing/70_fnirs_processing.html
- MNE `scalp_coupling_index` documentation: https://mne.tools/stable/generated/mne.preprocessing.nirs.scalp_coupling_index.html
- MNE-NIRS GLM documentation: https://mne.tools/mne-nirs/stable/
- TDDR paper: Fishburn et al., 2019, Temporal Derivative Distribution Repair.
- SC paper: https://doi.org/10.1371/journal.pone.0244186 