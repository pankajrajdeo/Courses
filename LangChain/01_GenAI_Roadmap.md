# 📘 Lecture Notes: Introduction to Generative AI and Curriculum Plan

---

## 1. Context and Motivation

* **Speaker’s Journey**:

  * Initially planned to make videos on Generative AI but found it challenging due to:

    * The **fast-evolving nature** of the field (new models, papers, and tools daily).
    * Difficulty designing a stable curriculum.
    * Doubts about whether GenAI is a hype or a sustainable technology.

* **Solution**:

  * 3 months of deep research → built a **mental model** + curriculum for teaching.
  * Two perspectives in GenAI learning:

    1. **Builder’s perspective** (creating foundation models).
    2. **User’s perspective** (using foundation models in applications).

---

## 2. What is Generative AI?

### Definition:

> Generative AI is a type of AI that creates new content (text, images, music, code) by learning patterns from existing data, mimicking human creativity.

### Position in AI Landscape:

* **AI (outer layer)**: Symbolic AI, Expert Systems, Fuzzy Logic, Evolutionary algorithms, NLP, CV.
* **Machine Learning (inside AI)**: Predictive modeling using data & statistics.
* **Deep Learning (inside ML)**: Neural networks, advanced architectures.
* **Generative AI (inside DL)**: Originated after Transformer architecture breakthroughs.

---

## 3. Evolution of AI → Generative AI

* 1980s: Symbolic AI, Expert Systems.
* 1990s–2000s: Fuzzy Logic, Evolutionary Algorithms, NLP, CV.
* 2010s: **Machine Learning dominance** → regression, classification, ranking.
* 2017+: **Transformers + Deep Learning** → enabled GenAI.
* **Unique Power of GenAI**: Can *create* (text, images, videos, code) → mimics human creativity.

---

## 4. Impact Areas of Generative AI

1. **Customer Support**:

   * Shift from human-heavy call centers → chatbot-first support.
   * Reduces workforce requirement (10 → 2–3 agents).

2. **Content Creation**:

   * Blogs, articles, videos, music, code.
   * Hard to distinguish AI vs human-generated text now.

3. **Education**:

   * ChatGPT-like tools = personal tutors.
   * Changing pedagogy in schools/universities.

4. **Software Development**:

   * AI can generate production-ready code.
   * Developers now need fewer manual coding hours.

---

## 5. Is Generative AI a Successful Technology?

Comparison with:

* **Internet**: Clearly solved real-world problems → massive success.
* **Blockchain/Crypto**: Mixed adoption, not solving obvious daily problems.

✅ GenAI answers positively to these 5 questions:

1. **Does it solve real-world problems?**
   → Yes (customer support, education, automation).
2. **Useful on daily basis?**
   → Yes (ChatGPT, copilots, AI assistants).
3. **Impacting world economics?**
   → Yes (e.g., DeepSeek R1 launch wiped \$1 trillion off US tech stocks).
4. **Creating new jobs?**
   → Yes (AI Engineers, Prompt Engineers, LLMOps specialists).
5. **Accessible to everyone?**
   → Yes (parents can use it without coding, via natural language).

**Conclusion**: GenAI is on the *Internet pathway*, not the *Crypto hype pathway*. Worth investing time to learn.

---

## 6. Key Challenge: Information Overload

* Daily advancements = overwhelming.
* Too much noise (FOMO, hype).
* No single trusted knowledge base.
* **Solution**: Design a **mental model** to structure learning.

---

## 7. Mental Model of Generative AI

* **Centerpiece: Foundation Models**
  (e.g., GPT, LLaMA, Claude, Gemini).

  * Large, general-purpose models.
  * Trained on huge datasets + massive compute.
  * Can perform multiple tasks (not task-specific).

* **Two Perspectives**:

  1. **Builder’s Perspective** → How foundation models are created.
  2. **User’s Perspective** → How foundation models are applied.

---

## 8. Builder’s Perspective Curriculum (Creating Foundation Models)

### Prerequisites:

* ML Fundamentals.
* Deep Learning Fundamentals.
* Familiarity with PyTorch (preferred) or TensorFlow.

### Curriculum Steps:

