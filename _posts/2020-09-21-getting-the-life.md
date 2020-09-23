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

### 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTI3Nzg5NjI0MCwxNzYwNTIwMDY4LDIwNT
UxNDA5NTgsLTg1NDgwOTUzLDU2MTczNjgyOSwxMjc3MTQwODAz
XX0=
-->