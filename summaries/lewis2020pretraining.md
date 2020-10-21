##

+++++++++++++++++++++++++++++++  
<ins>Authors</ins>: M. Lewis et al.  
<ins>Date</ins>: 2020-06  
<ins>Tags</ins>:   
+++++++++++++++++++++++++++++++  


### Intro

- **Masked language models** (MLMs) provide highly effective self supervision for pre-training by removing and then reconstructing parts of an input text.
- In this paper, they present the first viable pretraining **alternative** to the Masked Language Modeling (MLM) objective.


### Contributions

- They introduce **MARGE** (Multilingual Autoencoder that Retrieves and Generates), a pre-trained sequence-to-sequence model learned with an **unsupervised** multi-lingual multi-document **paraphrasing objective**.
- They train MARGE by self-supervising the **reconstruction** of target text by first **retrieving** a set of related texts (in many languages) and then conditioning on them to maximize the likelihood of generating the original. 
- Tehe noise from the retrieval step is much more diverse than masking: retrieved documents may have little lexical overlap with the target, and may not even be in the same language, but should communicate the same underlying information. In this way, the pre-training task is designed to emphasize paraphrasing and reduce the amount of encyclopedic knowledge the model must memorize.
- Overall, their pre-trained models **capture** elements of traditional paraphrasing, translation, multidocument summarization, and information retrieval tasks **without** any fine tuning. This allows effective **zero-shot learning** in many cases and provides a step towards pre-trained models that can perform any task with little or no fine-tuning.
- With fine-tuning, they achieve **competitive** performance with masked language models on a range of discriminate and generative tasks in many languages, making MARGE the most **generally applicable** pre-training method to date.

***

### Model overview
- During pre-training, the input to the model is a batch of **evidence** documents *z1...zM* and **target** documents *x1...xN*. The model is trained to maximize the likelihood of the targets, conditioned on the evidence documents, and the relevance of each evidence document to each target:
  - (1) The model first computes a **relevance score** *f(xi,zj)* between every pair of documents *xi* and *zi*, by embedding each document and computing their **cosine similarities**.
  - (2) The model then computes the **likelihood** of reconstructing each *xi* conditioned on *z1...zM* and each *f(xi,.)*, using a modified **seq2seq model**. The similarity score encourages the model to attend more to relevant evidence documents. Backpropagating the reconstruction loss therefore improves both the sequence-to-sequence model and the relevance model.
  - (3) They construct batches so that evidence documents are relevant to the targets, using the relevance model for retrieval.
- Training this model is a **chicken-and-egg problem**, as the reconstruction and relevance models cannot be effectively updated if the batches do not contain relevant evidence documents, but batch construction relies on a relevance model. However, they found that, in practice, the model is able to learn from a **random initialization**.
  
 ### (1) Relevance scores

- To learn the relevance scores *f(xi,zj)* for a pair of documents, they train a **document encoder** *g* that maps a list of tokens to a fixed size representation.
- They apply the **same encoder** to both the target and evidence document, and take the **cosine similarity** between their representations. Note that using the same encoder for both the target and evidence documents allows even random models to compute meaningful similarity functions, as documents with higher lexical overlap are more likely to be projected to more similar representations (this is crucial at initialization).
- They encode documents by taking the representation of the **first token** from the top of a **4-layer Transformer**.
  
### (2) Reconstruction Model

- Given a set of evidence documents *z1...zM* and similarity scores *f(xi,zj)*, the reconstruction model computes the likelihood of target document *xi*.
- This provides an auto-encoder loss where the reconstruction of document *xi* is indirectly conditioned on *xi*, but with an intermediate bottleneck provided by the retrieved documents and relevance scores.
- First, the input documents are encoded individually with a bidirectional Transformer, and then the resulting embeddings are concatenated.
- Then, instead of computing a matrix of cross-attention probabilities between all elements of target document *xi* and evidence document *zj* like in the standard Transformer sequence-to-sequence model, they compute cross-attention over a set of evidence documents *z1...zM*, biasing the attention scores with the document relevant score.

### (3) Batch Construction

- Batches are constructed to create evidence document sets *z1...zM* that give useful information for reconstructing target documents *x1...xN*.
- Overall, they **divide** the data into shards of related documents using **simple heuristic constraints**. Specifically, for news text, documents are in the same shard iff they were published on the same date. For Wikipedia, they split articles into chunks of length 512. They create 1000 shards, where all chunks from the same article, or the equivalent article in another language, are in the same shard (otherwise dividing chunks randomly).
- **Periodically**, they compute the similarities between pairs of documents within each shard, using the relevance model, and apply a **threshold** to keep the strongest connections. Specifically, every 10k model updates, they sample a set of shards of documents. For each shard, they compute *f(x,z)* for every pair of target and evidence documents, using the current relevance model. Then, they select which documents are sufficiently related by taking the **top k** most similar document pairs across all pairs in the shard.
- The final batches are constructed to **maximize** connectivity between evidence and target documents. Specifically, the output from the thresholding step is a **bipartite graph** of evidence and target documents with edges between them. A batch is a **subgraph**, and they perform a small local search to find subgraphs maximizing the sum of the weights of all edges in the subgraph. To encourage the model to build **multilingual batches**, edges where the evidence and target are in different languages are given weight 100, and other edges have weight 1.
  
