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

- <ins>REALM’s generative process</ins>
  - For both pre-training and fine-tuning, REALM takes some input *x* and learns a distribution *p(y|x)* over possible outputs *y*.
  - REALM decomposes *p(y|x)* into two steps: retrieve, then predict. Given an input *x*, they first retrieve possibly helpful documents *z* from a knowledge corpus *Z*. They model this as a sample from the distribution *p(z|x)*. Then, they condition on both the retrieved *z* and the original input *x* to generate the output *y* — modeled as *p(y|z,x)*.
  - Hence, *p(y | x) = SUM_z p(y | z, x) p(z | x)*.
- <ins>Model architecture</ins>
  - **Knowledge retriever**
    - The neural knowledge retriever models *p(z|x)*.
    - They define the *relevance score* *f(x,z)* between x and z is defined as the inner product of the vector embeddings.
    - The retrieval distribution *p(z|x)* is then the softmax over all relevance scores.
    - The vector embeddings are implemented using BERT. For both the input and the documents, they only consider the embedding of the [CLS] token as a “pooled” representation of the sequence. They also reduce the dimensionality of the vectors by performing a linear projection. Note that to encode a document, they pass the document title and body as inputs (separated by the [SEP] token).
  - **Knowledge-Augmented Encoder**
    - The knowledge-augmented encoder defines *p(y|z,x)*.
    - Given an input *x* and a retrieved document *z*, they join *x* and *z* into a single sequence that they feed into a Transformer (distinct from the one used in the retriever).
  - **Training**
    - For both pre-training and fine-tuning, they train by maximizing the log-likelihood *log p(y | x)* of the correct output *y*. Since both the knowledge retriever and knowledge-augmented encoder are differentiable neural networks, they can compute the gradient of *log p(y | x)* with respect to the model parameters and optimize using stochastic gradient descent.




### 2. 

