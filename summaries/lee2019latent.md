## Latent Retrieval for Weakly Supervised Open Domain Question Answering

+++++++++++++++++++++++++++++++  
<ins>Authors</ins>: K. Lee et al.  
<ins>Date</ins>: 2019-06  
<ins>Tags</ins>:   
+++++++++++++++++++++++++++++++  


![Model illustration](images/ORQA.png)


## Intro

- Recent work on open domain question answering (QA) assumes **strong supervision** of the supporting evidence and/or assumes a **blackbox** information retrieval (IR) system to retrieve evidence candidates.
- They argue that both are **suboptimal**, since gold evidence is not always available, and QA is fundamentally **different** from IR. Whereas IR is concerned with lexical and semantic matching, questions are by definition **under-specified** and require more language understanding, since users are explicitly looking for unknown information.
- Instead of being subject to the recall ceiling from blackbox IR systems, **we should directly learn to retrieve using question-answering data**.


## Contributions

- They introduce the first **Open-Retrieval Question Answering** system (ORQA). ORQA learns to retrieve evidence from an open corpus, and is supervised only by question answer **string** pairs.
- The main **challenge** to fully end-to-end learning is that retrieval over the open corpus must be considered a **latent variable** that would be **impractical** to train from scratch. The **key** insight of this work is that end-to-end learning is **possible** if we pre-train the retriever with an unsupervised **Inverse Cloze Task** (ICT).
- They evaluate ORQA on open versions of five existing **QA datasets**. On datasets where the question writers already know the answer, the retrieval problem resembles traditional IR, and BM25 provides state-of-the-art retrieval. On datasets where question writers do not know the answer, they show that learned retrieval is crucial, providing improvements of 6 to 19 points in exact match over BM25.

***

## 1. Model

- They propose an end-to-end model where the retriever and reader components are jointly learned.
- Models define a scoring function *S(b,s,q)* indicating the goodness of an answer derivation *(b,s)* given a question *q*. Typically, this scoring function
is decomposed over a retrieval component *S<sub>retr</sub>(b,q)* and a reader component *S<sub>read</sub>(b,s,q)*: *S(b,s,q)* = *S<sub>retr</sub>(b,q)* + *S<sub>read</sub>(b,s,q)*.

### ICT

- In ICT, a sentence is treated as a pseudo-question, and its context is treated as pseudo-evidence. Given a pseudo-question, ICT requires selecting the corresponding pseudo-evidence out of the candidates in a batch.
- ICT pre-training provides a sufficiently strong initialization such that ORQA can be fine-tuned end-to-end by simply optimizing the marginal log-likelihood of correct answers that were found.
