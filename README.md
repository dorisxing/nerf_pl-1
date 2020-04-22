# nerf_pl
Unofficial implementation of [NeRF](https://arxiv.org/pdf/2003.08934.pdf) (Neural Radiance Fields) using [pytorch-lightning](https://github.com/PyTorchLightning/pytorch-lightning).

Official implementation: [nerf](https://github.com/bmild/nerf)

Reference pytorch implementation: [nerf-pytorch](https://github.com/yenchenlin/nerf-pytorch)

## Features

* Multi-gpu training: Training on 8 GPUs finishes within 1 hour for the synthetic dataset!

# Installation

## Hardware

* OS: Ubuntu 18.04
* NVIDIA GPU with **CUDA>=10.1** (tested with 1 RTX2080Ti)

## Software

* Clone this repo by `git clone --recursive https://github.com/kwea123/nerf_pl`
* Python 3.8 (installation via [anaconda](https://www.anaconda.com/distribution/) is recommended, use `conda create -n nerf_pl python=3.8` to create a conda environment and activate it by `conda activate nerf_pl`)
* Python libraries
    * Install core requirements by `pip install -r requirements.txt`
    * Install `torchsearchsorted` by `cd torchsearchsorted` then `pip install .`
    
# Training

Please see each subsection for training on different datasets. Available training datasets:

* [Blender](#blender) (Realistic Synthetic 360)

## Blender

### Data download

Download `nerf_synthetic.zip` from [here](https://drive.google.com/drive/folders/128yBriW1IG_3NJ5Rp7APSTZsJqdJdfc1)

### Training model

Run (example)
```
python train.py \
   --dataset_name blender \
   --root_dir $BLENDER_DIR \
   --N_importance 64 --img_wh 400 400 --noise_std 0 \
   --num_epochs 16 --batch_size 1024 \
   --optimizer adam --lr 5e-4 \
   --lr_scheduler steplr --decay_step 2 4 8 --decay_gamma 0.5 \
   --exp_name exp
```

These parameters are chosen to best mimic the training settings in the original repo.
See [opt.py](opt.py) for all configurations.

### Pretrained model and log
Download the pretrained model and training log in [release](https://github.com/kwea123/nerf_pl/releases).

### Comparison with other repos

|           | GPU mem in GB <br> (train) | Speed (1 step) |
| :---:     |  :---:     | :---:   | 
| [Original](https://github.com/bmild/nerf)  |  8.5 | 0.177s |
| [Ref pytorch](https://github.com/yenchenlin/nerf-pytorch)  |  6.0 | 0.147s |
| This repo | 3.2 | 0.12s |

The speed is measure on 1 RTX2080Ti. Detailed profile can be found in [release](https://github.com/kwea123/nerf_pl/releases).
Training memory is largely reduced, since the original repo loads the whole data to GPU at the beginning, while we only pass batches to GPU every step.

# Testing

See [test.ipynb](test.ipynb) for a simple view synthesis and depth prediction on 1 image.

Use [eval.py](eval.py) to create the whole sequence of moving views.
E.g.
```
python eval.py \
   --root_dir $BLENDER \
   --dataset_name blender --scene_name lego \
   --img_wh 400 400 --N_importance 64 --ckpt_path $CKPT_PATH
```
It will create folder `results/{dataset_name}/{scene_name}` and run inference on all test data, finally create a gif out of them.

Example of lego scene using pretrained model, shown at 1/2 scale: (PSNR=31.39, paper=32.54)

![lego](https://user-images.githubusercontent.com/11364490/79932648-f8a1e680-8488-11ea-98fe-c11ec22fc8a1.gif)

Example of fern scene using pretrained model, shown at 1/2 scale:

![fern](https://user-images.githubusercontent.com/11364490/79932650-f9d31380-8488-11ea-8dad-b70a6a3daa6e.gif)

# Notes on differences with the original repo

*  The learning rate decay in the original repo is **by step**, which means it decreases every step, here I use learning rate decay **by epoch**, which means it changes only at the end of 1 epoch.
*  The validation image for LLFF dataset is chosen as the last image here, whereas the original repo chooses every 8th image.
*  The rendering spiral path is slightly different from the original repo (I use approximate values to simplify the code).

# TODO
- [ ] Test multigpu for llff data with 1 val image only across 8 gpus..