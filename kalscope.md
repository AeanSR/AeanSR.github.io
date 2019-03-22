---
layout: page
title: KalScope
---

KalScope is an open-source Freestyle Gomoku AI, written during Jan 2014 ~ Mar 2014. KS was initially an exercise project for training my program optimizing techniques but reached pretty good competitiveness.

Kai Sun has linked this project in [his blog](http://www.aiexp.info/gomoku-renju-resources-an-overview.html) thus I would keep this link valid.

You can get the source and the compiled x64 binary at [GitHub](https://github.com/AeanSR/kalscope). The source code is kind of messy and buggy, I wrote several posts to explain how KS works, at my old blog site. I am trying to migrate my old posts to this new site.

KS is built upon classic alpha-beta search framework. The core innovation of KalScope is called "incremental evaluation". Beside the widely-used Zobrist hash keys, KS incrementally maintains the board presents and evaluations, cutting the time cost of evaluation to negligible.
