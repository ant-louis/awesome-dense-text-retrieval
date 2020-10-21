## REALM: Retrieval-Augmented Language Model Pre-Training

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

### Approach

- <ins>REALM’s generative process</ins>
  - For both pre-training and fine-tuning, REALM takes some input *x* and learns a distribution *p(y|x)* over possible outputs *y*.
  - REALM decomposes *p(y|x)* into two steps: retrieve, then predict. Given an input *x*, they first retrieve possibly helpful documents *z* from a knowledge corpus *Z*. They model this as a sample from the distribution *p(z|x)*. Then, they condition on both the retrieved *z* and the original input *x* to generate the output *y* — modeled as *p(y|z,x)*.
  - Hence, *p(y|x) = SUM_z  p(y|z,x) p(z|x)*.
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
    - For both pre-training and fine-tuning, they train by maximizing the log-likelihood *log p(y|x)* of the correct output *y*. Since both the knowledge retriever and knowledge-augmented encoder are differentiable neural networks, they can compute the gradient of *log p(y|x)* with respect to the model parameters and optimize using stochastic gradient descent.
    - Computational challenge: the marginal probability *p(y|x) = SUM_z  p(y|z,x) p(z|x)* involves a summation over all documents *z* in the knowledge corpus *Z*. They approximate this by instead summing over the top k documents with highest probability under *p(z|x)* — this is reasonable if most documents have near zero probability. Even with this approximation, they still need an efficient way to find the top k documents. As the ordering of documents under *p(z|x)* is the same as under the relevance score *f(x,z)* - which is an inner product - they can employ Maximum Inner Product Search (MIPS) algorithms to find the approximate top k documents, using running time and storage space that scale sub-linearly with the number of documents.
  - **Maximum Inner Product Search (MIPS)**
    - To employ MIPS, they must pre-compute an embedding for every document in Z and construct an efficient search index over these embeddings. However, this data structure will no longer be consistent with *p(z|x)* as the parameters of the networks (thus the embeddings) are constantly updated during training.
    - Their solution is to “refresh” the index by asynchronously re-embedding and re-indexing all documents every several hundred training steps.
    - They asynchronously refresh the MIPS index by running two jobs in parallel: a primary trainer job, which performs gradient updates on the parameters, and a secondary index builder job, which embeds and indexes the documents.
    - During training, the trainer sends the index builder a snapshot of its parameters. The trainer then continues to train while the index builder uses the updated parameters to construct a new index in the background. As soon as the index builder is done, it sends the new index back to the trainer, and the process repeats.
    - While asynchronous refreshes can be used for both pretraining and fine-tuning, in our experiments we only use it for pre-training. For fine-tuning, they just build the MIPS index once (using the pre-trained weights) for simplicity.
  - **Injecting inductive biases into pre-training**
    - **Salient span masking**: During REALM pre-training, they want to focus on examples *x* that require world knowledge to predict the masked tokens. To that end, they mask salient spans (such as “United Kingdom” or “July 1969”) that were identified in advance using a BERT-based tagger trained on CoNLL-2003 data. They show that this significantl outperforms other masking strategies.
    - **Null document**: they add an empty null document to the top k retrieved documents, allowing to consider the cases when no retrieval is necessary.
    - **Prohibiting trivial retrievals**: If the pre-training corpus *X* and the knowledge corpus *Z* are the same, there exists a trivial retrieval candidate *z* that is too informative (the knowledge augmented encoder can trivially predict *y* by looking at the unmasked version of *x* in *z*). If this occurs too often, the knowledge retriever ends up learning to look for exact string matches between *x* and *z*, which does not capture other forms of relevance. For this reason, they exclude this trivial candidate during pre-training.
    - **Initialization**: At the beginning of training, if the retriever does not have good embeddings for the input phrase and the documents, the retrieved documents *z* will likely be unrelated to *x*. This causes the knowledge augmented encoder to learn to ignore the retrieved documents. Once this occurs, the knowledge retriever does not receive a meaningful gradient and cannot improve, creating a vicious cycle. To avoid this cold-start problem, they warm-start the embeddings using a simple training objective known as the **Inverse Cloze Task (ICT)** where, given a sentence, the model is trained to retrieve the document where that sentence came from. For the knowledge-augmented encoder, they warm-start it with BERT pre-training — specifically, the uncased BERT-base model.


### Results

- REALM outperform all previous approaches by a significant margin.
- The generative Open-QA systems based on T5 are surprisingly powerful, but comes at significant computational cost (from Base to 11B, the model is 50 times larger, and gains roughly 5 points in accuracy). In contrast, REALM outperforms the largest T5-11B model while being 30 times smaller.
- Compared to other retrieval-based systems (HardEM, GraphRetriever, PathRetriever) which often retrieve from 20 to 80 documents, their system gets the overall best performance while only retrieving 5 documents

