##

+++++++++++++++++++++++++++++++  
<ins>Authors</ins>: K. Guu et al.  
<ins>Date</ins>: 2020-02  
<ins>Tags</ins>: `language model`, `neural knowledge retriever`   
+++++++++++++++++++++++++++++++  


### Intro

- Language model pre-training has been shown to capture a surprising amount of world knowledge, crucial for NLP tasks such as question answering. However, this knowledge is stored implicitly in the parameters of the underlying neural network. This makes it difficult to determine what knowledge is stored in the network and where. Furthermore, storage space is limited by the size of the network—to capture more world knowledge, one must train ever-larger networks, which can be prohibitively slow or expensive.


### Contributions

- To capture knowledge in a more modular and interpretable way, they augment language model pre-training with a latent knowledge retriever, which allows the model to retrieve and attend over documents from a large corpus (such as Wikipedia), used during pre-training, fine-tuning and inference.
- The key intuition of their Retrieval-Augmented Language Model (REALM) is to train the retriever using a performance-based signal from unsupervised text: a retrieval that improves the language model’s perplexity is helpful and should be rewarded, while an uninformative retrieval should be penalized.
- They demonstrate the effectiveness of REALM by fine-tuning on the challenging task of Open-domain Question Answering (Open-QA), and outperform all previous methods by a significant margin (4-16% absolute accuracy).

***

### 1. Approach

#### REALM’s generative process

- For both pre-training and fine-tuning, REALM takes some input *x* and learns a distribution *p(y|x)* over possible outputs *y*.
- REALM decomposes *p(y|x)* into two steps: retrieve, then predict. Given an input *x*, they first retrieve possibly helpful documents *z* from a knowledge corpus *Z*. They model this as a sample from the distribution *p(z|x)*. Then, they condition on both the retrieved *z* and the original input *x* to generate the output *y* — modeled as *p(y|z,x)*.
