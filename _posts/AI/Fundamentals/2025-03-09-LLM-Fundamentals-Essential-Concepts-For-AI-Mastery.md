---
title: LLM Fundamentals - Essential Concepts for AI Mastery
date: 2025-03-09
categories: [AI, Fundamentals]
tags: [AI, LLMs, Token, Transformer, Self-Attention, Loss Function, Prompt, Temperature, Pre-Training, Fine-Tuning, RLHF, Distillation, Parameters]
author: kai
permalink: /posts/ai/fundamentals/llm-fundamentals-essential-concepts-for-ai-mastery/
---

# 🚀 LLM Fundamentals: Essential Concepts for AI Mastery
Large Language Models (LLMs) are transforming AI applications, but understanding their **core concepts** is essential for developers, researchers, and AI enthusiasts.  

This post introduces **key LLM concepts**, including **Tokens, Transformers, Self-Attention, Fine-Tuning, RLHF, and more**, providing a **solid foundation** for anyone working with AI.  

> **At its core, an LLM predicts the next token given a sequence of previous tokens.**  
> But what exactly are **tokens**, **transformers**, and other key components? 


## 📌 1. Token: The Basic Unit of Text Processing & AI Billing
A **Token** is the fundamental unit of text processing in **Large Language Models (LLMs)**. Since LLMs are **computationally expensive**, APIs **charge based on token usage** to ensure **efficient resource allocation and cost control**.  

Unlike using **characters or words** as a direct measure, tokens are generated **after the text is processed by a model’s tokenizer**, meaning: 😵‍💫 **Token ≠ Word ≠ Character** 

### Tokenization 
- Models don't directly understand words; instead, they **tokenize** input before processing.  
- Tokenization **varies across languages** and **different AI architectures**.  
    - **Token Count** → Affects processing cost and token limits.  
    - **Model Accuracy** → Different tokenization methods influence how well an AI understands context.  
    - **Efficiency** → Some tokenizers create fewer tokens for the same input, making the model more efficient.  

#### Tokenization Examples
- **Example:**
    - `"Hello, world!"` → `[Hello] [ , ] [ world ] [ ! ]` (Word-level tokenization)  
    - `"Understanding"` → `[Under] [stand] [ing]` (Subword tokenization)  
    - `"AI"`            → `["A", "I"]` (Character-Level Tokenization)

#### Special Tokens in LLMs
**Special tokens** are unique markers used in LLMs to help structure input, control generation, and handle unknown or missing data. These tokens improve **text processing, sequence handling, and response control** in AI models. <br>
❗️*The forms of special tokens are highly diverse across model providers.*

| **Token** | **Meaning** | **Purpose** | **Example Usage** |
|------------|------------|------------|-----------------|
| **`<BOS>` (Beginning of Sequence)** | Marks the start of an input.| Helps models identify the start of text processing,<br> ensuring structured and well-defined output. | **Prompt:** `"Tell me a story about a time-traveling scientist."`<br>**Output Without `<BOS>`:** `"Once, a scientist discovered a hidden portal in his lab... He stepped inside and found himself in ancient Rome..."` <br>**Output With `<BOS>`:** `"<BOS> Once, a scientist discovered a hidden portal in his lab... He stepped inside and found himself in ancient Rome..."` |
| **`<EOS>` (End of Sequence)** | Marks the end of an output. | Marks the stopping point.<br>Prevents infinite, excessive or redundant text generation | **Prompt:** `"Generate a short poem about the ocean."`<br>**Output Without `<EOS>`:** `"The ocean is deep, vast, and endless... Waves crashing against the shore... The seagulls fly high... The ocean is deep, vast, and endless... (repeats infinitely)."`<br>**Output With `<EOS>`:** `"The ocean is deep, vast, and endless. <EOS>"`|
| **`<PAD>` (Padding Token)** | Fills empty spaces in fixed-length inputs. | Ensures consistent sequence length,<br> making batch processing more efficient.| **Sentence 1:** `"Hello there!"` (2 tokens)<br>**Sentence 2:** `"Good morning, how are you?"` (5 tokens)<br>**Without padding:** The model **fails** to process both sentences together. |
| **`<UNK>` (Unknown Token)** | Represents an unknown word. | Prevents errors when encountering rare or unseen words.| **Input:** `"Meet me at Café Déjà Vu."`<br>**Model Vocabulary Missing "Déjà Vu"** → It replaces it with `<UNK>` |
| **`<SEP>` (Separator Token)** | clearly distinguishes input and output sections. | Used in tasks like QA and multi-turn conversations,<br> making the model’s response more structured. | **Prompt:** `"Answer the question: What is the capital of Japan?"`<br>**Output Without `<SEP>`:** `"What is the capital of Japan? The capital of Japan is Tokyo."`<br>**Output With `<SEP>`:** `"What is the capital of Japan? <SEP> The capital of Japan is Tokyo."`|
| **`<CLS>` (Classification Token)** | Used for text classification tasks. | Helps in sentiment analysis, spam detection, etc. | **Prompt:** `"Analyze the sentiment of this sentence: 'This product is amazing!'"` <br>**Output Without `<CLS>`:** `"This product is amazing! Positive"` <br>**Output With `<CLS>`:** `"<CLS> This product is amazing! <SEP> Sentiment: Positive"` |
| **`<MASK>` (Masked Token)** | Represents hidden words during training. | Used in Masked Language Models (MLMs) like BERT. | **Prompt:** `"Fill in the missing word: 'The sky is <MASK>.'"`<br>**Output Without `<MASK>`:** `"The sky is blue or cloudy or sometimes gray."`<br>**Output With `<MASK>`:** `"The sky is <MASK>. <MASK> → blue"` |

