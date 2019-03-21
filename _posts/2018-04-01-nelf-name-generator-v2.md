---
layout: post
title: Nelf Name Generator V2
---

I have made a name generator based on LSTM-LM in [2016 post](2016/07/27/nelf-name/generator/). The quality is ok for novel-writing, if you would like to spend some time to look for a satisfying name in the generated list.

But considers the situation that you are designing a game to give unique names to every random NPCs, the quality of *Nelf Name Generator V1* can not meet the requirement without supervision.

I am looking for new technologies to refine the generator. The *Nelf Name Generator V2* is based on *WGAN-GP*, referenced from [Improved Training of Wasserstein GANs](https://arxiv.org/abs/1704.00028).

*GAN* is for *Generative Adversarial Network*. It constitutes a *Discriminator (`D`)* and a *Generator (`G`)*. To train the network, first use `G` to generate samples `y'`, and mix `y'` with real samples `y`. Let `D` to discriminate samples if it belongs to `y'` (generated samples) or `y` (real samples) and train it with errors. Deriviate `D` to `y'` as the gradient of `y'`, which indicates the direction to let `y'` be "more realistic". Apply `∂D/∂y'to `G` to train with back propagation, so that the generated samples move along with the "realistic" direction. After several epoches, the network arrives at a equilibrium: `D` cannot discriminate generated samples and real samples, `∂D/∂` convergences to zero, and `G` no longer changes. At this moment, the generated samples have the same probability distribution with real samples.

*Wasserstein GAN* is proposed in the breaking paper [Wasserstein GAN](https://arxiv.org/abs/1701.07875) by Martin Arjovsky in the early-2017. There is a [detailed interpretation in Zhihu](https://zhuanlan.zhihu.com/p/25071913). The paper discussed the reason why GAN is hard to train.
