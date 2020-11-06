## Dense Passage Retrieval for Open-Domain Question Answering

+++++++++++++++++++++++++++++++  
<ins>Authors</ins>: V. Karpukhin et al.  
<ins>Date</ins>: 2020-04  
<ins>Tags</ins>:   
+++++++++++++++++++++++++++++++  


## Intro

- Open-domain question answering (QA) is a task that answers factoid questions using a large collection of documents.
- The advances of reading comprehension models suggest a simple two-stage framework:
  1. a **context retriever** first selects a small subset of passages where some of them contain the answer to the question;
  2. a **machine reader** can thoroughly examine the retrieved contexts and identify the correct answer.
- Retrieval in open-domain QA is usually implemented using **TF-IDF** or **BM25**, which matches keywords efficiently with an inverted index and can be seen as representing the question and context in high-dimensional sparse vectors (with weighting). Conversely, the dense latent semantic encoding is complementary to sparse representations by design (e.g., synonyms or paraphrases that consist of completely different tokens may still be mapped to vectors close to each other). Retrieval with dense encodings can be doneefficiently using **maximum inner product search** (MIPS) algorithms.
- However, it is generally believed that learning a good dense vector representation needs a large number of labeled pairs of question and contexts. Dense retrieval methods have thus never be shown to outperform TF-IDF/BM25 for opendomain QA before [ORQA](./lee2019latent.md).


## Contributions

- Although ORQA successfully demonstrates that dense retrieval can outperform BM25, it also suffers from two weaknesses:
  1. First, inverse cloze task (ICT) pretraining is computationally intensive and it is not completely clear that regular sentences are good surrogates of questions in the objective function.
  2. Second, because the context encoder is not fine-tuned using pairs of questions and answers, the corresponding representations could be suboptimal.
- In this paper, they address the question: can we train a better dense embedding model using only pairs of questions and passages (or answers), without additional pretraining? In particular, they focus their research on improving the retrieval component in open-domain QA.
- By leveraging the BERT pretrained model and a dual-encoder architecture, they focus on developing the right training scheme using a relatively small number of question and passage pairs. The embedding is optimized for maximizing inner products of the question and relevant passage vectors, with an objective comparing all pairs of questions and passages in a batch.
- Their *Dense Passage Retriever* (DPR) outperforms BM25 by a large margin but also results in a substantial improvement on the end-to-end QA accuracy compared to *ORQA*.

***

## 1. Dense Passage Retriever (DPR)

### Overview

- DPR uses a dense encoder *E<sub>P</sub>(.)* which maps any text **passage** to a *d*-dimensional real-valued vectors and builds an index for all the *M* passages that we will use for retrieval.
- At run-time, DPR applies a different encoder *E<sub>Q</sub>(.)* that maps the input **question** to a *d*-dimensional vector and retrieves *k* passages of which vectors are the closest to the question vector, where they define the similarity between the question and the passage using the **dot product** of their vectors: *sim(q,p) = E<sub>Q</sub>(q) E<sub>P</sub>(p)*
