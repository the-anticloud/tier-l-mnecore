# L5 Narrow / L2 General Classification — L_MNECORE
**Platform:** Anticloud | **Tier:** TIER_7_BIOSIGNALS_NEURO | **PAX:** 27B
**IP:** USPTO pending 2026, Anticloud FZ LLE, 0-1.gg | **License:** Apache-2.0

## L5 Narrow
L_MNECORE integrates MNE-Python's full analysis suite into Anticloud: epochs, evoked potentials, source localization, and time-frequency analysis. Narrow scope: Anticloud clinical EEG/MEG — not general neuroimaging.

## L2 General
L2 General: L_MNECORE is the core EEG analysis library for all TIER_7 clinical applications. K_PYPREP, K_NEURODSP, and L_BRAINDECODE all feed through L_MNECORE's data structures.

## PAX 27B Integration
PAX 27B interprets MNE analysis results for clinical reporting: 'N200 ERP amplitude is significantly reduced at Fz and Cz — consider evaluation for attention processing deficits.'

## AIOSS Audit Chain
Every EEG analysis (epochs hash + evoked hash + source localization hash + PAX clinical report hash) is chained: H_n = SHA3-256(H_{n-1} || entry_hash_n || timestamp_n).
Offline-verifiable, tamper-evident, zero cloud dependency.

## Regulatory / Compliance
IEC 62304 (medical software). HIPAA. EU MDR. Good Clinical Practice (GCP).
