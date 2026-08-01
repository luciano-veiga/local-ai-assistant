# Setup Guide

## 1. Install Ollama (native, host machine)

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.2
ollama pull nomic-embed-text
```

Running Ollama natively (not in Docker) is recommended for CPU-only setups — it avoids virtualization overhead and gives the model direct access to host resources.

## 2. Start the Docker stack

```bash
cp .env.example .env
# edit .env with your own credentials and IPs
docker compose up -d
```

This starts:
- **Qdrant** on `localhost:6333`
- **Evolution API** on `localhost:8080`
- **n8n** on `localhost:5678`

## 3. Connect WhatsApp

1. Open `http://localhost:8080` and create an instance in Evolution API
2. Scan the QR code with your dedicated WhatsApp number
3. Set the webhook URL to your n8n instance's webhook endpoint

## 4. Import n8n workflows

In the n8n UI (`http://localhost:5678`):

1. Import `n8n/workflows/ingestion.json`
2. Import `n8n/workflows/agent.json`
3. Configure credentials for Ollama (base URL) and Qdrant (URL + collection name)
4. Activate both workflows

## 5. Ingest documents

Place files in the `./shared` folder (mounted into n8n as `/data/shared`) and manually trigger the ingestion workflow, or configure a folder watcher for automatic indexing.

## 6. Test

Send a message to your dedicated WhatsApp number. The agent should respond using both the local LLM and any relevant context retrieved from your indexed documents.

## Performance Tips (CPU-only)

- Prefer quantized models in the 3B–7B range (e.g., Q4_K_M quantization)
- Keep the context window (`ctx-size`) modest to reduce memory pressure
- Monitor resource usage with `docker stats` alongside `htop` for the native Ollama process
- If responses are too slow, try a smaller model before increasing context size
