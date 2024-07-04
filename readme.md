# PyTorch implementation of "High Quality Image Synthesis with CLIP DDGAN: Text-Guided Denoising Diffusion GANs" #

Generative denoising diffusion models typically assume that the denoising distribution can be modeled by a Gaussian distribution. This assumption holds only for small denoising steps, which in practice translates to thousands of denoising steps in the synthesis process. In our denoising diffusion GANs, we represent the denoising model using multimodal and complex conditional GANs, enabling us to efficiently generate data in as few as two steps.

## Set up datasets ##
We trained on several datasets, including CIFAR10, LSUN Church Outdoor 256 and CelebA HQ 256. 
For large datasets, we store the data in LMDB datasets for I/O efficiency. Check [here](https://github.com/NVlabs/NVAE#set-up-file-paths-and-data) for information regarding dataset preparation.


## Training Denoising Diffusion GANs ##
We use the following commands on each dataset for training denoising diffusion GANs.

#### CIFAR-10 ####
We train Denoising Diffusion GANs on CIFAR-10 using 2 nodes 1 GPU / node.

Node 1
```
python -m torch.distributed.launch --nproc_per_node=1 --nnodes=2 --node_rank=0 --master_addr="127.0.0.1" --master_port=6020  train_ddgan.py --dataset cifar10 --exp ddgan_cifar10_exp1 --num_channels 3 --num_channels_dae 128 --num_timesteps 4 --num_res_blocks 2 --batch_size 64 --num_epoch 1800 --ngf 64 --nz 100 --z_emb_dim 256 --n_mlp 4 --embedding_type positional --r1_gamma 0.02 --lr_d 1.25e-4 --lr_g 1.6e-4 --lazy_reg 15 --num_process_per_node 1 --ch_mult 1 2 2 2 --save_content --num_proc_node 2 --node_rank 0
```

Node 2
```
python -m torch.distributed.launch --nproc_per_node=1 --nnodes=2 --node_rank=1 --master_addr="127.0.0.1" --master_port=6020  train_ddgan.py --dataset cifar10 --exp ddgan_cifar10_exp1 --num_channels 3 --num_channels_dae 128 --num_timesteps 4 --num_res_blocks 2 --batch_size 64 --num_epoch 1800 --ngf 64 --nz 100 --z_emb_dim 256 --n_mlp 4 --embedding_type positional --r1_gamma 0.02 --lr_d 1.25e-4 --lr_g 1.6e-4 --lazy_reg 15 --num_process_per_node 1 --ch_mult 1 2 2 2 --save_content --num_proc_node 2 --node_rank 1
```

We train Denoising Diffusion GANs on CIFAR-10 using 1 80-GB H100 GPU. 
```
python3 train_ddgan.py --dataset cifar10 --exp ddgan_cifar10_exp1 --num_channels 3 --num_channels_dae 128 --num_timesteps 4 \
--num_res_blocks 2 --batch_size 64 --num_epoch 1800 --ngf 64 --nz 100 --z_emb_dim 256 --n_mlp 4 --embedding_type positional \
--use_ema --ema_decay 0.9999 --r1_gamma 0.02 --lr_d 1.25e-4 --lr_g 1.6e-4 --lazy_reg 15 --num_process_per_node 1 \
--ch_mult 1 2 2 2 --save_content
```

#### LSUN Church Outdoor 256 ####

We train Denoising Diffusion GANs on LSUN Church Outdoor 256 using 1 80-GB H100 GPU. 
```
python3 train_ddgan.py --dataset lsun --image_size 256 --exp ddgan_lsun_exp1 --num_channels 3 --num_channels_dae 64 --ch_mult 1 1 2 2 4 4 --num_timesteps 4 \
--num_res_blocks 2 --batch_size 8 --num_epoch 500 --ngf 64 --embedding_type positional --use_ema --ema_decay 0.999 --r1_gamma 1. \
--z_emb_dim 256 --lr_d 1e-4 --lr_g 1.6e-4 --lazy_reg 10 --num_process_per_node 1 --save_content
```

#### CelebA HQ 256 ####

We train Denoising Diffusion GANs on CelebA HQ 256 using 1 80-GB H100 GPUs. 
```
python3 train_ddgan.py --dataset celeba_256 --image_size 256 --exp ddgan_celebahq_exp1 --num_channels 3 --num_channels_dae 64 --ch_mult 1 1 2 2 4 4 --num_timesteps 2 \
--num_res_blocks 2 --batch_size 4 --num_epoch 800 --ngf 64 --embedding_type positional --use_ema --r1_gamma 2. \
--z_emb_dim 256 --lr_d 1e-4 --lr_g 2e-4 --lazy_reg 10  --num_process_per_node 1 --save_content
```

## Pretrained Checkpoints ##
We have released pretrained checkpoints on CIFAR-10 and CelebA HQ 256 at this 
[Google drive directory](https://drive.google.com/drive/folders/1UkzsI0SwBRstMYysRdR76C1XdSv5rQNz?usp=sharing).
Simply download the `saved_info` directory to the code directory. Use `--epoch_id 1200` for CIFAR-10 and `--epoch_id 550`
for CelebA HQ 256 in the commands below.

## Evaluation ##
After training, samples can be generated by calling ```test_ddgan.py```. We evaluate the models with single V100 GPU.
Below, we use `--epoch_id` to specify the checkpoint saved at a particular epoch.
Specifically, for models trained by above commands, the scripts for generating samples on CIFAR-10 is
```
python3 test_ddgan.py --dataset cifar10 --exp ddgan_cifar10_exp1 --num_channels 3 --num_channels_dae 128 --num_timesteps 4 \
--num_res_blocks 2 --nz 100 --z_emb_dim 256 --n_mlp 4 --ch_mult 1 2 2 2 --epoch_id $EPOCH
```
The scripts for generating samples on CelebA HQ is 
```
python3 test_ddgan.py --dataset celeba_256 --image_size 256 --exp ddgan_celebahq_exp1 --num_channels 3 --num_channels_dae 64 \
--ch_mult 1 1 2 2 4 4 --num_timesteps 2 --num_res_blocks 2  --epoch_id $EPOCH
```
The scripts for generating samples on LSUN Church Outdoor is 
```
python3 test_ddgan.py --dataset lsun --image_size 256 --exp ddgan_lsun_exp1 --num_channels 3 --num_channels_dae 64 \
--ch_mult 1 1 2 2 4 4  --num_timesteps 4 --num_res_blocks 2  --epoch_id $EPOCH
```

We use the [PyTorch](https://github.com/mseitzer/pytorch-fid) implementation to compute the FID scores, and in particular, codes for computing the FID are adapted from [FastDPM](https://github.com/FengNiMa/FastDPM_pytorch).

To compute FID, run the same scripts above for sampling, with additional arguments ```--compute_fid``` and ```--real_img_dir /path/to/real/images```.

For Inception Score, save samples in a single numpy array with pixel values in range [0, 255] and simply run 
```
python ./pytorch_fid/inception_score.py --sample_dir /path/to/sampled_images
```
where the code for computing Inception Score is adapted from [here](https://github.com/tsc2017/Inception-Score).

For Improved Precision and Recall, follow the instruction [here](https://github.com/kynkaat/improved-precision-and-recall-metric).


## License ##
Please check the LICENSE file. Denoising diffusion GAN may be used non-commercially, meaning for research or 
evaluation purposes only. For business inquiries, please contact 
[researchinquiries@nvidia.com](mailto:researchinquiries@nvidia.com).

## Bibtex ##
Cite our paper using the following bibtex item:

```
@inproceedings{
xiao2022tackling,
title={Tackling the Generative Learning Trilemma with Denoising Diffusion GANs},
author={Zhisheng Xiao and Karsten Kreis and Arash Vahdat},
booktitle={International Conference on Learning Representations},
year={2022}
}
```

## Contributors ##
Denoising Diffusion GAN was built primarily by [Zhisheng Xiao](https://xavierxiao.github.io/) during a summer 
internship at NVIDIA research.
