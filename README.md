# NEKO 🐈
[![Check out our Discord Here](https://dcbadge.vercel.app/api/server/a8uDbxzEbM)](https://discord.gg/a8uDbxzEbM)

## Status
This implementation is currently in progress.

## Vision

The NEKO Project is an open source effort to build a "generalist" model of greater scale and capability as that reported in DeepMind’s 2022 Paper, [A Generalist Agent](https://www.deepmind.com/publications/a-generalist-agent). This constitutes the first major step in a longer goal of building multimodal, multiobjective models that work well across a variety of domains.

Ultimately, collective high performance across many objectives in varied modalities constitutes a SoTA direction. We hope that in building such models and making them open source, we can move humanity’s collective understanding and capability in building complex AI systems forward.

Practically, building these systems provides us deeper understanding into the utility and mechanics of modern deep learning approaches in building better representational capability and scaling to new problem domains. Put another way, this research direction serves as an excellent foundation to test other, new kinds of general AI approaches to build next generation learning systems. 

## [NEKO Project Roadmap](https://docs.google.com/document/d/e/2PACX-1vQ2JVJvSiYmwjDFnppj0_38NCUEdLG8pAdj0Q2tSy1yy4wwQxJOAAzNFwz2Is4TONhgUVnvJzuu5o85/pub)

## [NEKO Open Issues and Tasks](https://github.com/orgs/ManifoldRG/projects/12)

## [Contributing Guide](https://github.com/ManifoldRG/NEKO/blob/master/CONTRIBUTING.md)

## [NEKO Implementation](https://github.com/ManifoldRG/NEKO/tree/master/gato)

If you use this project, we'd love for you to refer back to Manifold in your code!

# Setup

## Manual Setup
```bash
conda env create -f env.yml 
```
Our code works in colab, a minimal example can be found [here](https://colab.research.google.com/drive/1MEusPZYV4eFsljb88JEyivqP-ogNi0Ni?usp=sharing).

### Minari
We rely on [Minari](https://minari.farama.org/) to provide a standard for datasets. However, we are currently testing with MuJoCo locomotion and Atari tasks, which are not included in Minari by default. These datasets are generated by: https://github.com/daniellawson9999/data-tests/
and can be downloaded by: 

```bash
cd ..
python ./gato/data/download_custom_datasets.py
```

(this will only download MuJoCo datasets, but refer to file for downloading others like [Breakout](https://drive.google.com/drive/folders/1Elos7A-NbpDzr5bPpPmoM-_2qY_68KFi?usp=drive_link))

## Docker

```bash
docker build -t gato-control -f ./docker/Dockerfile .
docker run -it --mount "type=bind,source=$(pwd),target=/app/gato-control" --entrypoint /bin/bash --gpus=all gato-control

```




# Training
Below are some example training commands. 

Training on 3 MuJoCo locomotion tasks:
```bash
python train.py --embed_dim=768 --layers=6 --heads=24 --training_steps=100000 --log_eval_freq=10000 --warmup_steps=10000 --batch_size=32 -k=240 --eval_episodes=10 --activation_fn=gelu --save_model --save_mode=checkpoint --control_datasets d4rl_halfcheetah-expert-v2 d4rl_hopper-expert-v2 d4rl_walker2d-expert-v2 -w
```
example run log: https://wandb.ai/daniellawson9999/gato-control/runs/j9u26q9p/overview?workspace=user-daniellawson9999


Atari (in progress):
```bash
python train.py --embed_dim=128 --layers=3 --heads=1 --training_steps=10000 --log_eval_freq=1 --warmup_steps=100 --batch_size=4 -k=512 --eval_episodes=1 --device=cuda --control_datasets Breakout-top1-s1-v0
```
example run log: https://wandb.ai/daniellawson9999/gato-control/runs/qagorj06/workspace?workspace=user-daniellawson9999\

In general, control_datasets can contain lists of any strings in download_custom_datasets.py or a dataset in https://minari.farama.org/ with Box or Discrete observation or action spaces, although not all default Minari environments have not been tested yet. 
can mix in a single run, e.g:
`--control_datasets Breakout-top1-s1-v0 hammer-expert-v0`

Image-Caption (in progress):
```bash
python train.py --use_wandb --embed_dim=768 --layers=6 --heads=24 --training_steps=1000 --log_eval_freq=10 --warmup_steps=10 --batch_size=4 -k=240 --eval_episodes=10 --sequence_length=1024 --activation_fn=gelu --save_model --caption_prop=1.0 --caption_dataset="/<your data path>/Caption_Data" --caption_train_data=train --caption_test_data=test
```
VQA (in progress):
```bash
python train.py --embed_dim=768 --layers=6 --heads=24 --training_steps=1000 --log_eval_freq=10 --warmup_steps=10 --batch_size=4 -k=240 --eval_episodes=10 --sequence_length=1024 --activation_fn=gelu --save_model --vqa_prop=1.0 --vqa_dataset='/<your data path>/VQA_Data/' --vqa_train_data=train2014 --vqa_test_data=val2014 --train_img_name_prefix=COCO_train2014_ --train_img_file_name_len=27 --test_img_name_prefix=COCO_val2014_ --test_img_file_name_len=25
```
The `--caption_prop` and `--vqa_prop` are the proportions of samples of data from each of the two tasks (cation and VQA) that is fed into the model training. Such proportions from all tasks should sum up to 1.0 if multiple tasks are trained simultaneously, which should be the case for normal training. The above-mentioned examples single out each task, that is for demo and test purpose.

## Atari Datasets
All Atari datasets now follow the convention of `{Name}-top1-s1-v0`, e.g. `Breakout-top1-s1-v0`. Previously, we old runs may have `Breakout-expert_s0-v0` which is depreciated. These datasets are top-1% dqn-replay converted to Minari, refer [here](https://github.com/daniellawson9999/data-tests#port) for more details.

You will be able to train on any env in https://github.com/ManifoldRG/gato-control/blob/master/gato/envs/atari.py. To train on all 40 training games, pass `--datasets TOP1_ATARI_TRAIN` or `--datasets TOP1_ATARI_TEST` for the 5 testing environmments.

Currently, only Breakout is provided here for testing but others will be available shortly. 

## Image-Caption Datasets
So far we have identified two datasets, and the number can increase in the future. For both datasets, we have used a tool "img2dataset" to download the data into webdataset format -
- Data are downloaded into .tar files, each .tar file contains multiple bundles
- Each bundle contain one image in jpg format resized to the designated size (256*256 by default), one txt file that is the caption for the 
- image and one .json file that is the metadata for this bundle (the URL of the image, the caption, the image size, etc.)

The two datasets:
- https://github.com/rom1504/img2dataset/blob/main/dataset_examples/cc3m.md
Follow the instruction there to download the data, and the metadata file name mentioned on the page "cc3m.tsv" might be different from the most up to date source URL: https://ai.google.com/research/ConceptualCaptions/download
    
- https://github.com/rom1504/img2dataset/blob/main/dataset_examples/mscoco.md
Simply follow insturctions to download the dataset

## Evaluation
```bash
python eval.py --model_path={model_path} --eval_episodes={n_episodes}
```

## Examples

```python
import torch
from gato.policy.gato_policy import GatoPolicy

model = GatoPolicy(
        device='cpu',
        embed_dim=128,
        layers=2,
        heads=4,
        dropout=0.1,
)

# This computes logits and (cross-entropy) loss over a batch of size three, where each diciontary is an episode in the batch

logits, loss = model([
    {
        'images': torch.randn(20, 3, 80, 64),
        'discrete_actions': torch.randint(0, 55, (20, 1)),
    },
    {
        'continuous_obs': torch.randn(15, 8),
        'continuous_actions': torch.randn(15, 4),
    },
    {
        'images': torch.randn(100, 3, 224, 224),
        'continuous_actions': torch.randn(100, 11),
    }
], compute_loss=True)

```

# Pretrained models

We provide some pretrained models, which are not geared for high-performance or reliable external use, but to aid in our open-source development. These can be found for [3 MuJoCo tasks](https://drive.google.com/drive/folders/1hws2ip5SKU6KLOVfRU_N8GNPTHPkxVLP?usp=sharing) and [Breakout](https://drive.google.com/drive/folders/1qzUaY6Qh_MmS8o0EqDw3OL55H3Yn6yxe?usp=sharing). Other models may be added [here](https://drive.google.com/drive/folders/1xVo462ZAs54DxsYTsp7NxmCGvrGVBMFj?usp=sharing) where the directory contains checkpoints, arguments, and link to WandB run in each info.txt.

# Future
Our implementation does not directly mirror Gato. Features left out or planned to be added in the future can be found in [todo.md](https://github.com/ManifoldRG/gato-control/blob/master/misc/todo.md). We are working on adding modular tasks, check out the Issues tab.

# Credits

This implementation is influenced and uses components from:
- https://github.com/OrigamiDream/gato/tree/main
- https://github.com/LAS1520/Gato-A-Generalist-Agent
- [VIMA: General Robot Manipulation with Multimodal Prompts](https://github.com/vimalabs/VIMA)
- [Decision Transformer: Reinforcement Learning via Sequence Modeling](https://github.com/kzl/decision-transformer) 
- [Can Wikipedia Help Offline Reinforcement Learning?](https://github.com/machelreid/can-wikipedia-help-offline-rl)  
