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
is decomposed over a retrieval component *S<sub>retr</sub>(b,q)* and a reader component *S<sub>read</sub>(b,s,q)*: *S(b,s,q) = S<sub>retr</sub>(b,q) + S<sub>read</sub>(b,s,q)*. Here, all scoring components are derived from **BERT**.
- During inference, the model outputs the answer string of the highest scoring derivation.
- <ins>Retriever component</ins> 
  - In order for the retriever to be learnable, we define the retrieval score as the **inner product** of dense vector representations of the question *q* and the evidence block *b*
  - *h<sub>q</sub>=W<sub>q</sub> BERT<sub>Q</sub>(q)[CLS]*
  - *h<sub>b</sub>=W<sub>b</sub> BERT<sub>B</sub>(b)[CLS]*
  - *S<sub>retr</sub>(b,q)=h<sub>q</sub>.h<sub>b</sub>*
- <ins>Rader component</ins> 
  - The reader is a span-based variant of the reading comprehension BERT model.
  - *S<sub>read</sub>(b,s,q) = MLP([h<sub>start</sub>;h<sub>end</sub>])*
- <ins>Learning challenges</ins>
  - The proposed model is conceptually simple, however, inference and learning are challenging since (1) an open evidence corpus presents an enormous search space (over 13 million evidence blocks), (2) how to navigate this space is entirely latent. 
  - They address these challenges by carefully initializing the retriever with unsupervised pre-training.
  - They use the English Wikipedia snapshot from December 20, 2018 as the evidence corpus. The corpus is greedily split into chunks of at most 288 wordpieces based
on BERT’s tokenizer, while preserving sentence boundaries. This results in just over 13 million evidence blocks. The title of the document is included in the block encoder.



## 2. ICT

- An unsupervised analog of a question-evidence pair is a sentence-context pair. Following this intuition, they propose to pre-train their retrieval module with an Inverse Cloze Task (ICT).
- In ICT, a sentence is treated as a pseudo-question, and its context is treated as pseudo-evidence. Given a pseudo-question, ICT requires selecting the corresponding pseudo-evidence out of the candidates in a batch.
- An important aspect of ICT is that it requires learning more than word matching features, since the pseudo-question is not present in the evidence.
- However, they also do not want to dissuade the retriever from learning to perform word matching—lexical overlap is ultimately a very useful feature for retrieval. Therefore, they only remove the sentence from its context in 90% of the examples.
- ICT pre-training provides a sufficiently strong initialization such that ORQA can be fine-tuned end-to-end by simply optimizing the marginal log-likelihood of correct answers that were found.


## 3. Inference

- The block encoder is used as it is (BERT) without further pre-training whereas the question encoder is fine-tuned according to the weakly supervised QA data.
- Since the block encoder is fixed, they can pre-compute all block encodings in the evidence corpus and it can be pre-compiled into an index for fast maximum inner product search.
- With the pre-compiled index, they retrieve the top-k evidence blocks and only compute the expensive reader scores for those k blocks.


## 4. Experiments

- They train and evaluate on data from 5 existing question-answering or reading comprehension datasets:
  1. *Natural Questions*: contains question from aggregated queries to Google Search.
  2. *WebQuestions*: contains questions that were sampled from the Google Suggest API.
  3. *CuratedTrec*: is a corpus of question-answer pairs derived from TREC QA data.
  4. *TriviaQA*: is a collection of trivia questionanswer pairs that were scraped from the web.
  5. *SQuAD*: was designed to be a reading comprehension dataset rather than an open domain QA dataset.
- In the Natural Questions, WebQuestions, and CuratedTrec, the question askers do not already know the answer. This accurately reflects a distribution of genuine information-seeking questions. However, annotators must separately find correct answers, which requires assistance from automatic tools and can introduce a moderate bias towards results from the tool.
- In TriviaQA and SQuAD, automatic tools are not needed since the questions are written with known answers in mind. Question writing is not motivated by an information need. This often results in many hints in the question that would not be present in naturally occurring questions.
- They compare against other retrieval methods by using alternate retrieval scores, but with the same reader. Namely with *BM25* and *unsupervised pooled representations* from neural language models (NNLM and ELMO).
- Results:
  - The first result to note is that BM25 is a powerful retrieval system.
  - On questions that were derived from real users who are seeking information, their ICT pre-trained retriever outperforms BM25 by a large marge.
  - However, in datasets where the question askers already know the answer, the retrieval problem resembles traditional IR and BM25 is better.
