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
