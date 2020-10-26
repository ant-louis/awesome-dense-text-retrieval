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

### Approach

#### Document Retriever

- They use an efficient (non-machine learning) document retrieval system to first narrow our search space and focus on reading only articles that are likely to be relevant.
- A simple **inverted index lookup** followed by **term vector model scoring** performs quite well on this task for many question types. Specifically, articles and questions are compared as **TF-IDF** weighted bag-of-word vectors. They further improve their system by taking local word order into account with n-gram features (their best performing system uses bigram counts).
- They use *Document Retriever* as the first part of their full model, by setting it to return 5 Wikipedia articles given any question. Those articles are then processed by *Document Reader*.
