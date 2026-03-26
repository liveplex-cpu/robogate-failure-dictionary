# RoboGate Failure Dictionary

> **50,000+ Physics-Validated Pick & Place Failure Patterns across 4 Robots (Franka Panda, UR5e, UR3e, UR10e)**

![Experiments](https://img.shields.io/badge/experiments-50%2C000%2B-blue)
![Robots](https://img.shields.io/badge/robots-Franka%20%2B%20UR5e%20%2B%20UR3e%20%2B%20UR10e-green)
![AUC](https://img.shields.io/badge/Risk%20Model%20AUC-0.777-orange)
![Simulator](https://img.shields.io/badge/simulator-NVIDIA%20Isaac%20Sim-76b900)
![License](https://img.shields.io/badge/license-MIT-green)

A structured database of robot AI failure patterns collected from **NVIDIA Isaac Sim** physical simulations using **Two-Stage Adaptive Sampling**. Each experiment records the exact conditions under which a robot succeeded or failed at Pick & Place tasks, enabling **pre-deployment risk assessment** for industrial robotics.

---

## Quick Stats

| | Franka Uniform | Franka Boundary | UR5e | UR3e | UR10e | Combined |
|---|---|---|---|---|---|---|
| **Experiments** | 10,000 | 10,000 | 10,000 | 10,000 | 10,000 | **50,000+** |
| **Success Rate** | 33.3% | 63.8% | 74.3% | — | — | — |
| **Franka Combined** | — | 48.6% | — | — | — | — |
| **Danger Zones** | 7,808 | — | 2,570 | — | — | 10,378+ |
| **Risk Model AUC** | 0.65 | **0.777** | — | — | — | 0.777 |
| **Parameters** | 8 | 8 | 11 | 11 | 11 | 11 |
| **Sampling** | Uniform LHS | Boundary LHS | Uniform LHS | Uniform LHS | Uniform LHS | Two-Stage |

---

## Two-Stage Adaptive Sampling

### Stage 1 — Uniform Exploration (40,000)
- **Franka Panda 10K** + **UR5e 10K** + **UR3e 10K** + **UR10e 10K** via Latin Hypercube Sampling
- Uniform parameter space coverage — 2-3× better than random
- Identified boundary regions and initial risk model (AUC 0.65)

### Stage 2 — Boundary-Focused (10,000)
- **Franka Panda only**, targeting boundary/transition regions
- Concentrated sampling near friction threshold μ* = 0.492
- Revealed failure mode transitions invisible to uniform sampling
- **Boosted Risk Model AUC to 0.777 (+19.5%)**

### Result
- Boundary equation: **μ*(m) = (1.469 + 0.419·m) / (3.691 - 1.400·m)**
- Failure mode transition discovered: friction↓ → timeout → collision → grasp_miss

---

## Key Findings

- **friction × mass interaction z = -10.00** — strongest predictor of failure across all 4 robots
- **Friction threshold: μ* = 0.492 ± 0.031** — below this, failure cascades through modes
- **Mass > 0.93 kg** → Both robots fail at **< 40%** SR (universal danger zone)
- **UR5e never drops** → SurfaceGripper (suction) with breakForce=MAX; all failures are `grasp_miss`
- **2.2× success gap** → UR5e 74.3% vs Franka 33.3% (z = -58.15, p < 0.001)
- **AUC 0.65 → 0.777** (+19.5%) with boundary-focused sampling

### Universal Danger Zones (mass > 0.93 kg)

| Mass Range | Franka SR | UR5e SR |
|---|---|---|
| 0.93 – 1.23 kg | 21.4% | 30.9% |
| 1.23 – 1.52 kg | 14.9% | 25.3% |
| 1.52 – 1.82 kg | 12.5% | 28.9% |
| 1.82 – 2.11 kg | 6.6% | 28.1% |

---

## 4-Robot Comparison

| | Franka Panda | UR5e | UR3e | UR10e |
|---|---|---|---|---|
| **Experiments** | 20,000 | 10,000 | 10,000 | 10,000 |
| **Success Rate** | 48.6% (20K combined) | 74.3% | — | — |
| **Failure Modes** | grasp_miss, drop, collision | grasp_miss only | — | — |
| **Gripper** | Finger (parallel jaw) | SurfaceGripper (suction) | — | — |
| **DOF** | 7 | 6 | 6 | 6 |
| **Collision Count** | 1,124 (uniform 10K) | ~0 | — | — |

---

## Top Failure Correlations

### Franka Panda

| Parameter | Correlation (r) | Interpretation |
|-----------|:---:|---|
| friction | **+0.36** | Higher friction → higher success (strongest factor) |
| mass | **-0.20** | Heavier objects → more drops |
| ik_noise | **-0.11** | Control noise → approach errors |

### UR5e

| Parameter | Correlation (r) | Interpretation |
|-----------|:---:|---|
| mass | **-0.35** | Heavier objects → suction failure (strongest factor) |
| friction | **+0.18** | Moderate effect (suction less friction-dependent) |
| ik_noise | **-0.12** | Control noise → approach miss |

### Cross-Robot Interactions

| Interaction | z-score | Interpretation |
|---|---|---|
| friction × mass | **-10.00** | Strongest predictor — low friction + high mass = catastrophic |
| friction threshold | **0.492 ± 0.031** | Decision boundary for success/failure |

---

## Research Foundations

| Design Choice | Paper | Venue/Year | How We Used It |
|---|---|---|---|
| Two-Stage Adaptive Sampling | ALEAS | RSS Workshop 2025 | Stage 1 uniform 20K + Stage 2 boundary 10K |
| friction × mass interaction | SIMPLER | CoRL 2024 | Joint sampling, interaction z-test |
| Failure taxonomy | RoboFAC | NeurIPS 2025 | 6 failure type classification |
| Cross-robot validation | RoboMIND | RSS 2025 | Franka + UR5e simultaneous comparison |
| UR-specific failures | Guardian | ICRA 2025 | UR robot singularity/reach categories |
| Confidence intervals | SureSim | Badithela et al. 2025 | Wilson Score 95% CI (50K+ → ±0.4%) |
| GPU simulation | Isaac Lab | NVIDIA 2025 | Newton Physics + 60Hz physics |

---

## Data Files

| File | Robot | Experiments | Sampling | Description |
|---|---|---|---|---|
| `failure_dictionary_large.json` | Franka | 10,000 | Uniform LHS | Stage 1 uniform exploration |
| `franka_boundary_10k.json` | Franka | 10,000 | Boundary LHS | Stage 2 boundary-focused |
| `ur5e_failure_dictionary.json` | UR5e | 10,000 | Uniform LHS | Stage 1 uniform exploration |
| `ur3e_failure_dictionary.json` | UR3e | 10,000 | Uniform LHS | Stage 1 uniform exploration |
| `ur10e_failure_dictionary.json` | UR10e | 10,000 | Uniform LHS | Stage 1 uniform exploration |

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

# Load all Franka data (uniform + boundary)
with open("failure_dictionary_large.json") as f:
    franka_uniform = json.load(f)["experiments"]
with open("franka_boundary_10k.json") as f:
    franka_boundary = json.load(f)["experiments"]
franka_all = franka_uniform + franka_boundary
print(f"Franka total: {len(franka_all)}")  # 20,000

# Find mass danger zones
import numpy as np
heavy = [e for e in franka_all if e["mass"] > 0.93]
sr = sum(1 for e in heavy if e["success"]) / len(heavy)
print(f"Mass > 0.93kg SR: {sr:.1%}")  # ~21%
```

Or via HuggingFace:
```python
from datasets import load_dataset
ds = load_dataset("liveplex/robogate-failure-dictionary")
# Splits: franka, franka_boundary, ur5e, ur3e, ur10e, train (all 50K+ combined)
```

---

## Citation

```bibtex
@dataset{robogate_failure_dictionary_2026,
  title={RoboGate Failure Dictionary: 50K+ Physics-Validated Pick & Place Failure Patterns},
  author={RoboGate Team},
  year={2026},
  url={https://github.com/liveplex-cpu/robogate-failure-dictionary},
  note={Franka Panda + UR5e + UR3e + UR10e, Two-Stage Adaptive Sampling, AUC 0.777}
}
```

---

## VLA Benchmark — 4-Model Leaderboard

Four VLA models evaluated on RoboGate's 68-scenario adversarial suite. **All scored 0% SR** — including NVIDIA's official GR00T N1.6.

| Model | Params | SR | Confidence | Failure Pattern |
|-------|--------|-----|-----------|-----------------|
| Scripted Controller | — | **100%** (68/68) | 76/100 | — |
| **GR00T N1.6 (NVIDIA)** | 3B | 0% (0/68) | 1/100 | grasp_miss + collision |
| OpenVLA (Stanford + TRI) | 7B | 0% (0/68) | 27/100 | grasp_miss dominant, 0 collision |
| Octo-Base (UC Berkeley) | 93M | 0% (0/68) | 1/100 | grasp_miss 79%, collision 21% |
| Octo-Small (UC Berkeley) | 27M | 0% (0/68) | 1/100 | grasp_miss 79.4%, collision 20.6% |

Model size is not the bottleneck — even NVIDIA's flagship 3B model cannot bridge the training-deployment distribution gap.

**Leaderboard:** [robogate.io/vla](https://robogate.io/vla) · **Paper:** [arXiv:2603.22126](https://arxiv.org/abs/2603.22126)

---

## Links

- **RoboGate Platform**: [github.com/liveplex-cpu/robogate](https://github.com/liveplex-cpu/robogate)
- **HuggingFace Dataset**: [huggingface.co/datasets/liveplex/robogate-failure-dictionary](https://huggingface.co/datasets/liveplex/robogate-failure-dictionary)
- **Interactive Explorer**: Run `npm run dev` in the `web/` directory → `/failures`
- **Methodology**: `/methodology` page on the RoboGate website

---

**License:** MIT

Built with NVIDIA Isaac Sim · Newton Physics · Franka Panda · UR5e · UR3e · UR10e · Two-Stage Adaptive Sampling
