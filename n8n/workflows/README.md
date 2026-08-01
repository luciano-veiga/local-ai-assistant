# Workflows do n8n

Exporte seus workflows reais pela interface do n8n (menu ⋮ → Download) e coloque aqui como arquivos JSON. Arquivos esperados:

- `ingestion.json` — monitora/lê a pasta compartilhada, faz o chunking dos documentos, gera embeddings e insere os vetores no Qdrant.
- `agent.json` — recebe o webhook do WhatsApp, executa o nó AI Agent (Ollama + recuperação via Qdrant) e envia a resposta de volta através da Evolution API.

**Antes de commitar:** revise os arquivos JSON exportados em busca de credenciais embutidas, chaves de API ou caminhos de arquivo pessoais — exports do n8n às vezes incluem isso. Higienize antes de subir para um repositório público.
