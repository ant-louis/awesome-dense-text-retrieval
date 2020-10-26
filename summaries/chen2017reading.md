## Reading Wikipedia to Answer Open-Domain Questions

+++++++++++++++++++++++++++++++  
<ins>Authors</ins>: D. Chen et al.  
<ins>Date</ins>: 2017-04  
<ins>Tags</ins>: `tf-idf`, `lstm`     
+++++++++++++++++++++++++++++++  


### Intro

- This paper considers the problem of answering factoid questions in an open-domain setting using Wikipedia as the unique knowledge source, such as one does when looking for answers in an encyclopedia.
- Using Wikipedia articles as the knowledge source causes the task of question answering (QA) to combine the challenges of both large-scale open-domain QA and of machine comprehension of text.
- In order to answer any question, one must first retrieve the few relevant articles among more than 5 million items, and then scan them carefully to identify the answer. We term this setting, **machine reading at scale** (MRS).


### Contributions

- They develop DrQA, a strong system for question answering from Wikipedia composed of: 
  1. *Document Retriever*, a module using **bigram hashing** and **TF-IDF** matching designed to, given a question, efficiently return a subset of relevant articles;
  2. *Document Reader*, a multi-layer **recurrent neural network** (RNN) machine comprehension model trained to detect answer spans in those few returned documents.
- Their experiments show that *Document Retriever* outperforms the built-in Wikipedia search engine (ElasticSearch) and that *Document Reader* reaches state-of-the-art results on the very competitive SQuAD benchmark.

***

### 1. Approach

#### Document Retriever

- They use an efficient (non-machine learning) document retrieval system to first narrow our search space and focus on reading only articles that are likely to be relevant.
- A simple **inverted index lookup** followed by **term vector model scoring** performs quite well on this task for many question types. Specifically, articles and questions are compared as **TF-IDF** weighted bag-of-word vectors. They further improve their system by taking local word order into account with n-gram features (their best performing system uses bigram counts).
- They use *Document Retriever* as the first part of their full model, by setting it to return 5 Wikipedia articles given any question. Those articles are then processed by *Document Reader*.


#### Document Reader

- Given a question *q* consisting of *l* tokens *{q1,...,ql}* and a document or a small set of documents of *n* paragraphs where a single paragraph consists of *m* tokens, they develop an **RNN** model that they apply to each paragraph in turn and then finally aggregate the predicted answers.
- <ins>Paragraph encoding</ins>
  - They first represent all tokens *pi* in a paragraph as a sequence of feature vectors comprised of the following parts:
    - *Word embedding*: 300-dimensional **Glove** word embeddings trained from 840B Web crawl data.
    - *Exact match*: whether *pi* can be exactly matched to one question word in *q*, either in its original, lowercase or lemma form.
    - *Token features*: part-of-speech (POS) and named entity recognition (NER) tags and its (normalized) term frequency (TF).
    - *Aligned question embedding*: word embedding (the Glove ones) weighted with an attention score that captures the similarity between the token *pi* and each question words *qj*.
  - They then pass the sequence of feature vectors as the input to a recurrent neural network which outputs a token representation for each token *pi*. Specifically, they choose to use a multi-layer bidirectional long short-term memory network (LSTM), and take the token representation as the concatenation of each layer’s hidden units in the end.
- <ins>Question encoding</ins>
  - They only apply another recurrent neural network on top of the word embeddings of *qi* and combine the resulting hidden units into one single vector, computed by weighting each word vector with a learned attention score encoding the importance of each question word.
- <ins>Prediction</ins>
  - At the paragraph level, the goal is to predict the span of tokens that is most likely the correct answer.
  - They take the the paragraph vectors and the question vector as input, and simply train two classifiers independently for predicting the two ends of the span (*start* and *end*). Specifically, they compute the probabilities of each token being *start* and *end*.
  - During prediction, they choose the best span from token *i* to token *i'* such that *i <= i' <= i+15* and *P<sub>start</sub>(i) x P<sub>end</sub>(i')* is maximized.
