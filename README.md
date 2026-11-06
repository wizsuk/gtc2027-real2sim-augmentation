# From 20 Real Demos to Unseen Positions: A 3DGS NuRec Digital Twin Powering Cosmos Predict and Transfer for VLA Data Augmentation

**Logan (Jae Suk) Lee, Megazone** · NVIDIA GTC 2027 poster companion repository

## Abstract

Vision-language-action (VLA) policies fine-tuned on a few real demonstrations succeed near the object positions they saw and fail elsewhere. We fix the real data at 20 teleoperated demos, all on the left side of the table, and ask whether augmentation alone can teach an SO-101 arm running π0.5 to pick and place at 20 unseen right-side positions. The workspace is 3D-printed and mirrored by a NuRec 3DGS digital twin in Isaac Sim. Five steps share the same 20 demos: (1) baseline; (2) Cosmos Predict 2.5 clips labeled by an inverse dynamics model (IDM); (3) MimicGen right-side episodes in the twin with exact sim labels; (4) Predict post-trained on twin clips, labeled by a sim-fine-tuned IDM; (5) Cosmos Transfer 2.5 turns twin episodes into real-looking video while keeping sim labels.

On the right, the Step 2 IDM `shoulder_pan` stays within −7.5° while inverse kinematics (IK) needs −18° to −66°; 19/20 real demos pass a replay gate in the twin and MimicGen keeps 211/380 right-side episodes. On the same 20 right spots (R00–R19), the left-only baseline S1 shows no right-side motion and 0/20 success; real+predict100 (S2) matches that (0/20 motion, 0/20 success); real+sim (S3v2) and real+Transfer (S5) reach right-side motion 19/20 and success 12/20 and 14/20; real+gen10 (S4v3r10) also scores 19/20 on right-side motion but 0/20 success.

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

**Pipeline generation vs evaluated policy.** The Step 2 and Step 4 generation runs were each described as 20 spots × 3 seeds = 60 clips. The official right20 table below evaluates specific π0.5 mixes: **S2 = real+predict100**, **S4 = S4v3r10 (real+gen10)**. Those evaluated mix sizes are not the same as the 60-clip generation counts.

Abbreviations: S1 = Step 1 · S2 = Step 2 · S3v2 = Step 3, generation version 2 · S4v3r10 = Step 4, generation version 3, 10 generated clips · S5 = Step 5.

## Official results (formal right20, 2026-11-06)

Locked eval: the same 20 right spots (R00–R19) for every step. Two axes: **place success** (block inside the bowl when the gripper opens), and **right-side motion** (binary: any rightward `shoulder_pan`).

| Step | Training data (20-spot right eval) | Success (place) | Right-side motion |
|---|---|---|---|
| S1 | real left 20 only | 0/20 | 0/20 |
| S2 | real + predict100 | 0/20 | 0/20 |
| S3v2 | real + sim 171 | 12/20 | 19/20 |
| S4v3r10 | real + gen10 | 0/20 | 19/20 |
| S5 | real + Transfer 171 | 14/20 | 19/20 |

- Why: on the same 20 right spots, which augmentation yields right-side pick-and-place, given that left-only S1 never enters the right workspace (0/20 motion, 0/20 place)?
- Real+sim (S3v2) and real+Transfer (S5) acquire right-side motion (19/20) and complete places (12/20 and 14/20); Transfer is slightly stronger.
- Real+gen10 (S4v3r10) shares the 19/20 motion flag but its turn is too small to reach the block, so 0/20 places; equal motion scores ≠ equal competence.
- Left-only S1 and real+predict100 (S2) never turn right (0/20 motion, 0/20 places).
- Two different generated-data failures: Predict (S2) never rotated right (0/20 motion, 0/20 places); gen10 (S4v3r10) rotated (19/20) but did not reach the block (0/20 places). S3v2 (sim) 12/20 and S5 (Transfer) 14/20 are what actually placed.
- **Takeaway:** Generated video can look plausible yet fail when joints or reach are wrong; accurate sim trajectories and Transfer recover the right side.

Table notes: S4v3r10 often started a rightward motion (19/20) but the turn was too small to reach the block, so pick-and-place success stayed 0/20. Limitation: right-side motion is binary (any rightward `shoulder_pan` counts), so it overstates S4v3r10's spatial competence.

**Future work (from the poster):** Generated clips (S4v3r10) turn right (19/20) but undershoot (0/20 places): improve their turn size and reach, or rely on sim (S3v2) and Transfer (S5) for right-side coverage.

Earlier informal real-robot tries (2026-11-03, small samples, person resetting the block) are **not** this eval and are not mixed into the table. Those tries used an earlier Step 4 policy (S4v2) and did not include a formal S2 row.

## Videos

The files below are **earlier informal real-robot runs**, not recordings of the formal 20-spot (R00–R19) eval.

- [videos/s1_real_run.mp4](videos/s1_real_run.mp4) — Step 1 baseline (informal run)
- [videos/s3v2_real_run.mp4](videos/s3v2_real_run.mp4) — Step 3 (sim MimicGen; informal run)
- [videos/s4v2_real_run.mp4](videos/s4v2_real_run.mp4) — earlier Step 4 version (S4v2, not the evaluated S4v3r10; informal run)
- [videos/s5_real_run.mp4](videos/s5_real_run.mp4) — Step 5 (Cosmos Transfer; informal run)

## Figures

- [figures/poster.png](figures/poster.png) — poster export in this repo (older layout; the official right20 numbers are in the Results section above)
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
