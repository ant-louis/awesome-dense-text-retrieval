##

+++++++++++++++++++++++++++++++  
<ins>Authors</ins>: M. Lewis et al.  
<ins>Date</ins>: 2020-06  
<ins>Tags</ins>:   
+++++++++++++++++++++++++++++++  


### Intro

- 


### Contributions

- In this paper, they present the first viable pretraining alternative to MLMs; self-supervision is instead provided by learning to paraphrase collections of related documents in many languages.
- More specifically, they introduce **MARGE** (Multilingual Autoencoder that Retrieves and Generates), a pre-trained sequence-to-sequence model learned with an unsupervised multi-lingual multi-document paraphrasing objective.
- They train MARGE by self-supervising the reconstruction of target text by first retrieving a set of related texts (in many languages) and then conditioning on them to maximize the likelihood of generating the original. This objective noisily captures aspects of paraphrase, translation, multi-document summarization, and information retrieval, allowing for strong zero-shot performance on several tasks.
- 

***
