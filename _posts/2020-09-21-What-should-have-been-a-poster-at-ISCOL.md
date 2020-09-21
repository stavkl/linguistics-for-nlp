---
toc: false
layout: post
description: Overview of the SIGMORPHON 2020 paper.
categories: [markdown]
title: What should have been a poster at ISCOL
---
*This post will review the paper I presented at SIGMORPHON this year (see link in the publications section), that should have been presented at the ISCOL conference too but eventually didn't.*
---
### Getting the ###life out of Living: How adequate are word-pieces for modelling complex morphology?
**tldr;** 
not at all.

The task: Part-of-speech Tagging<br>
The language: Hebrew<br>
The data: [The Hebrew treebank](https://github.com/OnlpLab/Hebrew_UD)<br>
The means: fine-tuning Multilingual BERT<br>
### The challenge(s) with POS-tagging in Hebrew
This section only deals with the challenges in POS-tagging Hebrew, and in another post I'll review the challenges of processing non-concatenative morphology in general.
In NLP, a word is a sequence of characters, separated by a space
In English, each word (i.e. space-delimited sequence of characters) has a single, simple POS tag. 

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIyODM4NDI2MywtMTIzOTAxMzg1LC03Nj
k5ODMyNTEsMTQ2NzE1Njg0OSwtMTYxOTE2Njk1LDEwOTY3Mjkz
MTYsMTk4NTI2NDE4OV19
-->