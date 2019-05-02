---
layout: post
title: Nelf Name Generator V3
---

Research progress in deep learning has slowed down in last 2 years. I decide to try some classical models.

Although the quality of V2 (Wasserstein GAN Based) is pretty good, it is still hard to be compiled along
with a bloated deep learning framework into a mini fan game. This time I would try to use a model that are
good with quality, but easy to be implemented without frameworks, with a light generating cost.

I selected _Hidden Markov Model_ this time. The code (both training and generating / evaluating), dataset
and pretrained models have been uploaded to [GitHub](https://github.com/AeanSR/NameGen-v3).

Online service is also available at [Server _Nighthaven_](https://nighthaven.aean.net/namegen).

<div style="position:relative;padding-bottom:56.25%;background-color: white;height:0;"><iframe src="https://nighthaven.aean.net/namegen" scrolling="no" border="0" frameborder="no" framespacing="0" style="position:absolute;top:0;left:0; height: 100%; width: 100%;">
  <p>Your browser does not support iframes. Please visit [Server _Nighthaven_](https://nighthaven.aean.net/namegen) manually.</p>
</iframe></div>

More races other than nelf are on the way.
