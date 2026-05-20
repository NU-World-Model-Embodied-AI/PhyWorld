<div align="center">

# PhyWorld — Physics-Faithful World Model for Video Generation

[![Project page](https://img.shields.io/badge/Project_page-0d9488?style=for-the-badge&logo=githubpages&logoColor=white)](https://nu-world-model-embodied-ai.github.io/PhyWorld/)
[![Paper](https://img.shields.io/badge/Paper-A42C25?style=for-the-badge&logo=arxiv&logoColor=white)](https://arxiv.org/abs/2605.19242)
[![Model](https://img.shields.io/badge/model-fcd022?style=for-the-badge&logo=huggingface&logoColor=black)](https://huggingface.co/NU-World-Model-Embodied-AI/phyworld)
[![PhyGround](https://img.shields.io/badge/sibling-PhyGround-1f6feb?style=for-the-badge)](https://phyground.github.io/)

</div>

**PhyWorld** is a video-generation world model designed to produce
*temporally coherent* and *physically faithful* scene continuations.
Starting from [Wan2.2-I2V-A14B](https://github.com/Wan-Video/Wan2.2), we
post-train in two stages: (1) flow-matching fine-tuning for video-to-video
continuation, then (2) Direct Preference Optimization (DPO) over physics
preference pairs sourced from the
[PhyGround](https://phyground.github.io/) human-annotation pool.

On the [PhyGround](https://phyground.github.io/) physical-faithfulness
benchmark PhyWorld reaches **3.09** overall vs. **2.99** for the strongest
open baseline; on [VBench](https://vchitect.github.io/VBench-project/) it
reaches **0.769** vs. **0.756** or below for SOTA baselines.

| Companion artifact | Where |
| --- | --- |
| Live project page | [nu-world-model-embodied-ai.github.io/PhyWorld](https://nu-world-model-embodied-ai.github.io/PhyWorld/) |
| Model checkpoint | [🤗 NU-World-Model-Embodied-AI/phyworld](https://huggingface.co/NU-World-Model-Embodied-AI/phyworld) |
| Paper | [arXiv:2605.19242](https://arxiv.org/abs/2605.19242) |
| Sibling benchmark | [PhyGround](https://github.com/NU-World-Model-Embodied-AI/PhyGround) — the benchmark + PhyJudge-9B judge we evaluate against |

---

## What's in this repo

This repository currently hosts the **public project page** for PhyWorld.
The page is served via GitHub Pages from `main` and includes the abstract,
method overview, both leaderboards (VBench and PhyGround), a featured
carousel, and a 4-model head-to-head video grid (PhyWorld vs. Cosmos, LTX,
OmniWeave) across 17 shared prompts.

```
index.html        # the project page
static/css/       # Bulma + PhyWorld custom styles
static/js/        # carousel + FontAwesome loaders
static/videos/    # PhyWorld and baseline mp4 clips
static/images/    # favicons, teaser/framework figures (drop-in)
```

Training, inference, and evaluation code will land in this repo as the
project ships. For now, the model checkpoint is available on
[Hugging Face](https://huggingface.co/NU-World-Model-Embodied-AI/phyworld)
and evaluations can be reproduced with the
[PhyGround](https://github.com/NU-World-Model-Embodied-AI/PhyGround)
pipeline and its released `PhyJudge-9B` judge.

---

## Method

**Stage 1 — Physical Consistency Enhancement.**
Flow-matching fine-tuning on a video-to-video continuation pipeline.
The conditioning video is encoded by the Wan-VAE, a binary mask delimits
preserved vs. synthesized frames, and a CLIP global-context embedding is
injected via decoupled cross-attention. Training data is
[OpenVid-1M](https://github.com/NJU-PCALab/OpenVid-1M), filtered for
inter-frame CLIP cosine similarity and per-clip
[UniMatch](https://github.com/autonomousvision/unimatch) optical-flow
magnitude.

**Stage 2 — Physics Enforcement via DPO.**
A rank-16 LoRA adapter wraps the Wan2.2 denoiser's attention and
feed-forward projections. Preference pairs are derived from the
[PhyGround](https://phyground.github.io/) human-annotation pool:
within-prompt winner/loser pairs with score margin ≥ 1.0, organized into a
1,000-pair class-balanced trainset over seven physical-event classes
(collision/rebound, destruction/deformation, fluids, shadow/reflection,
chain, rolling/sliding, throwing/ballistic). DPO is restricted to the
high-noise window *t* ∈ [901, 999] with β = 100 to suppress reward
hacking. Training: 16 × H100, 2 epochs, AdamW lr 1e-5.

---

## Results

### VBench — generation quality

| Model | Subject cons. | Background cons. | Motion smooth. | Dynamic | Aesthetic | Imaging | Avg. |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **PhyWorld (ours)** | **0.932** | **0.944** | **0.986** | **0.564** | **0.555** | **0.632** | **0.769** |
| Wan2.2-I2V-A14B | 0.912 | 0.928 | 0.977 | 0.554 | 0.543 | 0.622 | 0.756 |
| Cosmos-14B | 0.899 | 0.923 | 0.973 | 0.559 | 0.536 | 0.629 | 0.753 |
| LTX-2.3-22B | 0.894 | 0.918 | 0.982 | 0.549 | 0.532 | 0.626 | 0.751 |
| OmniWeaving | 0.903 | 0.907 | 0.972 | 0.556 | 0.541 | 0.621 | 0.750 |
| Cosmos-2-2B | 0.887 | 0.905 | 0.964 | 0.543 | 0.524 | 0.608 | 0.739 |

### PhyGround — physical faithfulness (PhyJudge-9B, 1–5 scale)

| Model | SA | PTV | Persist. | Solid-Body | Fluid | Optical | Overall |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **PhyWorld (ours)** | **2.78** | **3.07** | **3.23** | **2.84** | **3.04** | **3.57** | **3.09** |
| Wan2.2-I2V-A14B | 2.72 | 2.97 | 3.08 | 2.79 | 3.03 | 3.36 | 2.99 |
| Cosmos-14B | 2.60 | 2.73 | 3.07 | 2.72 | 2.92 | 3.53 | 2.80 |
| OmniWeaving | 2.68 | 2.73 | 2.92 | 2.71 | 2.99 | 3.13 | 2.78 |
| LTX-2.3-22B | 2.63 | 2.79 | 2.91 | 2.55 | 3.02 | 3.21 | 2.72 |
| Wan2.2-TI2V-5B | 2.48 | 2.70 | 2.76 | 2.61 | 3.01 | 3.45 | 2.68 |
| LTX-2-19B | 2.50 | 2.62 | 2.79 | 2.49 | 3.01 | 3.09 | 2.62 |

---

## Local preview

The project page is static — clone and open it directly:

```bash
git clone https://github.com/NU-World-Model-Embodied-AI/PhyWorld.git
cd PhyWorld
python3 -m http.server 8000
# then visit http://localhost:8000
```

Pushing to `main` triggers a GitHub Pages rebuild within ~30–60 seconds.

---

## Citation

```bibtex
@misc{zhao2026phyworld,
  title         = {PhyWorld: Physics-Faithful World Model for Video Generation},
  author        = {Pu Zhao and Juyi Lin and Timothy Rupprecht and Arash Akbari and Chence Yang and Rahul Chowdhury and Elaheh Motamedi and Arman Akbari and Yumei He and Chen Wang and Geng Yuan and Weiwei Chen and Yanzhi Wang},
  year          = {2026},
  eprint        = {2605.19242},
  archivePrefix = {arXiv},
  primaryClass  = {cs.CV},
  url           = {https://arxiv.org/abs/2605.19242}
}
```

---

## Acknowledgements

PhyWorld is post-trained from [Wan2.2-I2V-A14B](https://github.com/Wan-Video/Wan2.2)
and evaluated with the [PhyGround](https://phyground.github.io/) benchmark
and its `PhyJudge-9B` judge. The project page template is adapted from
[OpenVLA](https://openvla.github.io/), which is based on
[Nerfies](https://nerfies.github.io/).
