---
layout: page
title: KalScope
---

KalScope is an open-source Freestyle Gomoku AI, written during Jan 2014 ~ Mar 2014. KS was an exercise project for training my program optimize techniques initially, but reached pretty good competitiveness.

Kai Sun have linked this project in [his blog](http://www.aiexp.info/gomoku-renju-resources-an-overview.html) thus I would keep this link valid.

You can get source and compiled x64 binary at [GitHub](https://github.com/AeanSR/kalscope). The source code is kind of messy and buggy, I wrote several posts to explain how KS works, at my old blog site. I am trying to migrate my old posts to this new site.

KS is built upon classic alpha-beta search framework. The core innovation of KalScope is called "incremental evaluation". Beside the widely-used zobrist hash key, KS incremental maintains the board present and evaluation, cutting the time cost of evaluation to negligible.