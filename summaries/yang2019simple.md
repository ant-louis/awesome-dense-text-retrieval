## Simple Applications of BERT for Ad Hoc Document Retrieval

+++++++++++++++++++++++++++++++  
<ins>Authors</ins>: W. Yang et al.  
<ins>Date</ins>: 2019-03   
<ins>Tags</ins>: `bert`   
+++++++++++++++++++++++++++++++  


### Intro

- The dominant approach to ad hoc document retrieval using neural networks today is to deploy the neural model as a reranker over an initial list of candidate documents retrieved using a standard bag-of-words term-matching technique.
- Would it be possible to apply BERT to improve document retrieval as well?


### Contributions

- They explore the use of BERT to ad hoc document retrieval. 
- This required confronting the challenge posed by documents that are typically longer than the length of input BERT was designed to handle.
- They address this issue by applying inference on sentences individually, and then aggregating sentence scores to produce document scores.

***
