##

+++++++++++++++++++++++++++++++  
<ins>Authors</ins>: K. Guu et al.  
<ins>Date</ins>: 2020-02  
<ins>Tags</ins>: `language model`, `neural knowledge retriever`   
+++++++++++++++++++++++++++++++  


### Intro

- Language model pre-training has been shown to capture a surprising amount of world knowledge, crucial for NLP tasks such as question answering. However, this knowledge is stored implicitly in the parameters of a neural network, requiring ever-larger networks to cover more facts.


### Contributions

- To capture knowledge in a more modular and interpretable way, they augment language model pre-training with a latent knowledge retriever, which allows the model to retrieve and attend over documents from a large corpus (such as Wikipedia), used during pre-training, fine-tuning and inference.
- They demonstrate the effectiveness of Retrieval-Augmented Language Model pre-training (REALM) by fine-tuning on the challenging task of Open-domain Question Answering (Open-QA), and outperform all previous methods by a significant margin (4-16%absolute accuracy).

***
