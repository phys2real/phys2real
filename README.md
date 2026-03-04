<div align="center">

# Phys2Real

**Fusing VLM Priors with Interactive Online Adaptation for Uncertainty-Aware Sim-to-Real Manipulation**

[[Paper]](https://arxiv.org/abs/2510.11689) &emsp; [[Project Page]](https://phys2real.github.io/) &emsp; ICRA 2026

Maggie Wang, Stephen Tian, Aiden Swann, Ola Shorinwa, Jiajun Wu, Mac Schwager

<img src="https://raw.githubusercontent.com/phys2real/phys2real.github.io/master/static/images/phys2real_xl_clean.gif" width="100%">

</div>

## Overview

Learning robotic manipulation policies directly in the real world can be expensive and time-consuming. While reinforcement learning (RL) policies trained in simulation present a scalable alternative, effective sim-to-real transfer remains challenging, particularly for tasks that require precise dynamics. Phys2Real is a real-to-sim-to-real RL pipeline that combines vision-language model (VLM)-inferred physical parameter estimates with interactive adaptation through uncertainty-aware fusion. Our approach consists of three core components:

1. **High-fidelity geometric reconstruction** with 3D Gaussian splatting
2. **VLM-inferred prior distributions** over physical parameters
3. **Online physical parameter estimation** from interaction data

Phys2Real conditions policies on interpretable physical parameters, refining VLM predictions with online estimates via ensemble-based uncertainty quantification. We evaluate on two planar pushing tasks with a UFACTORY xArm 6 robot: T-block pushing with varying center of mass (CoM) and hammer pushing with an off-center mass distribution.

## 📦 Repositories

### [`isaaclab-phys2real`](https://github.com/phys2real/isaaclab-phys2real) — Simulation & Training
Fork of [Isaac Lab](https://github.com/isaac-sim/IsaacLab) with custom manager-based task definitions for planar pushing.
- **Environment configuration** (`push_env_cfg.py`): observation space (end-effector XY position, object pose, goal pose, privileged physical parameters), action space (delta end-effector commands via differential IK), reward functions (distance-to-goal, orientation alignment, action penalties), and episode termination conditions
- **Domain randomization**: center of mass (COM), friction coefficients, and initial object pose randomized during training to enable sim-to-real transfer; mass randomization for hammer
- **Robot and object configs** (`config/xarm/`): UFACTORY xArm 6 with differential IK controller, T-block asset (200g, variable COM), and hammer asset (620g, off-center mass distribution)
- **Custom MDP modules** (`mdp/`): observation functions, reward terms, termination conditions, and action processing

### [`rl_games_isaacsim`](https://github.com/phys2real/rl_games_isaacsim) — RL Framework & Encoders
Fork of [rl_games](https://github.com/Denys88/rl_games) with custom network architectures for the Phys2Real adaptation framework.
- **Physics encoder** (`encoders.py`): identity mapping that passes privileged physical parameters (COM) directly as the latent z during training, serving as the teacher signal for the history encoder
- **History encoder** (`encoders.py`): MLP that predicts the physics latent z from a flattened window of observation-action history, enabling online adaptation without privileged information at deployment
- **Ensemble uncertainty** (`encoders.py`): multiple history encoder heads providing uncertainty estimates over predicted physical properties, used to weight VLM priors vs. online estimates
- **VLM prior fusion** (`models.py`): at deployment, combines VLM-predicted physical properties with ensemble predictions using inverse variance weighting
- **Training phases** (`encoders_cfg.py`): multi-phase training — Phase 1 (base policy with privileged info), Phase 1.5 (base policy with COM noise for robustness), Phase 2 (history encoder fine-tuning with ensemble and VLM prior fusion)

### [`sim-to-real-rl`](https://github.com/phys2real/sim-to-real-rl) — Real-World Deployment
Deployment and evaluation code for running trained policies on hardware.
- **Real robot environment** (`real_env.py`): gym-compatible environment wrapping the xArm 6, end-effector control via xArm SDK, object pose tracking via OptiTrack motion capture (ROS2), observation construction matching the simulation format, and workspace boundary enforcement
- **Policy execution** (`play_real.py`): loads trained checkpoints and runs the policy loop, with model selection based on active training configuration (DR, RMA phase 1, RMA phase 2 with ensemble)
- **VLM physical property estimation** (`vlm/prompt_gpt_for_com.py`): queries an OpenAI model with RGB images of the object to estimate center of mass along with an uncertainty estimate
- **Camera calibration** (`camera_calibration/`): intrinsic calibration for Intel RealSense D435 cameras using checkerboard patterns
- **Data logging and analysis** (`data_logger.py`, `analysis/`): per-timestep logging of object poses, end-effector poses, actions, latent z values, and ensemble/VLM estimates; visualization scripts for trajectory plots and figures
- **Utilities** (`utils.py`): policy loading, action-to-joint conversion, and RL-Games configuration

## 🔧 Installation

### Clone

Clone all repositories at once:
```bash
git clone --recursive https://github.com/phys2real/phys2real.git
cd phys2real
```

Or individually:
```bash
git clone https://github.com/phys2real/isaaclab-phys2real.git
git clone https://github.com/phys2real/rl_games_isaacsim.git
git clone https://github.com/phys2real/sim-to-real-rl.git
```

### Prerequisites

- Ubuntu 22.04
- Python 3.10+
- NVIDIA GPU with CUDA 12+
- [Isaac Sim 4.0+](https://developer.nvidia.com/isaac-sim)
- [xArm-Python-SDK](https://github.com/xArm-Developer/xArm-Python-SDK)
- [Intel RealSense SDK](https://github.com/IntelRealSense/librealsense) + `pyrealsense2`
- ROS2 Humble
- [SuGaR](https://github.com/Anttwo/SuGaR) (for scene reconstruction)

### Setup

1. **Training environment** (Isaac Lab):
```bash
cd isaaclab-phys2real
# Follow Isaac Lab installation: https://isaac-sim.github.io/IsaacLab/
pip install -e source/isaaclab_tasks
```

2. **RL training framework**:
```bash
cd rl_games_isaacsim
pip install -e .
```

3. **Deployment code**:
```bash
cd sim-to-real-rl
pip install -r requirements.txt
```

## 🚀 Pipeline

1. **Define task** in `isaaclab-phys2real` — environment, rewards, domain randomization
2. **Train policy** with `rl_games_isaacsim` — base policy → RMA adaptation → ensemble + VLM fusion
3. **Deploy** with `sim-to-real-rl` — run on real xArm with fused VLM priors

## Acknowledgments

This work was conducted at the [Multi-Robot Systems Lab](https://msl.stanford.edu/) and the [Stanford Vision and Learning Lab](https://svl.stanford.edu/) at Stanford University. This work was supported in part by the NASA Space Technology Graduate Research Opportunities (NSTGRO) Fellowship.

## Citation

```bibtex
@inproceedings{wang2026phys2real,
  title={Phys2Real: Fusing VLM Priors with Interactive Online Adaptation
         for Uncertainty-Aware Sim-to-Real Manipulation},
  author={Wang, Maggie and Tian, Stephen and Swann, Aiden and Shorinwa, Ola
          and Wu, Jiajun and Schwager, Mac},
  booktitle={IEEE International Conference on Robotics and Automation (ICRA)},
  year={2026}
}
```

## License

- `isaaclab-phys2real`: BSD-3-Clause
- `rl_games_isaacsim`: MIT
- `sim-to-real-rl`: MIT