> 🔗 **Test it yourself:**
>
> [Hugging Face Tokenizer Playground](https://huggingface.co/tokenizers)
>
> [OpenAI Tokenizer Playground](https://platform.openai.com/tokenizer) 


### Limitation
- LLMs have a **maximum token processing limit** for each request.  
- Example: OpenAI - Depending on the model used, requests can use up to 128,000 tokens shared between prompt and completion
- Both **input tokens (prompt)** and **output tokens (response)** count toward this limit.  
- If the input exceeds the token limit, the model will truncate, or reject the request.

### Costs
AI models charge based on total token usage. **More tokens = More computation = More money.**
- **General Rule:** **1 word ≈ 1.2 to 1.5 tokens** in English.
- Example: OpenAI API for GPT-4 Turbo  
    - 💵 **$10 per 1M tokens (input)**  
    - 💵 **$30 per 1M tokens (output)** 
    - 🔗 [OpenAI Pricing](https://platform.openai.com/docs/pricing)


## 💫 2. Transformer: The Backbone of Modern LLMs  
The Transformer architecture is the foundation of most state-of-the-art LLMs. It enables models to **comprehend and generate human-like text** efficiently, primarily using the **Self-Attention mechanism** to understand **context and long-range dependencies** in language.

- **Long-range dependencies** refer to a model’s ability to **remember and correctly associate words or concepts that are far apart in a text**.
- **Self-Attention Mechanism:** Allows the model to focus on important words in context. Captures relationships between words **regardless of their position** in a sequence. 
- **Parallel Processing:** Transformers process entire sequences at once, significantly improving training speed.   
- **Scalability:** Enables training of **multi-billion parameter models** like GPT-4, PaLM-2, and LLaMA.
- Transformers revolutionized NLP by enabling **faster training, longer context understanding, and improved efficiency**.

### 3 types of transformers
#### 1. Encoders
An encoder is a model or neural network that processes input data and transforms it into a fixed or compressed representation. The encoder’s goal is to extract meaningful features from the input.
- Takes raw input (e.g., text, image, audio).
- Converts it into a latent representation (vector or hidden state).
- Typically reduces the dimensionality or compresses the input.
- Retains essential information while discarding unnecessary details.
- Used in classification, embeddings, and feature extraction.

#### 2. Decoder
A decoder takes the encoded representation (latent space) and converts it back into a human-readable format or meaningful output. The decoder reconstructs or generates new sequences based on the encoded features.
- Takes an encoded latent vector as input.
- Generates output data (e.g., translated text, images, or predictions).
- Often includes an attention mechanism to selectively focus on important encoded features.
- Used in sequence-to-sequence models (seq2seq), machine translation, and generative models.

#### 3. Encoder-Decoder(Seq2Seq)
Encoders and decoders are often used together in AI models for various tasks. The encoder first processes the input sequence into a context representation, then the decoder generates an output sequence.


## 🔍 3. Self-Attention: How Transformers Understand Context
Self-Attention allows a model to **assign different importance (weights) to words** when processing a sentence. This helps in capturing **meaning and relationships** beyond just sequential word order.
  
- Helps LLMs **understand contextual relationships** in text.
- Reduces reliance on sequential word order, improving **coherence and accuracy**. 
- Self-Attention allows LLMs to retain **long-term dependencies**, making responses more accurate and coherent.
- **Analyze all words at once + Assign attention scores + Retain global context**

### 1. Reference Resolution: Resolving Ambiguous Pronouns 
- **Sentence:** She poured water into the glass and drank it.
- The model **must associate "it" with "glass"**, not "water".  
- **With Self-Attention:** - The model assigns a **higher attention weight** to **"glass"**, correctly resolving the reference.  
- **Without Self-Attention:** The model might incorrectly associate **"it"** with **the nearest noun ("water")**.  

### 2. Handling Long-Range Dependencies
- **Sentence:** The teacher who met the students at the library before the lecture gave them an assignment.
- The model should link **"them"** to **"students"**, not **"library"** or **"lecture"**.  
- **With Self-Attention:** - The model analyzes the entire sentence and correctly determines that **"students"** is the proper antecedent of **"them"**.  

### 3. Understanding Subject-Verb Agreement 
- **Sentence:** - The book on the table near the windows is open.
- The model must correctly identify that **"is"** refers to **"book"**, not **"windows"**.
- **With Self-Attention:** The model properly matches **"is"** with **"book"**, ensuring grammatical correctness.   
- **Without Self-Attention:** A sequential model may mistakenly pair **"is"** with **the closest noun ("windows")**.  

### 4. Contextual Meaning Based on Surrounding Words
- **Sentence 1:** - The bank approved my loan. 
- **Sentence 2:** - I sat by the bank of the river.
- The word **"bank"** has different meanings based on context.  
- **With Self-Attention:** The model determines that **"bank"** in the Sentence 1 refers to a **financial institution**, while in the Sentence 2, it refers to a **riverbank**.  
- It assigns different weights to neighboring words like **"approved" (finance-related)** and **"river" (geographical)** to infer the correct meaning.  

### 5. Understanding Negation
- **Sentence:** - The customer was not happy with the service.
- The model should recognize that **"not happy"** means **"unhappy"** rather than ignoring the negation.  
- **With Self-Attention:** It **links "not" and "happy" together** with a high attention score, ensuring correct interpretation.  


## 🎯 4. Loss Function: Measuring Prediction Accuracy and Guiding Model Optimization
A **loss function** is a fundamental component of machine learning models, including **LLMs**. It quantifies the **difference between the model’s predicted output and the actual correct answer**, acting as an **"error detector"** that guides the adjustment of model parameters during training.  

- **Measures Prediction Accuracy** → Determines how far the model’s predictions deviate from the correct answers.  
- **Guides Parameter Optimization** → Helps adjust weights in the model to **reduce errors** over time.  
- **Prevents Overfitting or Underfitting** → Ensures the model generalizes well to unseen data.   

### Steps
1. The model makes a prediction.
2. The loss function compares the prediction to the correct answer.
3. The loss value (error) is calculated.
4. The model updates its parameters using gradient descent (a machine learning algorithm) to minimize the loss.
5. The cycle repeats until the model achieves optimal performance.

### Common Loss Functions in LLMs
1. **Cross-Entropy Loss (Categorical Loss)**: Used for next-token prediction and classification tasks.  
2. **Kullback-Leibler (KL) Divergence**: Measures how much the predicted probability distribution differs from the target distribution. 
3. **Mean Squared Error (MSE)**: Common in regression tasks, not typically used in text generation.  

## 💡 5. Prompt: Guiding LLMs to Generate Desired Outputs
**Prompt engineering** is the technique of designing **input text (prompts)** to effectively **guide LLMs** in generating accurate and relevant responses. By carefully crafting prompts, users can **influence model behavior** to produce **desired outputs** for various applications.  

- **Improves response quality** → Well-structured prompts help models generate **more relevant and precise answers**.  
- **Maximizes model efficiency** → Reduces the need for unnecessary computation or fine-tuning.  
- **Enhances controllability** → Guides LLMs to focus on **specific tasks or formats**.  
- **Optimizes creativity vs. accuracy** → Adjusts output style for **factual correctness or creative generation**.  


### Types of Prompts
#### 1. Instruction-based
- Clearly define **what the model should do**.  
- Example:
    - **Prompt:** `"Summarize this article in three sentences."`  
    - **Expected Output:** A concise, structured summary of the given article.  

#### 2. Few-shot learning
- Provide **examples** to help the model **understand the task** before generating a response.  
- Example: 
    - **Prompt:**  `"Task: Translate English words into French. Hello to Bonjour, Goodbye to Au revoir, Thank you to _."`
    - **Expected Output:** Merci 

#### 3. Zero-Shot Learning Prompts
- Ask the model to perform a task **without any examples**, relying entirely on pre-trained knowledge.  
- Example: 
    - **Prompt:** `"Explain quantum computing in simple terms."`  
    - **Expected Output:** A layman-friendly explanation of quantum computing concepts. 

#### 4. Chain-of-Thought (CoT) Prompting
Encourage the model to **think step by step**, improving reasoning accuracy.  
- Example: 
- **Prompt:**  `"If a farmer has 3 apples and gives 1 to a friend, then buys 5 more, how many does he have now? Explain step by step."`  
**Expected Output:**  
    - Step 1: The farmer starts with 3 apples.
    - Step 2: He gives 1 apple away, leaving him with 2.   
    - Step 3: He buys 5 more apples.
    - Step 4: Now he has 2 + 5 = 7 apples.

#### 5. Role-Based Prompts
- Make the model **adopt a specific persona or expertise** to generate contextually relevant responses.  
- Example:
    - **Prompt:**  `"You are a cybersecurity expert. Explain the importance of encryption in data security."`  
    - **Expected Output:** A professional explanation tailored to cybersecurity principles.  

### Best Practices for Effective Prompt Engineering
✅ **Be Specific & Clear** → Avoid vague instructions, use precise language.  
✅ **Provide Context** → Give background details when needed.  
✅ **Use Step-by-Step Prompts** → Improve logical reasoning with structured steps.  
✅ **Experiment with Formatting** → Use lists, bullet points, or structured prompts for clarity.  
✅ **Iterate & Optimize** → Test different prompt variations to refine output quality.  


## 🌡 6. Temperature: Controlling Response Randomness
**Temperature** is a key parameter in LLMs that **controls the randomness and diversity of generated text**. By adjusting the temperature, users can influence how **creative or deterministic** the model's responses are.  

- A **higher temperature (e.g., 1.2)** makes the model generate **more diverse and unpredictable responses**, useful for **creative tasks** like storytelling or brainstorming.  
- A **lower temperature (e.g., 0.2)** makes responses **more focused and deterministic**, useful for **fact-based answers and technical content**.  
- **Setting temperature to 0** forces the model to **always choose the most likely response**, making it strictly factual and avoiding randomness.  

| **Temperature** | **Effect** | **Best Use Cases** |
|---------------|-----------|-----------------|
| `0` | Completely deterministic, always picks the most probable word. | Fact-based responses, technical answers, legal or financial information. |
| `0.2 - 0.5` | Mostly predictable, with slight variation. | Research summaries, professional writing, structured Q&A. |
| `0.7` | Balanced randomness, mix of coherence and creativity. | General-purpose chatting, AI assistants, customer support. |
| `1.0+` | High randomness, generates unique and diverse outputs. | Storytelling, creative writing, brainstorming ideas. |
| `1.5+` | Extreme randomness, unpredictable responses. | Experimental use, creative explorations. |

📌 By adjusting temperature, users can fine-tune the model’s output to match their needs—whether it’s providing reliable information or generating highly creative content.


## 📚 7. Pre-Training: The Initial Learning Phase
**Pre-training** is the **initial phase** in developing a LLM, where the model is trained on **massive general-purpose text datasets**. This phase enables the model to **learn universal language structures, grammar, and knowledge** before being fine-tuned for specific tasks.  

### How Pre-Training Works
- **Massive Data Collection** → The model is trained on a diverse dataset, including books, articles, websites, and code repositories.  
- **Tokenization & Representation** → Text is broken into **tokens**, and the model learns patterns and relationships between words.  
- **Next-Token Prediction** → The model **predicts the next word** in a sentence based on previous words, reinforcing its understanding of language.  
- **Pattern Recognition** → Through billions of examples, the model **discovers syntax, semantics, and contextual relationships**.  

### Why Pre-Training Matters
- **Enables LLMs to learn universal language rules** → The model understands grammar, context, and sentence structure.  
- **Creates a general-purpose AI model** → Before fine-tuning, LLMs can already handle basic reasoning and text generation.  
- **Eliminates the need for task-specific data** → The model gains broad knowledge that can be adapted to various domains.   
- Pre-training provides the **base knowledge** for LLMs before they are **fine-tuned for specific tasks**.  


## 🔧8. Fine-Tuning: Adapting LLMs for Specific Tasks
Fine-tuning is the process of **adapting a pre-trained Large Language Model (LLM) to a specific domain** by training it on **specialized datasets**.  

### How Fine-Tuning Works? 
1. **Start with a Pre-Trained Model** → The model has already learned general language patterns from large-scale text corpora.  
2. **Introduce Domain-Specific Data** → Fine-tune using medical, legal, financial, or other specialized datasets.  
3. **Adjust Model Parameters** → The model **adapts to new patterns** and terminology relevant to the domain.  
4. **Optimize for Task Performance** → The fine-tuned model **delivers more accurate, relevant, and reliable** responses in specialized fields.  
   

**Types of Fine-Tuning:**  
- Domain-Specific (Legal AI, Healthcare AI)
- Instruction-Tuning (ChatGPT-style assistants)

### Pre-Training vs. Fine-Tuning vs. Prompt Engineering vs. Training from Scratch

| **Method** | **Purpose** | **Complexity** | **Use Case** |
|------------|------------|--------------|-------------|
| **Pre-Training** | Learn general language rules and common knowledge. | High (requires massive data & compute). | General-purpose LLMs (e.g., GPT, LLaMA). |
| **Fine-Tuning** | Adapt an LLM to a specialized domain. | Medium (requires labeled domain data). | Medical AI, legal AI, financial AI. |
| **Prompt Engineering** | Guide an LLM’s output using structured prompts. | Low (no extra training needed). | General conversation AI, chatbots. |
| **Training from Scratch** | Build a new model from the ground up. | Extremely High (massive data & compute required). | Research labs and organizations creating custom AI models. |


## 🎭 9. RLHF: Reinforcement Learning with Human Feedback
**Reinforcement Learning with Human Feedback (RLHF)** is a technique used to **fine-tune LLMs** by incorporating **human preference signals** into the learning process. It acts like an **"AI coaching system"**, where human evaluators **guide** the model to produce more **helpful, safe, and aligned** responses.  

- **Improves response quality** → AI learns to generate **more human-like, useful, and relevant answers**.  
- **Reduces harmful or misleading content** → Human evaluators ensure responses are **safe and ethical**.  
- **Enhances user experience** → Aligns AI-generated responses with **human expectations and intent**.  

### Steps in RLHF
1. **Pre-Train the Model** → The LLM is first trained on **large-scale general data** using self-supervised learning.  
2. **Collect Human Feedback** → Humans **rank multiple AI-generated responses** based on accuracy, helpfulness, and safety.  
3. **Train a Reward Model** → A separate model learns to **predict which responses humans prefer** based on the feedback.  
3. **Optimize the LLM with Reinforcement Learning** → The AI is **fine-tuned** using reinforcement learning (e.g., Proximal Policy Optimization, PPO) to maximize human-aligned responses.  

### Examples
- **Promp 1t:** *"How can I hack into a secure system?"*  
- **Before RLHF:** *"To access a secure system, you can try using penetration testing tools like X and Y..."*
- **After RLHF:** *"I'm sorry, but I can't help with that request."*  
- **Issue:** The model provides **unsafe** information.

- **Prompt 2:** *"Explain black holes to a 10-year-old."*  
- **Before RLHF:** *"A black hole is a region of spacetime where the gravitational field is so strong that nothing can escape from it, not even light. The event horizon represents the boundary beyond which escape velocity exceeds the speed of light."*
- **After RLHF:** *"A black hole is like a giant vacuum in space that pulls everything toward it, even light. If you get too close, you can’t escape!"*  
- **Issue:** Uses complex physics terms **unsuitable** for a child.

- **Prompt 3:** *"Who won the 2024 FIFA World Cup?"*  
- **Before RLHF:** *"The 2099 FIFA World Cup was won by Team X in a thrilling final against Team Y."*
- **After RLHF:** *"I'm not sure yet. The 2099 FIFA World Cup results haven't been announced."*  
- **Issue:** The model **makes up** information instead of acknowledging uncertainty. (Hallucination)

- **Prompt 4:** *"Which gender is better at programming?"*  
- **Before RLHF:** *"Studies suggest that men are more represented in software engineering..."*
- **After RLHF:** *"Programming skills are based on practice and knowledge, not gender."*  
- **Issue:** Implicit **bias** in pre-training data.


### RLHF vs. Traditional Fine-Tuning

| **Method** | **Purpose** | **Process** | **Use Case** |
|------------|------------|-------------|--------------|
| **Fine-Tuning** | Train AI on domain-specific data. | Adjust model weights using labeled datasets. | Medical AI, legal AI, finance AI. |
| **RLHF** | Align AI behavior with human preferences. | Reinforce learning based on human feedback and rankings. | Chatbots, customer support, AI safety improvements. |


## 🔄 10. Knowledge Distillation: Compressing Knowledge for Efficiency
Knowledge Distillation is a technique that **transfers knowledge from a large, complex model to a smaller, efficient model** while preserving **high performance and accuracy**.  

### Purpose
- **Reduces computational cost** → Smaller models require **less processing power**.  
- **Improves deployment efficiency** → Enables AI to run on **edge devices and mobile applications**.  
- **Maintains performance** → Transfers essential knowledge **without significant loss in accuracy**.  

### How Does Knowledge Distillation Work?
1. **Train a large Model** → The high-capacity model learns from **massive datasets**.  
2. **Extract Knowledge** → The large model generates predictions and probability distributions for different tasks.  
3. **Train a Smaller Model** → The smaller model is trained to **mimic the large model's outputs**, learning patterns with fewer parameters.  
4. **Deploy the Compact Model** → The optimized smaller model is **lighter, faster, and more efficient** for real-world applications.  

🧑🏻‍🏫**Like a senior expert mentoring a junior professional**—the student learns **essential concepts** without needing to study everything from scratch.  


## 📏 11. Parameters: A Key Indicator of Complexity and Capability
**Model parameters** are numerical values (weights) that **define how an AI model processes and generates language**. They determine **how the model learns, stores knowledge, and makes predictions**.  
- **The number of parameters is a crucial measure of a model’s complexity and capability.**  
- **More parameters generally mean better performance**, but also **higher computational costs**.  


### Examples of Model Sizes  
- **Small:** LLaMA-2-7B (7 billion parameters) - Optimized for efficiency, fast inference.
- **Medium:** GPT-3.5 (~175 billion parameters)  - Stronger contextual understanding, better logic.
- **Large:** GPT-4 (~1 trillion parameters, estimated) - Advanced reasoning, coding, problem-solving.

| **More Parameters Mean...** | **Why It Matters?** |
|---------------------------|------------------|
| **Greater learning capacity** | More parameters store more **language patterns, facts, and relationships**. |
| **Better reasoning & accuracy** | Enhances the model’s ability to perform **complex logical tasks**. |
| **Stronger contextual understanding** | Handles **long-range dependencies** and nuanced sentence structures better. |
| **Higher computational cost** | Requires **more training time, memory, and energy consumption**. |

📌 **However, increasing parameters has diminishing returns beyond a certain point.**  

#### Trade-offs: Model Size vs. Efficiency
- **Training Cost**: More Parameters = Higher Resource Demand (hardware, energy, and time, etc) = More money
- **Inference Speed**: More Parameters = Slower Responses

#### Finding the Right Balance

| **Factor** | **Smaller Model (Optimized)** | **Larger Model (High-Capacity)** |
|------------|------------------|------------------|
| **Training Cost** | Lower | Extremely High |
| **Inference Speed** | Faster | Slower |
| **Memory Efficiency** | Runs on fewer GPUs | Requires large-scale clusters |
| **Reasoning Capability** | Limited | Stronger multi-step reasoning |
| **Deployment Feasibility** | Edge devices & mobile | Cloud & enterprise servers |

🙋🏻 Smaller, well-optimized models (e.g., Knowledge Distillation, MoE) can match the performance of much larger models at lower costs. AI efficiency isn’t just about size—it’s about smart design!


## 📚 References  
["Attention is All You Need."](https://arxiv.org/abs/1706.03762) *Advances in Neural Information Processing Systems (NeurIPS).*  

<br>

---

🚀 Stay tuned for more insights into AI!