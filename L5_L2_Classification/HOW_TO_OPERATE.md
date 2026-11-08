# How to Operate — L_MNECORE
**Platform:** Anticloud | **IP:** USPTO pending 2026, Anticloud FZ LLE, 0-1.gg

## Module Overview
L_MNECORE — MNE-Core: MNE-Python integration for Anticloud clinical EEG/MEG analysis
Stack: Python 3.11, mne 1.7+, numpy, scipy, PAX 27B, AIOSS_FORMAT

## Daily Operations
1. `aioss verify --chain ./l_mnecore.aioss`
2. Check service health via api-oss-monitor
3. Review api-oss-logging for error-level events
4. Confirm PAX 27B is loaded and responding

## Incident Response
- Chain tamper: halt, notify compliance, restore from backup
- GPU OOM: reduce batch size, check memory leak
- High latency >2s P99: check queue depth, scale workers
- Compliance gap: run api-oss-compliance report

## Backup (nightly)
```bash
python -m api_oss_backup backup --sources ./l_mnecore.aioss --output ./backups/
```
