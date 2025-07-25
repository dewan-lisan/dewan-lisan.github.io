---
title: Generative AI Core Concepts
date: 2025-07-23 12:00:00 +0200
categories: [Generative AI, Core Concepts]
tags: [genai]     # TAG names should always be lowercase
---

### Goals
- Deep technical understanding of LLMs for AI enthusiasts, engineers, and data scientists
- Explore the power of LLMs as a general-purpose technology
- The generative AI project lifecycle, including model training, instruction tuning, fine-tuning, and deployment

## Prompt Engineering Concepts
Interaction with LLMs uses natural language prompts (text instructions) rather than formal code.
- **Context Window:** The model’s memory for processing prompts, typically holding a few thousand words (varies by model).
- **Prompt:** Input text (e.g., a question like “Where is Ganymede?”).
- **Completion:** The model’s output, including the prompt and generated text (e.g., “Ganymede is a moon of Jupiter…”).
- **Inference:** The process of generating text from a prompt.

### Types of Inrefence
#### Zero-Shot Inference
No examples are provided, just the task and input. Works well with large models e.g. GPT-3.5, GPT-4

**Example:**  
- **Prompt:** “Classify the sentiment of this review: I really loved this movie!”  
- **Completion:** “Positive”

#### One-Shot Inference
One example is included in the prompt to guide the model. It helps smaller models to understand what to do.

**Example:**  
“Classify the sentiment:  
Review: I loved this movie → Sentiment: Positive  
Review: [your input] → Sentiment: [??]”

#### Few-Shot Inference
Multiple examples are provided. Increases the model's ability to generalize and understand varied cases.

**Example:**  
Include both positive and negative review examples to help the model learn both classes.

### Conext Window Limit
You can only include a limited number of examples based on model capacity.
- GPT-4o (OpenAI): Up to 128,000 tokens.
- Claude 3.5 Sonnet (Anthropic): Up to 200,000 tokens.
- Llama 3.1 (Meta AI): Up to 128,000 tokens.
- Gemini 1.5 Pro (Google): Up to 2 million tokens.

Biggest token size doesn't necessarily mean the biggest model. It also depends on parameter count, architecture, training data etc.

## Transformer Architecture Building Blocks

### Encoder and Decoder
- **Encoder:** Processes the input sequence (used in tasks like translation, classification).  
- **Decoder:** Generates output sequences (used in text generation or translation).  

Encoder only model: BERT.  
Encoder-decoder model: BART, T5.  
In GPT-style LLMs, only the decoder is used.  

![Transformer Architecture](/assets/img/TransformerArch.png){: width="550" height="780" }
_Transformer Architecture depicted in the **Attention Is All You Need** paper_

### Tokenization
- Converts input text (words) into numbers that the model can process.
- Token can be full words or sub-words.
- Same tokenizer must be used for the training and inference.
**Example:**
- The sentence **"The teacher gave the book to the student"** might be tokenized as: [101, 2025, 4321, 2047, 1012]

### Embedding Layer
- Converts token IDs into high-dimentional dense vectors that represents meaning and context.
- Words with similar meanings will have similar vectors and will be close to each other.
**Example:** Token 101 → [0.1, 0.7, -0.3, ..., 0.05] (a 512-dimensional vector)

### Positional Encoding
- Add information about the order of the words since the transformer process the words in parallel.
**Example:** “The teacher gave the book” vs. “The book gave the teacher” Without positional encoding, both might be treated similarly.

### Self-Attention
It helps the model to determine which words to focus on in the input sentence.
**How it works:**
- Each word looks at every other word and calculates attention score
- These scores determine how much "attention" a word should pay to others
- This is not limited to nearby words — every word can attend to all others.

**Example**
In "The teacher gave the book to the studen", the word book might strongly attend to **teacher** and **student**

### Multi-Head Attention
- Learn multiple types of relations between words in parallel.
- Instead of one attention layer, the transformer uses multiple (e.g., 12–100) attention heads.
- Each head is randomly initialized and learns different features during training.
  
**Example:**  
- Head 1 might focus on subject-object relations (e.g., “teacher” → “student”).
- Head 2 might learn tense patterns or rhyme structures.

### Feed-Forward Network
- ???

### Softmax Layer
Converts these scores into probabilities and predicts the next work (or classification output)
**Example:**   
Output for the next token might be: "book": 0.2, "pen": 0.05, "student": 0.7, ...   
→ Most likely next token = "student"


## Generative Configuration

Training parameters are learned during model training. Inference parameters are used at runtime to control how the model generates output.

### Key Inference Parameters
#### Max New Tokens
Sets a limit on the maximum number of tokens the model can generate. It’s a cap, not a guarantee — generation might stop earlier if the model outputs an end-of-sequence token.

#### Decoding Strategies
- **Greedy Decoding:** Picks the most probable word every time, cases repeating word in the output.
- **Random Sampling:** Picks the next word at random, but may produce nonsensical output if too random.

#### Samping Controls
- Top-k Sampling: Selects top k most likely tokens. Example: k = 3 -> choose from 3 highest-probability words.
- Top-p Sampling: Selects from the smallest set of tokens whose cumulative probability ≤ p. Example: p = 0.3 → use the most probable tokens that together make up 30% probability mass.

#### Temperature
Its a **Softmax output** settings, controls randomness by scaling the probability distribution.
- Higher temp -> more creative
- Lower temp -> more focused.
- Default softmax behavior = 1

