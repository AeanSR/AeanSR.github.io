---
layout: post
title: WoW Dictionary
---

Did some study about NLP.

We are planning to build a patch note bot, to publish patch changes to the forum automatically. I am working on the tooltips compare algorithm.

Compare between strings are just simple with dynamic programming. If you don't know how to do this [just read this slide](http://bioinfo.ict.ac.cn/~dbu/AlgorithmCourses/Lectures/Lec6-EditDistance.pdf). That algorithm works on the granularity of alphabets. To scale the granularity to words, we need to do word split before we start this algorithm.

Split latin words is easy. It won't be more than [5 lines of code](http://stackoverflow.com/questions/236129/split-a-string-in-c#adzerk342792211).

However it is hard to split chinese words. Billions of chinese Ph.D students graduates each year, by spamming all kinds of word segmentation methods. We need a dictionary before doing word segmentation. A dictionary for WoW.

I extracted all strings from DBC and filtered out zhCN corpus. Then implemented [this](http://www.matrix67.com/blog/archives/5044) algorithm.

The dictionary I got is ok for me. There are still garbage words in it, but rare. Some words are filtered out because they are rarely used in tooltips, e.g. "灰谷"(_Ashenvale_, I'm really unhappy about this :X).

I would make my dictionary extractor open when the bot is ready. If you are looking for the dictionary too, you could get it from my [github](https://github.com/AeanSR/wow_dict).
