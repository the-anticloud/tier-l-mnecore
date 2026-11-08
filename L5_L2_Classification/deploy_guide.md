# Deploy Guide — L_MNECORE
**Tier:** TIER_7_BIOSIGNALS_NEURO | **Stack:** Python 3.11, mne 1.7+, numpy, scipy, PAX 27B, AIOSS_FORMAT
**Air-gap capable after initial setup.**

## Prerequisites
Python 3.11+, mne 1.7+, numpy 1.26+, scipy 1.13+. CPU for most analyses. GPU for PAX.

## Environment
16GB RAM for large datasets. CPU-intensive. Plotting requires display/backend.

## AIOSS Integration
```bash
aioss init --module L_MNECORE --output ./l_mnecore.aioss
aioss append --chain ./l_mnecore.aioss --payload ./output.bin --module L_MNECORE
aioss verify --chain ./l_mnecore.aioss
```

## Air-Gap Setup
```bash
pip download -r requirements.txt -d ./wheels/
pip install --no-index --find-links ./wheels/ -r requirements.txt
```

## PAX 27B Harness Wiring
```python
from anticloud_pax import PAXHarness
harness = PAXHarness(
    model_path="./pax-27b-q4.gguf",
    module="L_MNECORE",
    aioss_chain="./L_MNECORE.aioss",
    classification="L5_NARROW_L2_GENERAL"
)
result = harness.process(input_data)
```

## Verification
```bash
aioss verify --chain ./L_MNECORE.aioss --verbose
python -m L_MNECORE.tests.smoke
```
