# 360-1M
# ODIN Installation and Usage Guide

## Overview
This repository provides tools for training the ODIN model and processing video data for various tasks. The metadata with URLs for all videos can be found at the following link:

[Metadata with Video URLs](https://huggingface.co/datasets/mwallingford/360-1M/tree/main) 

---

## Installation Guide

### Environment Setup
a. Create a new Conda environment:
   ```
   conda create -n ODIN python=3.9
   conda activate ODIN
```
b. Clone the repository:

```bash
cd ODIN
pip install -r requirements.txt
```
c. Install additional dependencies:

```bash
git clone https://github.com/CompVis/taming-transformers.git
pip install -e taming-transformers/
```
```bash
git clone https://github.com/openai/CLIP.git
pip install -e CLIP/
Processing Videos
Install MAST3R
```
a. Clone the MAST3R repository:
```bash
git clone --recursive https://github.com/naver/mast3r
cd mast3r
```
b. Install MAST3R dependencies:

```bash
pip install -r requirements.txt
pip install -r dust3r/requirements.txt
For detailed installation instructions, visit the MAST3R repository.
```
## Data Download and Preprocessing
Downloading Videos
The videos can be downloaded using the provided script:

```bash
python downloadvideos.py --path 360-1M.parquet
```

### Extracting Frames
To extract frames from videos, use the video_to_frames.py script:


```bash
python video_to_frames.py --path /path/to/videos --out /path/to/frames
```

Extracting Pairwise Poses
Once frames are extracted, pairwise poses can be calculated using:

```bash
python extract_poses.py --path /path/to/frames
```

## Training
Download Stable Diffusion Checkpoint
Download the image-conditioned Stable Diffusion checkpoint released by Lambda Labs:


wget https://cv.cs.columbia.edu/zero123/assets/sd-image-conditioned-v2.ckpt
Start Training
Run the training script:

```bash
python main.py \
    -t \
    --base configs/sd-ODIN-finetune-c_concat-256.yaml \
    --gpus 0,1,2,3,4,5,6,7 \
    --scale_lr False \
    --num_nodes 1 \
    --check_val_every_n_epoch 1 \
    --finetune_from sd-image-conditioned-v2.ckpt
```