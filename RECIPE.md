# Recipe

Settings per step, taken from the project notes and the actual training/generation configs. Anything not confirmed in those sources is marked **TBD**.

## Common

| Item | Value |
|---|---|
| Task | Pick up the blue block and put it in the gray bowl |
| Success criterion | Block is inside the bowl when the gripper opens |
| Robot | SO-101 arm (5-DoF + gripper) |
| Camera | One front camera, 640×480 (policy uses front camera only). Frame rate: 15 fps per poster draft; Step 1 notes list 30 fps — **TBD (to reconcile)** |
| Real data | 20 real demonstrations, all with the block on the **left**, LeRobot format (20 episodes / 8,993 frames). Never extended. |
| Right-side spots | 20 fixed spots (R00–R19), mirrored from the left demos and moved until IK passes (position ≤2 mm, rotation ≤10°); 32 mm spacing, within 0.306 m of the base, outside 90 mm of the bowl. Used for Steps 2–5. |

## π0.5 training (policy; same recipe for every step)

| Item | Value |
|---|---|
| Base | `lerobot/pi05_base` (LeRobot) |
| Steps | 10k |
| Batch | 32 |
| LR | 2.5e-5, cosine |
| Precision | bf16 |
| Action chunk | 50 |
| Inputs | front camera only |

Dataset sizes: S1 = 20 ep; S3v2 = 191 ep (real_left 20 + sim 171); S4v2 = 28 ep (real_left 20 + 8 right Predict clips); S5 = 191 ep (real_left 20 + 171 Transfer clips that passed QC).

> "Predict post-training" (training the Cosmos video model) and "π0.5 training" (training the robot policy) are separate jobs.

## Step 1 — baseline

π0.5 on real_left 20 only.

## Step 2 — Cosmos Predict 2.5 + IDM

- **Predict post-training:** Cosmos Predict 2.5 (2B) post-trained on our robot/camera/table video (real_left only). Generation used the iteration-3000 weights. Clip count, resolution and prompt for this post-training: **TBD**.
- **Conditioning:** first front frame of each real left demo with the block removed and re-composited at each of the 20 right spots (with measured shadow).
- **Generation:** 20 right spots × 3 seeds = 60 clips.
- **IDM labeling:** IDM trained on real_left 20 only (video → joint values). IDM architecture/steps: **TBD**.
- **Result:** IDM shoulder_pan never went below −7.5°, while IK needs −18° to −66°; 38/60 clips never reached the block. No π0.5 training for Step 2 on the right-side clips (decision: hold).

## Step 3 — Isaac Sim + NuRec, MimicGen

- **Scene:** NuRec 3DGS (3DGUT) background of the 3D-printed workspace; COLMAP → 3DGUT (MCMC) → USDZ → Isaac Sim 6.0.1. Camera re-calibrated to the real one.
- **Replay gate:** real_left 20 replayed in sim with joint values. Block placed at the center of the open jaw gap ("gapc"): 19/20 lift. The 19 passing demos are the MimicGen seeds.
- **Generation:** 19 seed demos × 20 right spots = 380 attempts. Grasp segment transformed relative to the block; place segment follows the bowl; inside 75 mm of the bowl the gripper stays ≥63 mm above the table.
- **QC (keep rule):** block inside bowl on release, rotation ≤10°, position ≤8 mm, bowl moved <10 mm and tilted <10°. **211/380 kept (55.5%).**
- **Render:** 211 episodes re-rendered from recorded states (front camera); frame 0 dropped and labels shifted by one frame.
- **Labels:** sim ground-truth joints (no IDM).
- **π0.5 training (S3v2):** real_left 20 + sim 171 = 191 ep. Why 171 of the 211 were used: **TBD**.

## Step 4 — Predict re-post-trained on real + sim right, sim-fine-tuned IDM

- **Predict post-training:** Cosmos Predict 2.5 (2B) video2world, from base, on **103 clips** = 23 real_left clips + 80 Step 3 sim right clips (front, 640×480). Resolution 480 (480×640), 93 frames per clip, batch size 1, max 4000 iterations, save every 500; generation used the **iteration-3500** weights (pilot 5/6 pass vs. 3/6 at iter 4000).
- **Generation:** image2world, prompt `"Pick_blue_block_and_put_it_gray_bowl. Simulation render."`, guidance 7, 35 sampling steps, 480×640, 185 output frames, autoregressive in 93-frame chunks with 1-frame overlap. 60 clips generated.
- **IDM fine-tune:** IDM fine-tuned on real_left 20 + sim right episodes (sim joint labels). Exact steps/LR: **TBD**.
- **QC:** automatic QC + visual check → 20 episodes / 7,400 frames (S4 dataset). S4v2 policy used real_left 20 + 8 right clips (28 ep); how the 8 were chosen from the 20: **TBD**.
- **Result:** S4v2 never turned right (0/2). Right clips covered only 6 of 20 spots, pan −28° to −51° vs. −28° to −79° needed.

## Step 5 — Cosmos Transfer 2.5 of sim episodes

- **Input:** Step 3 sim right episodes (front, 10 fps) + edge control.
- **Settings (adopted "v5w"):** Cosmos Transfer 2.5 (2B, distilled), edge control only, strength 1.0; 720p; 4 steps; guidance 3; seed 2025. Prompt describes the real scene with a "plain white desk" / "smooth plain matte white" surface (replacing a wooden-desk wording). Full prompt text: **TBD**.
- **Pilot QC (3 clips):** block IoU 0.85 / 0.89 / 0.80, block center error 1–2 px.
- **Bulk:** 211 transferred → **171 pass QC** / 40 fail. Long clips truncated at 93 frames; labels 1:1 with sim.
- **Labels:** sim ground-truth joints (no IDM).
- **π0.5 training (S5):** real_left 20 + 171 = 191 ep.

## Evaluation

Informal runs only so far (see README). Planned: 20 right spots in fixed order, same success rule. **TBD.**
