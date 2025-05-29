---
title: RAG
slug: rag
description: Simple RAG implementation using Langchain and Weaviate
date: "2024-10-27"
tags: ["rag", "python", "langchain", "weaviate", "vectorstore"]
subjects: ["python", "rag"]
showTaxonomies: true
---

# Building a Lightweight RAG Library with LangChain and Weaviate

As someone who frequently works with Large Language Models (LLMs), I found myself repeatedly writing similar boilerplate code for Retrieval Augmented Generation (RAG) applications: setting up document retrieval, implementing QA chains, validating results, and generating structured outputs. To avoid rewriting nearly identical code, I decided to create a [lightweight library](https://github.com/RomainEconomics/rag) to streamline this process.

## Why Another RAG Library?

While there are several excellent RAG implementations out there, I wanted something that was:

- Lightweight and focused
- Built on top of libraries I frequently use (LangChain and Weaviate)
- Flexible enough to handle different types of chains and validation strategies
- Easy to customize for my specific use cases

## Key Features

The library provides several chain types out of the box, ranging from basic question-answering to comprehensive validation chains. Here are some highlights:

### Multiple Chain Types

- **Basic QA**: Simple retrieval and question-answering
- **Structured Output**: Get responses in a structured format using Pydantic models
- **Relevance Checking**: Validate that retrieved documents are actually relevant
- **Full Validation**: Complete pipeline with relevance and hallucination checks
- **Image QA**: Support for image-based questions and structured outputs

### Flexible LLM Configuration

Since for different tasks, we may want to use different model, we have the ability to use different models for different components. For example, you might want to use a smaller, faster model for relevance checking while using a more powerful model for the final answer generation:

```python
llm_config = LLMConfig(
    default_llm=ChatOpenAI(model="gpt-4o-mini"),
    component_llms={
        ChainComponent.EXTRACTION: ChatOpenAI(model="gpt-4o-mini"),
        ChainComponent.RELEVANCE: ChatOpenAI(model="gpt-4o-mini"),
        ChainComponent.VALIDATION: ChatOpenAI(model="gpt-4o-mini"),
        ChainComponent.IMAGE: ChatOpenAI(model="gpt-4o"),
    }
)
```

## Getting Started

While the library isn't yet published on PyPI, you can clone it from GitHub and install it using `uv sync`. Here's a quick example of how to use it:

```python
from rag.factory import ChainManager, LLMConfig

# Initialize chain manager with your config
manager = ChainManager(
    vectorstore=vectorstore,
    llm_config=llm_config,
)

# Run a basic QA chain
result = manager.run_chain(
    chain_type=ChainType.BASIC_QA,
    question="Your question here"
)
```

## Current Limitations and Future Plans

The library currently only supports Weaviate as the vector store, but I have plans to expand its capabilities:

- Implement more advanced document ingestion strategies:
  - Parent/child relationships for better context management
  - Automatic summarization for improved search
- Add support for additional retrieval strategies (XGBoost, Cohere)
- Integrate reranking chains (Colbert, Cohere)
- Support more vector stores (PG Vector, Faiss)

## Final Thoughts

This library is primarily a personal tool to avoid again and again similar code. While it may never make it to PyPI, I hope it can serve as a useful reference or starting point for others building RAG systems with LangChain and Weaviate.

The code is available [on GitHub](https://github.com/RomainEconomics/rag) and more examples are available in the following Jupiter Notebook [here](https://github.com/RomainEconomics/rag/blob/master/exemples.ipynb)

Feel free to check out the code on GitHub, and contributions are always welcome!
