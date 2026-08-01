# Architecture

## Overview

This assistant is built as a set of loosely-coupled services orchestrated by n8n, with all AI inference happening locally through Ollama.

## Components

### 1. WhatsApp Gateway (Evolution API)

Acts as an unofficial WhatsApp client, connected via QR code pairing to a dedicated phone number. Forwards incoming messages to n8n via webhook, and exposes an endpoint to send messages back.

Chosen over the official WhatsApp Cloud API for simplicity of setup in a personal/low-volume context — no Business verification required. Can be swapped for the official Meta Cloud API without changing the rest of the stack, since n8n only interacts with a webhook + HTTP endpoint contract.

### 2. Orchestration (n8n)

Two main workflow types:

- **Ingestion workflow**: watches a shared folder for new documents, chunks them, generates embeddings via Ollama, and stores vectors in Qdrant.
- **Agent workflow**: receives incoming WhatsApp messages via webhook, passes them to an AI Agent node configured with:
  - Ollama as the chat model
  - Qdrant as a retrieval tool (RAG)
  - Response sent back through the Evolution API

### 3. LLM Inference (Ollama)

Runs both the chat model and the embedding model. Deployed natively on the host (rather than in Docker) for better performance on CPU-only setups — avoids the overhead of containerized access to host resources.

Model selection is tuned for CPU-only inference on 16GB RAM: small quantized models (3B–7B range) rather than larger models that would require a GPU or induce heavy swapping.

### 4. Vector Store (Qdrant)

Stores embeddings of the personal knowledge base for retrieval-augmented generation. Keeps responses grounded in the actual source documents rather than relying purely on model recall.

## Data Flow

1. Documents are dropped into the shared folder and indexed once via the ingestion workflow.
2. A WhatsApp message arrives → Evolution API webhook → n8n.
3. The AI Agent node queries Qdrant for relevant context, then queries Ollama with the augmented prompt.
4. The response is sent back to WhatsApp via the Evolution API.

## Hardware Notes

Tested on: Intel i5 (8th gen), 16GB DDR4, no dedicated GPU. All inference runs on CPU. Model size and context window were deliberately kept small to keep latency reasonable without a GPU.
