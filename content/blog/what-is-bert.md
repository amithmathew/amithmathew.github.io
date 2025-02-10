---
title: "What Is Bert"
date: "2024-09-09T10:51:47-04:00"
description: "An optional description for SEO. If not provided, an automatically created summary will be used."
draft: true
tags: ["ai","llm","rag"]
---

# Rough Notes
- One of the OG language models from October 2018 from Google AI
- [Self-supervised learning](https://en.wikipedia.org/wiki/Self-supervised_learning)!
    - Learns to represent text as a sequence of vectors.
    - TODO Has a Transformer - Encoder architecture

- Trained by masked token prediction and next sentance prediction
- Learns contextual, latent representations of tokens in their context, similar to ELMo and GPT-2
- It's an evolutionary step over ELMo

- BERT is an "encoder-only" transformer architecture.
- Has 4 modules
    - Tokenizer: converts english text into a sequence of integers
    - Embedding: Converts a sequnce of tokens into an array of real-valued vectors representing the tokens. It represents the conversion of discrete token types into a lower dimensional Euclidean space.
    - Encoder: A stack of transformer blocks with self-attention but wihtout causal masking
    - Task Head: This module converst the final representation vectors into one-hot encoded tokens again by predicting probability distribution over the token types. It can be viewed as a single decoder, decoding the latent representation into token types, or as an "un-embedding layer".


# References


