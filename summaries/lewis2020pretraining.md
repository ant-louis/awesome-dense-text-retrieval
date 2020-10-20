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

### Model

- <ins>Overview</ins>
  - During pre-training, the input to the model is a batch of **evidence** documents *z1...zM* and **target** documents *x1...xN*. The model is trained to maximize the likelihood of the targets, conditioned on the evidence documents, and the relevance of each evidence document to each target:
    - (1) The model first computes a **relevance score** *f(xi,zj)* between every pair of documents *xi* and *zi*, by embedding each document and computing their **cosine similarities**.
    - (2) The model then computes the **likelihood** of reconstructing each *xi* conditioned on *z1...zM* and each *f(xi,.)*, using a modified **seq2seq model**. The similarity score encourages the model to attend more to relevant evidence documents. Backpropagating the reconstruction loss therefore improves both the sequence-to-sequence model and the relevance model.
    - (3) They construct batches so that evidence documents are relevant to the targets, using the relevance model for retrieval.
  - Training this model is a **chicken-and-egg problem**, as the reconstruction and relevance models cannot be effectively updated if the batches do not contain relevant evidence documents, but batch construction relies on a relevance model. However, they found that, in practice, the model is able to learn from a **random initialization**.
- <ins>(1) Relevance scores</ins>
  - To learn the relevance scores *f(xi,zj)* for a pair of documents, they train a **document encoder** *g* that maps a list of tokens to a fixed size representation.
  - They apply the **same encoder** to both the target and evidence document, and take the **cosine similarity** between their representations. Note that using the same encoder for both the target and evidence documents allows even random models to compute meaningful similarity functions, as documents with higher lexical overlap are more likely to be projected to more similar representations (this is crucial at initialization).
  - They encode documents by taking the representation of the **first token** from the top of a **4-layer Transformer**.
  
