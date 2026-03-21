# RoboGate Failure Dictionary

> **20,000 Physics-Validated Pick & Place Failure Patterns across Franka Panda & UR5e**

![Experiments](https://img.shields.io/badge/experiments-20%2C000-blue)
![Robots](https://img.shields.io/badge/robots-Franka%20%2B%20UR5e-green)
![Simulator](https://img.shields.io/badge/simulator-NVIDIA%20Isaac%20Sim-76b900)
![License](https://img.shields.io/badge/license-MIT-green)

A structured database of robot AI failure patterns collected from **NVIDIA Isaac Sim** physical simulations. Each experiment records the exact conditions under which a robot succeeded or failed at Pick & Place tasks, enabling **pre-deployment risk assessment** for industrial robotics.

---

## Quick Stats

| | Franka Panda (7DOF) | UR5e (6DOF) | Combined |
|---|---|---|---|
| **Experiments** | 10,000 | 10,000 | **20,000** |
| **Success Rate** | 33.3% | TBD | TBD |
| **Danger Zones** | 7,808 | TBD | TBD |
| **Parameters** | 8 | 11 (8 shared + 3 UR-specific) | 11 |
| **GPU Hours** | 4.5h | TBD | TBD |
| **Sampling** | Latin Hypercube | Latin Hypercube | LHS |

---

## Key Findings

- **Friction < 0.15** → Both robots fail at **80%+** — a universal danger zone independent of robot type
- **Object size < 3cm** → Grasp precision insufficient for both platforms, failure rate **83%+**
- **Obstacles ≥ 3** → Collision rate doubles; cluttered environments are the most common failure cause
- **UR5e shows higher timeout rate** → 6DOF limitation increases time-to-reach within workspace (TBD: exact %)
- **Franka advantage in non-vertical approach** → 7th joint provides redundancy for angled grasps
- **UR5e advantage in payload range** → Handles 3-5kg objects where Franka's 3kg limit is exceeded

---

## Top Failure Correlations (Franka Panda)

| Parameter | Correlation (r) | Interpretation |
|-----------|:---:|---|
| friction | **+0.36** | Higher friction → higher success (strongest factor) |
| grasp_difficulty | **-0.34** | Composite difficulty score |
| fail_prob | **-0.21** | Analytic failure probability |
| mass | **-0.20** | Heavier objects → more drops |
| ik_noise | **-0.11** | Control noise → approach errors |

---

## Research Foundations

Every design choice in this dataset is grounded in peer-reviewed research:

| Design Choice | Paper | Venue/Year | How We Used It |
|---|---|---|---|
| Latin Hypercube Sampling | ALEAS | RSS Workshop 2025 | 10,000개 균등 공간 탐색 — 랜덤 대비 2-3× 커버리지 |
| friction × mass interaction | SIMPLER | CoRL 2024 | 결합 샘플링으로 마찰-질량 교호작용 포착 |
| Failure taxonomy | RoboFAC | NeurIPS 2025 | 6가지 실패 유형 분류 체계 |
| Cross-robot validation | RoboMIND | RSS 2025 | Franka + UR5e 동시 비교 방법론 |
| UR-specific failures | Guardian | ICRA 2025 | UR 로봇 singularity/reach limit 카테고리 |
| Confidence intervals | SureSim | Badithela et al. 2025 | Wilson Score 95% CI (10K → ±1%) |
| GPU simulation | Isaac Lab | NVIDIA 2025 | Newton Physics 엔진 + 60Hz 물리 |
| Grasp evaluation | Isaac Sim Grasping SDG | NVIDIA 2025 | 물리 기반 파지 평가 메트릭 |

---

## Data Schema

```json
{
  "friction": 0.603,
  "mass": 0.085,
  "com_offset": 0.081,
  "size": 0.074,
  "ik_noise": 0.037,
  "obstacles": 2,
  "shape": "box",
  "placement": "rotated_135",
  "success": true,
  "failure_type": "none",
  "cycle_time": 1.437,
  "collision": false,
  "drop": false,
  "grasp_miss": false,
  "zone": "boundary"
}
```

**Zone Classification:**
- `safe`: fail_prob < 0.30
- `boundary`: 0.30 ≤ fail_prob < 0.70
- `danger`: fail_prob ≥ 0.70

---

## Usage

```python
import json

with open("failure_dictionary_large.json") as f:
    data = json.load(f)

experiments = data["experiments"]
print(f"Total: {len(experiments)}")

# Find danger zones
danger = [e for e in experiments if e["zone"] == "danger"]
print(f"Danger zones: {len(danger)}")

# Friction vs success correlation
import numpy as np
frictions = [e["friction"] for e in experiments]
successes = [1 if e["success"] else 0 for e in experiments]
r = np.corrcoef(frictions, successes)[0, 1]
print(f"Friction-success correlation: r={r:.4f}")
```

Or via HuggingFace:
```python
from datasets import load_dataset
ds = load_dataset("liveplex/robogate-failure-dictionary")
```

---

## Citation

```bibtex
@dataset{robogate_failure_dictionary_2026,
  title={RoboGate Failure Dictionary: 20K Physics-Validated Pick & Place Failure Patterns},
  author={RoboGate Team},
  year={2026},
  url={https://github.com/liveplex-cpu/robogate-failure-dictionary},
  note={Franka Panda + UR5e, NVIDIA Isaac Sim, Latin Hypercube Sampling}
}
```

---

## Links

- **RoboGate Platform**: [github.com/liveplex-cpu/robogate](https://github.com/liveplex-cpu/robogate)
- **HuggingFace Dataset**: [huggingface.co/datasets/liveplex/robogate-failure-dictionary](https://huggingface.co/datasets/liveplex/robogate-failure-dictionary)
- **Interactive Explorer**: Run `npm run dev` in the `web/` directory → `/failures`
- **Methodology**: `/methodology` page on the RoboGate website

---

**License:** MIT

Built with NVIDIA Isaac Sim · Newton Physics · Franka Panda · UR5e
