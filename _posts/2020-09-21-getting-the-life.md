---
toc: false
layout: post
description: What should have been a poster at ISCOL
categories: [fastpages, jupyter]
image: images/bert.png

title: Getting the ###life out of Living: How adequate are word-pieces for modelling complex morphology?
---
*This post reviews the paper I presented at SIGMORPHON this year (see link in the publications section), that should have been presented at the ISCOL conference too but eventually didn't.*
---
### How adequate are word-pieces for modelling complex morphology?
**tldr;** 
not at all.

The task: Part-of-speech Tagging<br>
The language: Hebrew<br>
The data: [The Hebrew treebank](https://github.com/OnlpLab/Hebrew_UD)<br>
The means: fine-tuning Multilingual BERT<br>

### The challenges with POS-tagging in Hebrew
This section only deals with the challenges in POS-tagging Hebrew, and in another post I'll review the challenges of processing languages with non-concatenative morphology in general.
In NLP, a word is a space-delimited sequence of characters. Each word is composed of at least one **morpheme**. A morpheme is the smallest unit of meaning, and it comes in two shapes: bound and free. A bound morpheme has to be part of a word, like the English plural suffix *-s*, while a free morpheme can stand on its own, like the coordinator *and*. 
The main difference between English and Hebrew in this sense is that Hebrew has a much higher morphemes-to-word ratio than English, so for example the sequence *and when I saw* (4 free morphemes in English) is expressed in Hebrew using a single word - וכשראיתי /ve-kshe-raiti/ (and - bound, when - bound, saw.1st per.sg - free). Each morpheme in the Hebrew word has a different POS tag, which one should we choose? Easy, we don't, we take them all (otherwise we loose valuable syntactic information that we do encode for English) - Introducing the **multi-tag**, which is a POS-tag composed of POS-tags. In the example the multi-tag would be CONJ^REL\^VB with the '^' indicating the correct order.
This is absolutely crucial for the analysis of Hebrew, which has many bound morphemes that carry their own POS-tag.

**Trivia break!**
There is a **single** concept/meaning that is conveyed by a **bound morpheme in English** and a **free morpheme in Hebrew** (it's usually the other way around!), can you find it? Hint in the comments.

Another major challenge in Hebrew is that some morphemes are covert (due to orthographic rules), so the internal structure of a word doesn't necessarily correspond to the surface form. For example the word ב-בית /ba-bayit/ ('in the house') has two morphemes on the surface - in + house, and the definite article is covert (if there wasn't a definite article it would be בבית /b**e**-bayit/).

Lastly, Hebrew has an intertwined nature (a.k.a non-concatenative morphology), which for our purposes means that words can't necessarily be segmented linearly.

### Working through the challenges to find... more challenges...
The widely-accepted conclusion is that in order to parse Hebrew correctly we must first segment each word to its composing morphemes as part of necessary pre-processing, and then we can continue with the regular pipeline like we do for English. 
However, it's usually the case that a single word would have more than one possible segmentation, non of which is a-priori more likely, and the correct one is only recoverable in **context**.... see where this is going, right?

### Along came BERT
Not going to introduce BERT here, it took the world by storm and since then has been used for pretty much anything - turning regular rocks into gold, curing the blind, bringing people back from the dead... and wasn't trained on any of these tasks, what a guy!<br>
*(Hi, this is me being totally sarcastic because it's my blog. As you'll see here and in the future, I'm not the biggest fan of huge-but-dumb models that happen to work well for English)*<br>
So BERT has four important qualities that make it interesting for multi-tagging Hebrew:
1. It's a contextualized model (remember that the correct segmentation relies on context).
2. It (linearly...) segments words into sub-word units called word-pieces.
3. It's multilingual and the Hebrew part of it is trained on a larger corpus than previous pre-neural Hebrew models used.
4. It's really important for reviewers.

### And here's where the study actually begins..
After a long introduction we can now say that this study focuses on the 2nd point from BERT's qualities, and that is its segmentation process. We have established that we need segmentation of words, and implicitly meant that we were looking for the **correct** morphological segmentation. But what if we can't get the correct segmentation? can we safely use BERT's segmentation and succeed on a relatively simple task like POS-tagging?

### Hypothesis and Approach
Since the word-pieces themselves don't reflect the actual morphemes, we hypothesize that segmentation into word-pieces will deteriorate performance for multi-tagging Hebrew. By deteriorating performance we mean that either the accuracy levels will go down, or access to internal structure will be lost. We show how incorporating linguistic knowledge helps maintain access to internal structure as well as improving overall accuracy.

### Some Experimental Settings
The running example throughout will be of the word בבית /'in the house'/, for which the relevant multi-tag is IN^DEF\^NN and the BPE . Broadly speaking, because we only fine tune BERT and not changing the segmentation algorithm (something that would require pre-training from scratch) all we have left to play with are the tags that the word-pieces (WPs from now) receive. Here there are two strategies, one where all the WPs of a given word get the same tag, and another where each WP can get a different tag. The common practice is to give all the WPs the same tag, so let's start with that strategy.
**Predicting the entire multi-tag at word level**
<figure class="image"><img src="https://github.com/stavkl/linguistics-for-nlp/raw/master/images/iscol-post/raw.JPG" width="25%" align="right" alt="">
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus tincidunt leo vel lorem varius, non bibendum lectus volutpat. Suspendisse id augue et ipsum iaculis luctus. Vivamus luctus, ligula euismod pharetra luctus, dolor ex dictum velit, ut tempus mauris purus vel ante. Proin euismod ante sed magna ultrices rhoncus. Nulla et sollicitudin orci. Pellentesque at libero ex. Duis nec lorem eu leo euismod tempus. Fusce semper ultrices felis, id lacinia turpis faucibus nec. In posuere pellentesque arcu accumsan pharetra.





