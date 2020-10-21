## Generalization through Memorization: Nearest Neighbor Language Models

+++++++++++++++++++++++++++++++  
<ins>Authors</ins>: U. Khandelwal  et al.  
<ins>Date</ins>: 2019-11  
<ins>Tags</ins>: `knn-LM`   
+++++++++++++++++++++++++++++++  


### Intro

- Neural language models (LMs) typically solve two subproblems: 
  1. mapping sentence prefixes to fixed-sized representations;
  2. using these representations to predict the next word in the text.
- They present a new language modeling approach that is based on the hypothesis that the representation learning problem may be easier than the prediction problem. For example, any English speaker knows that *"Dickens is the author of"* and *"Dickens wrote"* will have essentially the same distribution over the next word, even if they do not know what that distribution is.


### Contributions

- They introduce **kNN-LM**, an approach that extends a pre-trained LM by linearly interpolating its next word distribution with a k-nearest neighbors (kNN) model.
- The nearest neighbors are computed according to distance in the pre-trained embedding space and can be drawn from any text collec
tion, including the original LM training data.
- They also show that the approach has implications for efficientl scaling up to larger training sets and allows for effective domain adaptation, by simply varying the nearest neighbor datastore. Training a model on 100-million tokens and using kNN search over a 3-billion token dataset can outperform training the same model on all 3-billion tokens, opening a new path for efficiently using large datasets in language models.

***

#### Nearest Neighbor Language Modeling

- Language models (LMs) assign probabilities to sequences. Given a context sequence of tokens *c<sub>t</sub>=(w<sub>1</sub>,...,w<sub>t-1</sub>)*, autoregressive LMs estimate *p(w<sub>t</sub>|c<sub>t</sub>)*, the distribution over the target token *w<sub>t</sub>*.
- The kNN-LM involves augmenting such a pre-trained LM with a nearest neighbors retrieval mechanism, without any additional training (the representations learned by the LM remain unchanged). This can be done with a single forward pass over a text collection (potentially including the original LM training set), where the resulting context-target pairs are stored in a key-value datastore that is queried during inference.
- <ins>Datastore</ins>
  - Let *f(.)* be the function that maps a context *c* to a fixed-length vector representation computed by the pre-trained LM.
  - Then, given the *i*-th training example *(c<sub>i</sub>,w<sub>i</sub>)* in *D*, they define the key-value pair *(k<sub>i</sub>,v<sub>i</sub>)*, where the key *k<sub>i</sub>* is the vector representation of the context *f(c<sub>i</sub>)* and the value *v<sub>i</sub>* is the target word *w<sub>i</sub>*.
  - The datastore *(K,V)* is thus the set of all key-value pairs constructed from all the training examples in *D*.
  - The keys are the 1024-dimensional representations fed to the feedforward network in the final layer of a **decoder-only Transformer** (they perform a single forward pass over the training set with the **trained** model, in order to save the keys and values).
- <ins>Inference</ins>
  - At test time, given the input context *x* the model generates the output distribution over next words *p<sub>LM</sub>(y|x)* and the context representation *f(x)*.
  - Then, the model queries the datastore with *f(x)* to retrieve its k-nearest neighbors *N* according to a distance function *d(.,.)*. Note that the datastore contains an entry for each target in the training set, which for LMs can be up to billions of examples. To search over this large datastore, they use FAISS, an open source library for fast nearest neighbor retrieval in high dimensional spaces. the FAISS index is created using 1M randomly sampled keys to learn 4096 cluster centroids (keys are quantized to 64-bytes for efficiency). During inference, they retrieve k=1024 neighbors, and the index looks up 32 cluster centroids while searching for the nearest neighbors. They found in preliminary experiments that using **L2 distance** for FAISS retrieval results in better performance for NN-LM, compared to inner product distance.
  - Then, it computes a distribution over neighbors *p<sub>kNN</sub>* based on a softmax of their negative.
  - Finally, they interpolate the nearest neighbor distribution *p<sub>kNN</sub>* with the model distribution *p<sub>LM</sub>* using a tuned parameter *lambda* to produce the final kNN-LM distribution: *p(y|x) = lambda *p<sub>kNN</sub>(y|x)* + (1-lambda) p<sub>LM</sub>(y|x)*


#### Interesting results

- <ins>Question: can retrieving nearest neighbors from data be a substitute for training on it?</ins>
  - To test this, they train a LM on WIKI-100M and use it to build a datastore from WIKI-3B, a corpus 30 times larger than the training set.
  - As expected, the model trained on 3B tokens dramatically outperforms the model trained on 100M tokens, improving perplexity from 19.59 to 15.17. However, adding nearest neighbors retrieval over those 3B examples to the model trained on 100M tokens improves perplexity from 19.59 to 13.73; i.e. *retrieving nearest neighbors from the corpus outperforms training on it*.
  - This result suggests that rather than training language models on ever larger datasets, we can use smaller datasets to learn representations and augment them with kNN-LM over a large corpus.
