# n8n Workflows

Export your actual workflows from the n8n UI (⋮ menu → Download) and place them here as JSON files. Expected files:

- `ingestion.json` — watches/reads the shared folder, chunks documents, generates embeddings, and upserts vectors into Qdrant.
- `agent.json` — receives the WhatsApp webhook, runs the AI Agent node (Ollama + Qdrant retrieval), and sends the response back via the Evolution API.

**Before committing:** double-check exported JSON files for any embedded credentials, API keys, or personal file paths — n8n exports can sometimes include these. Sanitize before pushing to a public repo.