1. **Transformer Architecture**:

   * Encoder, Decoder.
   * Embeddings.
   * Self-Attention.
   * Layer Normalization.
   * Language Modeling.

   **Code Example**: Mini self-attention in PyTorch

   ```python
   import torch
   import torch.nn as nn

   class SelfAttention(nn.Module):
       def __init__(self, embed_size, heads):
           super(SelfAttention, self).__init__()
           self.heads = heads
           self.embed_size = embed_size
           self.values = nn.Linear(embed_size, embed_size, bias=False)
           self.keys = nn.Linear(embed_size, embed_size, bias=False)
           self.queries = nn.Linear(embed_size, embed_size, bias=False)
           self.fc_out = nn.Linear(embed_size, embed_size)

       def forward(self, values, keys, query, mask=None):
           Q = self.queries(query)
           K = self.keys(keys)
           V = self.values(values)
           energy = torch.matmul(Q, K.transpose(-2, -1)) / (self.embed_size ** 0.5)
           attention = torch.softmax(energy, dim=-1)
           out = torch.matmul(attention, V)
           return self.fc_out(out)
   ```

2. **Types of Transformers**:

   * Encoder-only (e.g., BERT).
   * Decoder-only (e.g., GPT).
   * Encoder-Decoder (e.g., T5).

3. **Pre-Training**:

   * Objectives: Next-word prediction, Masked LM.
   * Tokenization strategies (BPE, SentencePiece).
   * Training strategies (single-node, distributed).
   * Challenges: Compute, data scale, optimization.

4. **Optimization**:

   * Model compression: quantization, pruning.
   * Knowledge distillation.
   * Reducing inference latency.

5. **Fine-Tuning**:

   * Task-specific tuning.
   * Instruction tuning.
   * RLHF (Reinforcement Learning with Human Feedback).
   * Parameter-efficient fine-tuning (PEFT, LoRA).

6. **Evaluation**:

   * Benchmarks (MMLU, HumanEval).
   * Metrics: perplexity, BLEU, ROUGE, accuracy.

7. **Deployment**:

   * Model serving.
   * Scaling APIs.
   * Handling latency/cost tradeoffs.

---

## 9. User’s Perspective Curriculum (Using Foundation Models)

1. **Basics**:

   * Open-source models (Hugging Face, LLaMA).
   * Closed-source APIs (OpenAI, Anthropic, Google).
   * Frameworks: LangChain, LlamaIndex.

   **Code Example: Using OpenAI API**

   ```python
   from openai import OpenAI
   client = OpenAI()

   response = client.chat.completions.create(
       model="gpt-4",
       messages=[{"role": "user", "content": "Explain transformers in AI"}]
   )
   print(response.choices[0].message.content)
   ```

2. **Improving LLM Responses**:

   * **Prompt Engineering** (design effective inputs).
   * **RAG (Retrieval-Augmented Generation)**:
     Combine LLM with private documents using vector databases.
   * **Fine-tuning**: Shallow fine-tuning for domain-specific improvements.

   **Code Example: RAG with FAISS**

   ```python
   from langchain.vectorstores import FAISS
   from langchain.embeddings.openai import OpenAIEmbeddings
   from langchain.chains import RetrievalQA

   embeddings = OpenAIEmbeddings()
   db = FAISS.load_local("my_index", embeddings)
   retriever = db.as_retriever()

   qa = RetrievalQA.from_chain_type(
       llm=client, retriever=retriever
   )

   print(qa.run("Summarize my company’s annual report"))
   ```

3. **AI Agents**:

   * LLMs + Tools (e.g., booking tickets, executing code).
   * Frameworks: LangChain Agents, AutoGPT.

4. **LLMOps**:

   * Monitoring, evaluation, scaling of LLM-based applications.
   * Tooling: Weights & Biases, MLflow, LangSmith.

5. **Miscellaneous Topics**:

   * Multimodal models (text+image+audio).
   * Diffusion models (Stable Diffusion).
   * Audio/video generation.

---

## 10. Strategic Advice

* **Builders vs Users**:

  * Builders: Data scientists, ML engineers, research scientists.
  * Users: Software developers, entrepreneurs, product engineers.
  * **Best roles (AI Engineer)** → combine both.

* **Learning Plan**:

  * Parallel coverage of both sides.
  * Small playlists/modules instead of a huge single track.
  * Target: 2–3 videos/week; full curriculum may take \~1 year.

---

## 11. Key Takeaways

* Generative AI is a **transformational technology** with Internet-level impact.
* Two complementary tracks:

  1. **Builder’s side** → Designing and training foundation models.
  2. **User’s side** → Building applications using foundation models.
* Mental model: Everything revolves around **Foundation Models**.
* Curriculum = structured into **modular learning paths**.
* Essential for future career growth in AI-related domains.
