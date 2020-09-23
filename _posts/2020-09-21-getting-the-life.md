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

### The challenge(s) with POS-tagging in Hebrew
This section only deals with the challenges in POS-tagging Hebrew, and in another post I'll review the challenges of processing languages with non-concatenative morphology in general.
In NLP, a word is a space-delimited sequence of characters. Each word is composed of at least one **morpheme**. A morpheme is the smallest unit of meaning, and it comes in two shapes: bound and free. A bound morpheme has to be part of a word, like the English plural suffix *-s*, while a free morpheme can stand on its own, like the coordinator *and*. 
The main difference between English and Hebrew in this sense is that Hebrew has a much higher morphemes-to-word ratio than English, so for example the sequence *and when I saw* (4 free morphemes in English) is expressed in Hebrew using a single word - וכשראיתי /ve-kshe-raiti/ (and - bound, when - bound, saw.1st per.sg - free). Each morpheme in the Hebrew word has a different POS tag, which one should we choose? Easy, we don't, we take them all (otherwise we loose valuable syntactic information that we do encode for English) - Introducing the **multitag**, which is a POS-tag composed of POS-tags. In the example the multitag would be CONJ^REL\^VB with the '^' indicating the correct order.
This is absolutely crucial for the analysis of Hebrew, which has many bound morphemes that carry their own POS-tag.

**Trivia break!**
There is a **single** concept/meaning that is conveyed by a **bound morpheme in English** and a **free morpheme in Hebrew** (it's usually the other way around!), can you find it? Hint in the comments.

### Working through the challenges to find... more challenges...
The widely-accepted conclusion is that in order to parse Hebrew correctly we must first segment each word to its composing morphemes as part of necessary pre-processing, and then we can continue with the regular pipeline like we do for English. 
However, it's usually the case that a single word would have more than one possible segmentation, non of which is a-priori more likely, and the correct one is only recoverable in **context**.... see where this is going, right?

### Along came BERT
Not going to introduce BERT here, it took the world by storm and since then has been used for pretty much anything - turning regular rocks into gold, curing the blind, bringing people back from the dead... and wasn't trained on any of these tasks, what a guy!<br>
*(Hi, this is me being totally sarcastic because it's my blog. As you'll see here and in the future, I'm not the biggest fan of huge-but-dumb models that happen to work well for English)*
So BERT has four important qualities that make it interesting for multi-tagging Hebrew:
1. It's contextualized and remember that the correct segmentation is relies on context
2. It breaks words into pieces (called word-pieces), which is needed in the pre-processing stage.
3. It's multilingual and the Hebrew part is trained on a larger corpus t
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE3ODgzNzA4NywxMzkxMzY5MjQwLC0zMz
UxOTIxNDMsLTMxODE4MTU0LDE3NjA1MjAwNjgsMjA1NTE0MDk1
OCwtODU0ODA5NTMsNTYxNzM2ODI5LDEyNzcxNDA4MDNdfQ==
-->