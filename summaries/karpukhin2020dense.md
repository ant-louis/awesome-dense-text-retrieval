## Dense Passage Retrieval for Open-Domain Question Answering

+++++++++++++++++++++++++++++++  
<ins>Authors</ins>: V. Karpukhin et al.  
<ins>Date</ins>: 2020-04  
<ins>Tags</ins>:   
+++++++++++++++++++++++++++++++  


## Intro

- Open-domain question answering (QA) is a task that answers factoid questions using a large collection of documents.
- The advances of reading comprehension models suggest a simple two-stage framework:
  1. a context retriever first selects a small subset of passages where some of them contain the answer to the question;
  2. a machine reader can thoroughly examine the retrieved contexts and identify the correct answer.
- Retrieval in open-domain QA is usually implemented using TF-IDF or BM25, which matches keywords efficiently with an inverted index and can be seen as representing the question and context in high-dimensional sparse vectors (with weighting). Conversely, the dense latent semantic encoding is complementary to sparse representations by design (e.g., synonyms or paraphrases that consist of completely different tokens may still be mapped to vectors close to each other). Retrieval with dense encodings can be doneefficiently using maximum inner product search (MIPS) algorithms.
- However, it is generally believed that learning a good dense vector representation needs a large number of labeled pairs of question and contexts. Dense retrieval methods have thus never be shown to outperform TF-IDF/BM25 for opendomain QA before (ORQA)[]


## Contributions

- 

***

