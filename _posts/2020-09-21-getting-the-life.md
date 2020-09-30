---
toc: false
layout: post
description: The theory behind the SIGMORPHON 2020 paper
categories: [fastpages, jupyter]
image: images/bert.png
hide: false

title: "Getting the ###life out of Living: How adequate are word-pieces for modelling complex morphology?"
---

*This post reviews the paper I presented at SIGMORPHON this year from its theoretical perspective. During the work on this paper we got a lot of good comments on the theoretical side of things, and a lot of questions about the actual implementation and the fine-tuning process, so there'll be another blog post with a walkthrough on one of the settings described here*

---
### tl;dr
The task: Part-of-speech Tagging<br>
The language: Hebrew<br>
The data: [The Hebrew treebank](https://github.com/OnlpLab/Hebrew_UD)<br>
The means: fine-tuning Multilingual BERT<br>
So how adequate are word-pieces? not at all


### The challenges with POS-tagging in Hebrew
This section only deals with the challenges in POS-tagging Hebrew, and in another post I'll review the challenges of processing languages with non-concatenative morphology in general.
In NLP, a word is a space-delimited sequence of characters. Each word is composed of at least one **morpheme**. A morpheme is the smallest unit of meaning, and it comes in two shapes: bound and free. A bound morpheme has to be part of a word, like the English plural suffix *-s*, while a free morpheme can stand on its own, like the coordinator *and*. 

The main difference between English and Hebrew in this sense is that Hebrew has a much higher morphemes-to-word ratio than English, so for example the sequence *and when I saw* (4 free morphemes in English) is expressed in Hebrew using a single word - וכשראיתי /ve-kshe-raiti/ (and - bound, when - bound, saw.1st per.sg - free). Each morpheme in the Hebrew example has a different POS tag, which one should we choose to represent the whole word? Easy, we don't choose, we take them all (otherwise we loose valuable syntactic information that we do encode for English).<br>
Introducing the **multi-tag**, which is a POS-tag composed of POS-tags. In the example the multi-tag would be CONJ^REL\^VB. We use the ^ to denote that the order of the components is crucial, that is CONJ^REL^VB != CONJ^VB^REL (if such tag existed).<br>

> **Trivia break!**<br>
There is a **single** concept/meaning that is conveyed by a **bound morpheme in English** and a **free morpheme in Hebrew** (it's usually the other way around!), can you find it? Hint in the comments.

Another major challenge in Hebrew is that some morphemes are covert (due to orthographic rules), so the internal structure of a word doesn't necessarily correspond to the surface form. For example the word ב-בית /ba-bayit/ ('in the house') has two morphemes on the surface - in + house, and the definite article is covert (if there wasn't a definite article it would be בבית /b**e**-bayit/), but we don't write the vowels in Hebrew script.

Lastly, Hebrew has an intertwined nature (a.k.a non-concatenative morphology), which for our purposes means that words can't necessarily be segmented linearly.

### Working through the challenges to find... more challenges!
The widely-accepted conclusion is that in order to parse Hebrew correctly we must first segment each word to its composing morphemes as part of a necessary pre-processing, and then we can continue with the regular pipeline like we do for English. 
However, it's usually the case that a single word would have more than one possible segmentation, non of which is a-priori more likely, and the correct one is only recoverable in **context**.... 

### Along came BERT
Not going to introduce BERT here, it took the world by storm and since then has been used for pretty much anything - turning regular rocks into gold, curing the blind, bringing people back from the dead... and wasn't trained on any of these tasks, what a guy!<br>
*(Hi, this is me being totally sarcastic because it's my blog. As you'll see here and in the future, I'm not the biggest fan of huge-but-dumb models that happen to work well for English)*<br>
BERT has four important qualities that make it interesting for multi-tagging Hebrew:
1. It's a contextualized model (remember that the correct segmentation relies on context).
2. It segments words into sub-word units called word-pieces as part of its pre-processing (but uses a linear segmentation, more on that later).
3. Its multilingual version includes a Hebrew model which is trained on a larger corpus than what was used in previous pre-neural models for Hebrew.
4. It's really important for reviewers.

### And here's where the study actually begins..
After a long introduction we can now say that this study focuses mostly on the 2nd point above which is the segmentation process. We have established that we need segmentation of words into sub-word units, and by that we implicitly meant that we were looking for the **correct** segmentation (i.e. into the actual morphemes). But what if we can't get the correct segmentation? it is kinda hard...  Can we just use BERT's segmentation into word-pieces (hence WPs) and get good enough results?

### Hypothesis and Approach
The fact is that WPs do not reflect the actual morphemes, so we hypothesize that segmentation into WPs will deteriorate performance for multi-tagging Hebrew. By deteriorating performance we mean that either the accuracy levels will go down, or access to internal structure will be lost. We show how incorporating linguistic knowledge helps maintain access to internal structure as well as improving overall accuracy.

### Some Experimental Settings
The running example throughout will be of the word בבית /'in the house'/, for which the relevant multi-tag is IN^DEF^NN and the BPE segmentation is  ב, ##בית . Broadly speaking, because we only fine tune BERT and not changing the segmentation algorithm (something that would require pre-training from scratch) all we have left to play with are the tags that the WPs receive. There are two strategies, one where all the WPs of a given word get the same tag, and another where each WP can get a different tag. The common practice is to give all the WPs the same tag, so let's start with that.<br>

**Predicting the entire multi-tag at word level**
<figure class="image"><img src="https://github.com/stavkl/linguistics-for-nlp/raw/master/images/iscol-post/raw.JPG" width="25%" align="right" alt=""></figure>

In this setting we predict for each WP the complete multi-tag of the entire word, that is, for the WP 'ב' predict the multi tag IN^DEF^NN and also for the second WP '##בית'. We choose the prediction of the first WP to be the multi-tag for the whole word.
On the upside, this is the easiest method to apply, but nothing comes without a price - we cannot generalize to new unseen tags, so if the model haven't seen the tag IN^DEF^NN during fine-tuning it won't be able to predict it in the evaluation. Also, the inner structure of the word remain inaccessible - eventually the model's prediction is that the word בבית has a multi-tag of IN^DEF^NN, but we can't infer which part of the word contributed the NN tag, for example, we can only tell that it's there. For POS tagging that might not make much of a difference, but for other tasks down the pipeline, like NER, it's really important to recognize where a proper noun begins and ends, for example.
The F1 score for this setting is **94.09**. We can do better! We just need to be a little more sophisticated with the linguistic knowledge that we have and still didn't use.

**Predicting Prefix (multi)-tag and the host (multi)-tag**
<figure class="image"><img src="https://github.com/stavkl/linguistics-for-nlp/raw/master/images/iscol-post/prefixhost.JPG" width="25%" align="right" alt=""></figure>
Hebrew words, as described before, are composed of morphemes, some morphemes come before the stem (called prefixes, like *dis-* in English), and some come after (like *-ness*). While in English affixes don't usually carry a separate POS-tag, in Hebrew they usually do.
In this section we use the fact that we can separate, fairly easily, the prefix from the rest of the word (which we'll call the host). For the record, separating the suffix is also possible but not as easily, so we didn't do that.

We fine-tune the model twice, once for the prefix classifier and once for the host classifier. Here the prefix classifier only predicts prefixes - it predicts IN^DEF   for both WPs, and the host classifier predicts NN for both WPs. We then combine the results to a single tag IN^DEF^NN. 

Because we split and reassemble the tags we can generalize better to unseen tags, and we also get more access to the inner structure as the distinction between prefix and host is essentially a distinction between function and content (respectively), and it makes intuitive sense to fine-tune two models separately. It also yields a slightly better F1 score of **94.22**.

**Strategy Summary**
<figure class="image"><img src="https://github.com/stavkl/linguistics-for-nlp/raw/master/images/iscol-post/strategy1.JPG" width="75%" align="center" alt=""></figure>

So far we see that incorporating linguistic knowledge can help in both accuracy and access to internal structure. Next we'll see what we can get when we step out of the common practices.<br>

**Predicting a single tag per word piece (decomposed)**
Moving on to the strategy where each WP can get a different tag. Let's say you're not entirely convinced that WPs don't reflect morphology (even though I told you it's a fact, they weren't designed to do that), what will happen if we just predict a single, simple POS tag for each WP? The advantages are obvious, we'll immediately know which WP contributed which tag, and the number of simple tags is really not that big (about 50) so we could generate any new multi-tag we'd like, right? 
<figure class="image"><img src="https://github.com/stavkl/linguistics-for-nlp/raw/master/images/iscol-post/decomposed.JPG" width="25%" align="right" alt=""></figure>
Not exactly. The obvious advantages are still true, but it is also true that the overall F1 score drops to as low as **76.65**, and that's because we lose so much information (and therefore accuracy) to those damn covert morphemes! See the example in the figure and remember that the multi-tag of the word בבית is IN^DEF^NN, but the tag DEF is not (and can't be) part of the output as it's not attached to any WP.<br>

**(Multi-)tag per word piece (decomposed informed)**
So ideally we would want the first WP ב to reflect that its tag is IN but as a WP it also encapsulates the DEF tag, and we want the second WP בית to have just the tag NN.
<figure class="image"><img src="https://github.com/stavkl/linguistics-for-nlp/raw/master/images/iscol-post/decomposedinformed.JPG" width="25%" align="right" alt=""></figure> 
Essentially that means we want each WP to receive a (possibly multi-)tag that is composed *only* of the morphemes contained in the WP, covert or not. That way we get all the advantages from before, without falling to the covert morphemes pitfall.<br>

So I wrote a function that does just that, based on the the data we have in the Hebrew Treebank and some linguistic traits and heuristics we know about the partial order of POS-tags in Hebrew and the covert nature of some POS-tags (namely the definite article). This setting gets an F1 score of **88.71**.<br>


**Overall Settings Summary**
<figure class="image"><img src="https://github.com/stavkl/linguistics-for-nlp/raw/master/images/iscol-post/strategy1-2.JPG" width="75%" align="center" alt=""></figure>
Overall, we saw that BERT's segmentation of words into sub-word units might work well for English since the linguistic properties of English (namely its linear nature and low morphemes-to-word ratio) work well with the BPE algorithm that BERT uses. Unfortunately this doesn't hold for Hebrew too, where we want the correct morphological segmentation. Using linguistic knowledge can help us overcome some of the challenges of processing Hebrew, but still there's a trade-off between prediction accuracy and the access to the internal structure that is not yet solved. Also note that all the linguistic knowledge we incorporated was really only introduced to the model at the very late stage of fine-tuning, which is probably too late (all the representations were already pre-trained on unsuitable segmentation). I'm no longer sure this calls for a designated BERT-for-Hebrew model, like in other languages, but all of these need to be taken into consideration anyway.
