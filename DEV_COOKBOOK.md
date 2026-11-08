# Dev Cookbook — L_MNECORE (Anticloud Integration)
**Component:** L_MNECORE (MNE-Python) | **Tier:** T7 Biosignals/Neuro  
**Anticloud Layer:** AIOSS Ledger + EEG/MEG sovereign processing  
**Date:** 2026-09-30

---

## What This Achieves

L_MNECORE wraps MNE-Python (Gramfort et al., 2013, *Frontiers in Neuroscience*) with AIOSS ledger integration, enabling:
- **Sovereign BCI pipelines** — raw EEG → cleaned signal → feature extraction, entirely on-device
- **Cryptographic audit** of every neural data processing step (SHA3-256 chain)
- **GDPR/HIPAA compliance** — neural data never leaves the deployment infrastructure

---

## Prerequisites

```bash
pip install mne>=1.6.0 numpy scipy antichain-aioss
# AIOSS binary
aioss --version
```

---

## 1. Load and Process EEG — Minimal Pipeline

```python
import mne
from aioss_integration import AIossLedger

ledger = AIossLedger("./bci_session.aioss")

# Load raw EEG (BrainFlow, OpenBCI, or .fif file)
raw = mne.io.read_raw_fif("subject_01_rest.fif", preload=True)
info = raw.info
print(f"Channels: {len(raw.ch_names)}, Sfreq: {info['sfreq']} Hz")

# Log data receipt to ledger
ledger.append(
    event="data_loaded",
    metadata={"subject": "01", "n_channels": len(raw.ch_names),
               "sfreq": info["sfreq"], "duration_s": raw.times[-1]}
)

# Band-pass filter: remove DC drift and high-freq noise
raw.filter(l_freq=1.0, h_freq=40.0, method="fir", fir_design="firwin")
ledger.append(event="filter_applied", metadata={"l_freq": 1.0, "h_freq": 40.0})

# Independent Component Analysis for artifact removal
ica = mne.preprocessing.ICA(n_components=20, random_state=42, max_iter=800)
ica.fit(raw)
# Automatically detect eye-blink components
eog_indices, eog_scores = ica.find_bads_eog(raw)
ica.exclude = eog_indices
raw_clean = ica.apply(raw.copy())
ledger.append(event="ica_artifact_removal",
              metadata={"excluded_components": eog_indices, "n_ica": 20})

print(f"Pipeline complete. Chain hash: {ledger.chain_hash}")
```

---

## 2. Epoching and Event-Related Potentials (ERPs)

```python
# Extract events from stimulus channel
events, event_id = mne.events_from_annotations(raw_clean)
# e.g., event_id = {'stimulus/target': 1, 'stimulus/nontarget': 2}

# Epoch: -200ms to +800ms around each event
epochs = mne.Epochs(
    raw_clean, events, event_id=event_id,
    tmin=-0.2, tmax=0.8,
    baseline=(-0.2, 0.0),    # baseline correction
    preload=True, reject={"eeg": 100e-6}   # reject high-amplitude artifacts
)

# Average to get ERP
evoked_target = epochs["stimulus/target"].average()
evoked_nontarget = epochs["stimulus/nontarget"].average()

# Log epoch stats
ledger.append(event="epochs_created", metadata={
    "n_target": len(epochs["stimulus/target"]),
    "n_nontarget": len(epochs["stimulus/nontarget"]),
    "rejection_threshold_uv": 100,
})

# P300 peak (target - nontarget at Pz, ~300ms)
import numpy as np
pz_idx = evoked_target.ch_names.index("Pz")
times = evoked_target.times
p300_window = (times >= 0.25) & (times <= 0.45)
p300_amplitude = np.max(evoked_target.data[pz_idx][p300_window]) * 1e6  # µV
print(f"P300 amplitude at Pz: {p300_amplitude:.2f} µV")
ledger.append(event="erp_computed", metadata={"p300_amplitude_uv": round(p300_amplitude, 2)})
```

---

## 3. Real-Time BCI Stream (BrainFlow Integration)

