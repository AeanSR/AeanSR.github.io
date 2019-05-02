---
layout: post
title: Nelf Name Generator V2
---

## Introduction

I have made a name generator based on LSTM-LM in [the 2016 post](/2016/07/27/nelf-name-generator/). The quality is ok for novel-writing if you would like to spend some time to look for a satisfying name in the generated list.

But considers the situation that you are designing a game to give unique names to every random NPCs, the quality of *Nelf Name Generator V1* cannot meet the requirement without supervision.

I am looking for new technologies to refine the generator. The *Nelf Name Generator V2* is based on *WGAN-GP*, referenced from [Improved Training of Wasserstein GANs](https://arxiv.org/abs/1704.00028).

## Wasserstein GAN

*GAN* is for *Generative Adversarial Network*. It constitutes a *Discriminator (`D`)* and a *Generator (`G`)*. To train the network, first use `G` to generate samples `y'`, and mix `y'` with real samples `y`. Let `D` to discriminate samples if it belongs to `y'` (generated samples) or `y` (real samples), and train it with errors. Derivate `D` to `y'` as the gradient of `y'`, which indicates the direction to let `y'` be "more realistic". Apply `∂D/∂y'` to `G` to train with back propagation, so that the generated samples move along with the "realistic" direction. After several epochs, the network arrives at an equilibrium: `D` cannot discriminate generated samples and real samples, `∂D/∂y'` convergences to zero, and `G` no longer changes. At this moment, the generated samples have the same probability distribution as real samples.

*Wasserstein GAN* is proposed in the breaking paper [Wasserstein GAN](https://arxiv.org/abs/1701.07875) by Martin Arjovsky in the early-2017. There is a [detailed interpretation in Zhihu](https://zhuanlan.zhihu.com/p/25071913). The paper discussed the reason why GAN is hard to train. The mechanism behind GANs is to minimize the *J-S distance* or *K-L distance* between the distribution of samples. However, the distributions are low-dimensional manifolds in high-dimensional space, where the overlapped volume is often zero. The *J-S distance* and *K-L distance* lose their functions in this circumstance. The improved method is to use *Wasserstein distance* instead, which could give directions even when the overlapped volume of distributions is zero.

The following paper of *WGAN-GP* is cooperated with Martin Arjovsky, published two months later. It revised one of the improving method (weight clipping) to a more reasonable one (gradient penalty).

Thanks to the excellent traits of *WGAN*, this is the very first time that people achieve text generating with GANs. In other GANs, the discriminator will discriminate real samples directly according to the value of one-hot encoding, so that it cannot provide any effective directions to the generator. *WGAN* will keep pulling the distance of two probability distributions closer even when it can discriminate with one-hot encoding already.

## Implement

I did not write codes. Just pulled the code from the original author of the paper and modified a little bit. The code is [here](https://github.com/AeanSR/improved_wgan_training). Since the corpus is very simple I decreased the space of parameters in the model to prevent over-fitting. The original model used 5 residual blocks with 512 dimensions of features, and I decreased them to 2 residual blocks and 32 dimensions.

The training data is extracted from a private server database of patch 7.35 (TrinityCore), combined with a database client from patch 7.25. I queried names of all female night elf NPCs and removed other parts except first names manually. There are 661 names in total.

## Result

**Left**: Results from *LSTM-LM (Nelf Name Generator V1)*. **Middle**: Random names in the client, prepared by Blizzard. **Right**: Results from *WGAN-GP (Nelf Name Generator V2)*.

<div><button class="collapsible">Collapsed</button><div class="collapsible-content">
  
|**LSTM-LM**|**Blizzard**|**WGAN-GP**|
|---|---|---|
|Liir|Aqulais|Alysna|
|Kyula|Selwynn|Myshaina|
|Aarael|Alayia|Saeurdore|
|Censa'oh|Alasia|Lilly|
|Salciea|Elybrook|Ishawnn|
|Mleharite|Alaria|Eloria|
|Aarnail|Rochelle|Novo|
|Sltthandris|Ivy|Jalena|
|Lashera|Elessaria|Falandria|
|'yiuamaliana|Mavralais|Jayanna|
|Gorallia|Aria|Tyranna|
|Cieia|Edelinn|Shyela|
|Derelien|Syyia|Asy'ia|
|Ly'ura|Brinna|Lyanis|
|Tira|Elyria|Sniela|
|Kllyoana|Adila|Silra|
|Nvla|Caylbrooke|Aeya|
|Juraia|Dara|Hesteral|
|Flara|Saelda|Ella'dria|
|Titianna|Annalore|Cinls|
|Yeainsiy|Elyda|Laurne|
|Aasephine|Kynlea|Dulvian|
|Ahmnnai|Cybelle|Aelysea|
|Reanl|Arlana|Ranelao|
|Eeyra|Saellea|Roow|
|Dalllyn|Shaulea|Ea'yssia|
|Myirill|Laana|Lunura|
|Lelytha|Saebrooke|Allanya|
|Kylda|Kynreith|Leana|
|Myiuaa|Syreith|Kinda|
|Mini|Lada|Nauianaa|
|Ahynysil|Catalin|Syli'nna|
|Jolania|Mavraena|Chellsane|
|Alilune|Belinna|Arlysea|
|Tynytha|Syda|Illay|
|Clyraste|Alareith|Csana|

</div></div>

A list containing 64,000 generated names (possibly duplicated) is [here](/ext/final_799999.txt). The left column is the generated names, the right column is the evaluation given by discriminator. It may help naming new characters in creation colleagues.
