# DiffSynth-Studio

We adopt the DiffSynth-Studio framework to run inference of our model. 

## Installation

Install from source (recommended):

```
git clone https://github.com/NU-World-Model-Embodied-AI/PhyWorld.git
cd PhyWorld
pip install -e .
```

For more installation methods and instructions for non-NVIDIA GPUs, please refer to the [Installation Guide](/docs/en/Pipeline_Usage/Setup.md).

## Inference

First download [our model](https://huggingface.co/NU-World-Model-Embodied-AI/phyworld) to local directory.

Then refer to scripts/inference_V2V.py to run inference of our model. Remember to update the model path. 