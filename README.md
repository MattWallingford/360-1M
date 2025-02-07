
## [Website](https://mattwallingford.github.io/ODIN/) | [HuggingFace](https://huggingface.co/datasets/mwallingford/360-1M/tree/main) | [Paper](https://openreview.net/pdf?id=otxOtsWCMb) | 

**360-1M** is a large-scale 360° video dataset consisting of over 1 million videos for training video and 3D foundation models. This repository contains the following:
1. Links to the videos URLs for download from YouTube. We also provide a smaller 24k filtered subset for experimentation.
2. Metadata for each video including category, resolution, and views. 
3. Code for downloading the videos locally and to Google Cloud Platform (recommended).
4. Code for filtering, processing, and obtaining camera pose for the videos.
5. Code for training the novel view synthesis model, [ODIN](https://openreview.net/pdf?id=otxOtsWCMb).
   
| **Reference Image**<br><img src="https://raw.githubusercontent.com/MattWallingford/ODIN/main/nyc-256x256.png" width="256" alt="NYC Reference" /><br>**Generated Scene Trajectory**<br><img src="https://raw.githubusercontent.com/MattWallingford/ODIN/main/nyc4.gif" width="256" alt="NYC Demo" /> | **Reference Image**<br><img src="https://raw.githubusercontent.com/MattWallingford/ODIN/main/livingroom-256x256.jpg" width="256" alt="Living Room Reference" /><br>**Generated Scene Trajectory**<br><img src="https://raw.githubusercontent.com/MattWallingford/ODIN/main/living_room_zoom.gif" width="256" alt="Living Room Demo" /> | **Reference Image**<br><img src="https://raw.githubusercontent.com/MattWallingford/ODIN/main/picnic-256x256.png" width="256" alt="Picnic Reference" /><br>**Generated Scene Trajectory**<br><img src="https://raw.githubusercontent.com/MattWallingford/ODIN/main/picnic2.gif" width="256" alt="Picnic Demo" /> |
| --- | --- | --- |

## Downloading Videos
Metadata and video URLs can be downloaded from here: [Metadata with Video URLs](https://huggingface.co/datasets/mwallingford/360-1M/tree/main) .
The filtered subset which is around 5 TB in size can be found here: [Filtered Subset](https://huggingface.co/datasets/mwallingford/360-1M/blob/main/Filtered_24k.parquet)

To download the videos we recommend using the yt-dlp package. To run our download scripts you'll also need pandas and pyarrow to parse the metadata parquet:
```bash
#Install packages for downloading videos
pip install yt-dlp
pip install pandas
pip install pyarrow
```

The videos can be downloaded using the provided script:
```bash
python DownloadVideos/download_local.py --in_path 360-1M.parquet --out_dir /path/to/videos
```
or to download the high quality subset:
```bash
python DownloadVideos/download_local.py --in_path Filtered_24k.parquet --out_dir /path/to/videos
```

The total size of all videos at max resolution is about 200 TB. We recommend downloading to a cloud platform due to bandwidth limitations and provide a script for use with GCP.

```bash
python DownloadVideos/Download_GCP.py --path 360-1M.parquet
```

<p align="center">
  <img src="https://raw.githubusercontent.com/MattWallingford/ODIN/main/sample_top.gif" width="410" alt="Sample 1" />
  <img src="https://raw.githubusercontent.com/MattWallingford/ODIN/main/sample_bot.gif" width="410" alt="Sample 2" />
</p>


---

## Installation Guide for Video Processing And Training

### Environment Setup
1. Create a new Conda environment:
   ```
   conda create -n ODIN python=3.9
   conda activate ODIN
```
2. Clone the repository:

```bash
cd ODIN
pip install -r requirements.txt
```
3. Install additional dependencies:

```bash
git clone https://github.com/CompVis/taming-transformers.git
pip install -e taming-transformers/
git clone https://github.com/openai/CLIP.git
pip install -e CLIP/
```

4. Clone the MAST3R repository:
```bash
git clone --recursive https://github.com/naver/mast3r
cd mast3r
```
5. Install MAST3R dependencies:

```bash
pip install -r requirements.txt
pip install -r dust3r/requirements.txt
For detailed installation instructions, visit the MAST3R repository.
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
Download the image-conditioned Stable Diffusion checkpoint released by Lambda Labs:

```bash
wget https://cv.cs.columbia.edu/zero123/assets/sd-image-conditioned-v2.ckpt
```
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
### Coming Soon
- Model weights with inference and fine-tuning code. 
