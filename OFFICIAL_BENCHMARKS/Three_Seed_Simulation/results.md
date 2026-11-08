# Three_Seed_Simulation — L_MNECORE

**Run:** 2026-09-30T18:11:11.493145+00:00

```json
{
  "project": "L_MNECORE",
  "seeds": [
    34007,
    65344,
    99543
  ],
  "seed_method": "sha256(name)[:8]+offsets[0,31337,65536]",
  "helm_standard": true,
  "run_ts": "2026-09-30T18:11:11.493145+00:00",
  "metrics": {
    "throughput_tokens_per_sec": {
      "seeds": [
        34007,
        65344,
        99543
      ],
      "values": [
        18.86,
        55.363,
        58.724
      ],
      "mean": 44.316,
      "std": 18.052,
      "ci_95_low": 8.934,
      "ci_95_high": 79.698
    },
    "latency_p50_ms": {
      "seeds": [
        34007,
        65344,
        99543
      ],
      "values": [
        69.745,
        81.203,
        9.608
      ],
      "mean": 53.519,
      "std": 31.4,
      "ci_95_low": -8.025,
      "ci_95_high": 115.063
    },
    "memory_mb": {
      "seeds": [
        34007,
        65344,
        99543
      ],
      "values": [
        37.67,
        27.868,
        97.807
      ],
      "mean": 54.448,
      "std": 30.919,
      "ci_95_low": -6.153,
      "ci_95_high": 115.049
    },
    "accuracy_score": {
      "seeds": [
        34007,
        65344,
        99543
      ],
      "values": [
        34.942,
        44.9,
        74.806
      ],
      "mean": 51.549,
      "std": 16.94,
      "ci_95_low": 18.347,
      "ci_95_high": 84.751
    },
    "f1_score": {
      "seeds": [
        34007,
        65344,
        99543
      ],
      "values": [
        4.61,
        33.141,
        64.747
      ],
      "mean": 34.166,
      "std": 24.562,
      "ci_95_low": -13.976,
      "ci_95_high": 82.308
    },
    "inference_tps": {
      "seeds": [
        34007,
        65344,
        99543
      ],
      "values": [
        26.586,
        40.709,
        86.723
      ],
      "mean": 51.339,
      "std": 25.676,
      "ci_95_low": 1.014,
      "ci_95_high": 101.664
    },
    "quality_score": {
      "seeds": [
        34007,
        65344,
        99543
      ],
      "values": [
        9.352,
        54.203,
        69.488
      ],
      "mean": 44.348,
      "std": 25.52,
      "ci_95_low": -5.671,
      "ci_95_high": 94.367
    }
  }
}
```
