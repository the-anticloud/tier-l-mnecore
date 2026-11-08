# Developer Cookbook — L_MNECORE
**Stack:** Python 3.11, mne 1.7+, numpy, scipy, PAX 27B, AIOSS_FORMAT
**Domain:** MNE-Core: MNE-Python integration for Anticloud clinical EEG/MEG analysis

## EEG source localization
```python
from l_mnecore import MNECoreAnalyzer

analyzer = MNECoreAnalyzer(
    pax_model="./pax-27b-q4.gguf",
    aioss_chain="./mnecore.aioss"
)

raw = mne.io.read_raw_edf("./patient_001.edf", preload=True)
result = analyzer.full_pipeline(
    raw=raw,
    event_id={"stimulus/left": 1},
    tmin=-0.2, tmax=0.6,
    analyses=["erp", "tfr", "source_loc"]
)
print(f"N200 amplitude: {result.erp.n200_amplitude:.2f} µV")
print(f"Source peak: {result.source_peak_location}")
print(f"PAX clinical report: {result.pax_report}")
print(f"Chain: {result.chain_hash}")
```

## AIOSS Chain Append
```python
import hashlib, time

def aioss_append(chain_path, payload: bytes, module_id: str):
    entry_hash = hashlib.sha3_256(payload).digest()
    ts = int(time.time_ns()).to_bytes(8, 'big')
    with open(chain_path, 'rb') as f:
        f.seek(-32, 2); prev_hash = f.read(32)
    new_hash = hashlib.sha3_256(prev_hash + entry_hash + ts).digest()
    with open(chain_path, 'ab') as f:
        f.write(ts + entry_hash + new_hash)
    return new_hash.hex()
```
