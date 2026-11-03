# Position Generalization from ~20 Real Demonstrations: A Pre-Registered Real-Robot Comparison of Cosmos Predict, Cosmos Transfer, and Isaac Sim + NuRec (3DGS) Augmentation

**Logan Lee, Megazone Cloud** · NVIDIA GTC 2027 poster companion repository

## Abstract

Vision-language-action (VLA) policies fine-tuned on about 20 real demonstrations work near the object positions they have seen but fail at unseen ones. Using a low-cost SO-101 arm on a pick-and-place task, we train a π0.5 policy on 20 real demonstrations recorded only on the **left** side of the table and ask which augmentation method lets it succeed on the **right** side without recording any more real data. We compare, step by step, Cosmos Predict 2.5 video generation with inverse-dynamics (IDM) labels, Isaac Sim + NuRec (3DGS) MimicGen-style motion generation with exact simulator labels, Predict re-post-trained on simulator data, and Cosmos Transfer 2.5 re-rendering of simulator episodes. Early, informal real-robot runs suggest that simulator-generated right-side episodes with exact labels fix the direction the arm turns, while video-generated clips labeled by an IDM do not. A controlled 20-spot evaluation is pending.

## Hypothesis

All augmentation is generated from the **same fixed 20 real left-side demonstrations**. No additional real data is ever recorded. Seeds, timing templates and real-to-sim calibration come only from these 20 demos.

## Setup

- **Robot:** SO-101 arm (5-DoF + gripper), one front camera (640×480).
- **Task:** pick up the blue block and put it in the gray bowl. Success = the block is inside the bowl when the gripper opens.
- **Workspace:** the real workspace (pegboard, bowl, fixtures) was built with a 3D printer. A NuRec 3DGS (3DGUT) reconstruction of it is used as the background of a digital twin in Isaac Sim, with the camera calibrated to the real one.

## Steps

| Step | Training data for π0.5 | Purpose |
|---|---|---|
| 1 | real_left 20 | Baseline: left works, right fails |
| 2 | real_left 20 + Cosmos Predict 2.5 clips at right spots, IDM-labeled | Show that IDM labels fail at new positions |
| 3 | real_left 20 + Isaac Sim + NuRec MimicGen episodes at 20 right spots (exact sim labels) | Right-side data with correct labels |
| 4 | real_left 20 + Predict (re-post-trained on real_left + sim right) clips, labeled by a sim-fine-tuned IDM | Does a better IDM + Predict close the gap? |
| 5 | real_left 20 + Cosmos Transfer 2.5 re-renders of the Step 3 sim episodes (sim labels, no IDM) | Real-looking pixels with exact labels |

Full settings: [RECIPE.md](RECIPE.md).

## Results so far (informal, small samples, 2026-11-03)

| Model | Right-side tries | Turned toward block | Success |
|---|---|---|---|
| S1 (real_left 20) | 2 | 0/2 | 0/2 |
| S3v2 (+ sim MimicGen) | 4 | 4/4 | 2/4 |
| S4v2 (+ Predict, sim-fine-tuned IDM) | 2 | 0/2 | 0/2 |
| S5 (+ Cosmos Transfer of sim) | 4 | 4/4 | 1/4 |

These are informal runs with a person resetting the block between tries. The controlled 20-spot right-side evaluation is **pending**.

## Videos

- [videos/s1_real_run.mp4](videos/s1_real_run.mp4) — Step 1 baseline
- [videos/s3v2_real_run.mp4](videos/s3v2_real_run.mp4) — Step 3 (sim MimicGen)
- [videos/s4v2_real_run.mp4](videos/s4v2_real_run.mp4) — Step 4 (Predict + IDM)
- [videos/s5_real_run.mp4](videos/s5_real_run.mp4) — Step 5 (Cosmos Transfer)

## Figures

- [figures/poster.png](figures/poster.png) — poster
- [figures/](figures/) — MimicGen paths, sim vs. Transfer, Predict post-trained frames, real-robot stills

## Image gallery

Real workspace, NuRec/Isaac Sim views, Cosmos Predict and Transfer success/failure examples and frame grids: see the "What it looks like" sections in [RECIPE.md](RECIPE.md) and [images/](images/).

## Hardware

NVIDIA RTX PRO 6000, NVIDIA RTX 5090, NVIDIA DGX Spark (GB10).

## Acknowledgements and References

This work builds on open models and tools from NVIDIA, Physical Intelligence and Hugging Face.

1. NVIDIA, "World Simulation with Video Foundation Models for Physical AI" (Cosmos-Predict2.5 / Cosmos-Transfer2.5), arXiv:2511.00062, 2025.
2. NVIDIA, "Cosmos World Foundation Model Platform for Physical AI," arXiv:2501.03575, 2025.
3. NVIDIA Isaac Sim (https://developer.nvidia.com/isaac/sim) and Isaac Lab (https://isaac-sim.github.io/IsaacLab/).
4. NVIDIA Omniverse NuRec (https://docs.nvidia.com/nurec); B. Kerbl et al., "3D Gaussian Splatting for Real-Time Radiance Field Rendering," ACM TOG (SIGGRAPH) 2023; Q. Wu et al., "3DGUT: Enabling Distorted Cameras and Secondary Rays in Gaussian Splatting," CVPR 2025.
5. A. Mandlekar et al., "MimicGen: A Data Generation System for Scalable Robot Learning using Human Demonstrations," CoRL 2023, arXiv:2310.17596.
6. K. Black et al. (Physical Intelligence), "π0.5: a Vision-Language-Action Model with Open-World Generalization," CoRL 2025, arXiv:2504.16054; openpi (https://github.com/Physical-Intelligence/openpi).
7. Hugging Face LeRobot (https://github.com/huggingface/lerobot) and the SO-101 arm (https://github.com/TheRobotStudio/SO-ARM100).

## License

Documents, figures and videos: [CC BY 4.0](LICENSE). No scripts are included (MIT would apply to any added later).
