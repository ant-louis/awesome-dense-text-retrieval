##

+++++++++++++++++++++++++++++++  
<ins>Authors</ins>: M. Lewis et al.  
<ins>Date</ins>: 2020-06  
<ins>Tags</ins>:   
+++++++++++++++++++++++++++++++  


### Intro

- Masked language models (MLMs) provide highly effective self supervision for pre-training by removing and then reconstructing parts of an input text.
- In this paper, they present the first viable pretraining alternative to the Masked Language Modeling (MLM) objective.


### Contributions

- They introduce **MARGE** (Multilingual Autoencoder that Retrieves and Generates), a pre-trained sequence-to-sequence model learned with an unsupervised multi-lingual multi-document paraphrasing objective.
- They train MARGE by self-supervising the reconstruction of target text by first retrieving a set of related texts (in many languages) and then conditioning on them to maximize the likelihood of generating the original. 
- Tehe noise from the retrieval step is much more diverse than masking: retrieved documents may have little lexical overlap with the target, and may not even be in the same language, but should communicate the same underlying information. In this way, the pre-training task is designed to emphasize paraphrasing and reduce the amount of encyclopedic knowledge the model must memorize.
- Overall, their pre-trained models capture elements of traditional paraphrasing, translation, multidocument summarization, and information retrieval tasks without any fine tuning. This allows effective zero-shot learning in many cases and provides a step towards pre-trained models that can perform any task with little or no fine-tuning.
- With fine-tuning, they achieve competitive performance with masked language models on a range of discriminate and generative tasks in many languages, making MARGE the most generally applicable pre-training method to date.

***