```python
from brainflow.board_shim import BoardShim, BrainFlowInputParams, BoardIds
from brainflow.data_filter import DataFilter, FilterTypes
import mne
import time

# Initialize board (OpenBCI Cyton = CYTON_BOARD = 0)
params = BrainFlowInputParams()
params.serial_port = "/dev/ttyUSB0"
board = BoardShim(BoardIds.CYTON_BOARD, params)
board.prepare_session()
board.start_stream()

eeg_channels = BoardShim.get_eeg_channels(BoardIds.CYTON_BOARD)
srate = BoardShim.get_sampling_rate(BoardIds.CYTON_BOARD)

ledger.append(event="stream_started", metadata={
    "board": "OpenBCI Cyton", "srate": srate, "n_eeg_channels": len(eeg_channels)
})

try:
    while True:
        data = board.get_current_board_data(srate)  # last 1 second
        eeg = data[eeg_channels, :]  # shape: (8, srate)

        # Online bandpass filter
        for ch in range(eeg.shape[0]):
            DataFilter.perform_bandpass(eeg[ch], srate, 1.0, 40.0, 4,
                                         FilterTypes.BUTTERWORTH, 0)

        # Feature: band power
        from scipy.signal import welch
        freqs, psd = welch(eeg, fs=srate, nperseg=srate//2)
        alpha_mask = (freqs >= 8) & (freqs <= 13)
        alpha_power = float(psd[:, alpha_mask].mean())

        ledger.append(event="realtime_sample", metadata={
            "timestamp": time.time(), "alpha_power": round(alpha_power, 6)
        })
        time.sleep(0.25)   # 4 Hz update
finally:
    board.stop_stream()
    board.release_session()
    ledger.append(event="stream_stopped")
    print(f"Final ledger chain hash: {ledger.chain_hash}")
```

---

## 4. AIOSS Ledger for HIPAA Audit Trail

Neural data is PHI (Protected Health Information) under HIPAA. The AIOSS ledger provides:

```python
# Every processing step is auditable
aioss_records = ledger.export_json()

# Verify integrity (e.g., for regulatory submission)
assert ledger.verify(), "Chain compromised — do not submit!"

# Export for IRB / FDA submission
ledger.export(
    path="./audit_trail.json",
    include_metadata=True,
    redact_raw_hashes=False  # auditors need full chain
)
```

HIPAA compliance checklist for L_MNECORE:
- [x] Neural data never leaves on-prem infrastructure
- [x] All processing steps cryptographically logged (SHA3-256)
- [x] Access log maintained in AIOSS ledger
- [x] Data encrypted at rest (OS-level FDE + AIOSS hash verification)

---

## 5. MOABB Benchmark Protocol

```python
# Run MOABB evaluation on your pipeline (SOTA comparison)
from moabb.evaluations import WithinSessionEvaluation
from moabb.datasets import BNCI2014_001
from moabb.paradigms import MotorImagery

paradigm = MotorImagery(n_classes=2, fmin=8., fmax=35.)
dataset = BNCI2014_001()

# Your Anticloud pipeline as a sklearn Pipeline
from sklearn.pipeline import Pipeline
from mne.decoding import CSP
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis

pipeline = Pipeline([
    ("CSP", CSP(n_components=8, reg="ledoit_wolf", log=True)),
    ("LDA", LinearDiscriminantAnalysis()),
])

evaluation = WithinSessionEvaluation(
    paradigm=paradigm, datasets=[dataset], overwrite=True
)
results = evaluation.process({"Anticloud-CSP-LDA": pipeline})
print(results.groupby("pipeline")[["score"]].mean())

# Log MOABB results to AIOSS ledger
ledger.append(event="moabb_benchmark", metadata={
    "dataset": "BNCI2014_001", "paradigm": "MotorImagery",
    "mean_accuracy": float(results["score"].mean()),
    "reference": "Jayaram & Barachant, 2018, J Neural Eng"
})
```

---

## 6. References

1. Gramfort, A., et al. (2013). MEG and EEG data analysis with MNE-Python. *Frontiers in Neuroscience*, 7, 267.
2. Jayaram, V. & Barachant, A. (2018). MOABB: trustworthy algorithm benchmarking for BCIs. *Journal of Neural Engineering*, 15(6).
3. Wolpaw, J.R., et al. (2002). Brain-computer interfaces. *Clinical Neurophysiology*, 113(6), 767–791.
4. AIOSS Ledger spec: `TIER_1_ANTICLOUD_CORE/AIOSS_FORMAT/`
5. NIST FIPS 202 (SHA3): https://csrc.nist.gov/publications/detail/fips/202/final
