


Getting the ###life out of Living: How adequate are word-pieces for modelling complex morphology?

---

tldr; not at all.

  

{% include info.html text="So this post will review the paper I presented at SIGMORPHON this year (see link in the publications section), that should have been presented at ISCOL too but eventually didn't. Fortunately though, a blog post is more interactive, fun, and "free-style" than a paper, poster or an 8-min presentation and definitely my favorite medium." %}

  

The task: Part-of-speech Tagging

The language: Hebrew

The data: [The Hebrew treebank](https://github.com/OnlpLab/Hebrew_UD)

The means: fine-tuning Multilingual BERT

  

# The problem(s) with Hebrew and all other Semitic languages

TODO: edit so that it'd reflect that english also has morphemes - this is not the issue.

It's meaningless to talk about words in Hebrew. In a language like English we take any space-delimited sequence of characters to be a word, and that's generally a good rule of thumb **for English**. Why? Because a space-delimited sequence (let's just go with *word* from now on, ok?) in English usually has a single meaning, also resulting in a single, simple part-of-speech tag. In Hebrew "words" are composed of u
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc4NDk4MzU1Ml19
-->