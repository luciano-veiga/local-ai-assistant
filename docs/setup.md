# Guia de instalação

## 1. Instale o Ollama (nativo, máquina host)

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.2
ollama pull nomic-embed-text
```

Rodar o Ollama nativamente (fora do Docker) é recomendado para configurações apenas com CPU — evita overhead de virtualização e dá ao modelo acesso direto aos recursos do host.

## 2. Suba a stack Docker

```bash
cp .env.example .env
# edite o .env com suas próprias credenciais e IPs
docker compose up -d
```

Isso inicia:
- **Qdrant** em `localhost:6333`
- **Evolution API** em `localhost:8080`
- **n8n** em `localhost:5678`

## 3. Conecte o WhatsApp

1. Abra `http://localhost:8080` e crie uma instância na Evolution API
2. Escaneie o QR code com seu número dedicado de WhatsApp
3. Configure a URL do webhook apontando para o endpoint de webhook da sua instância n8n

## 4. Importe os workflows do n8n

Na interface do n8n (`http://localhost:5678`):

1. Importe `n8n/workflows/ingestion.json`
2. Importe `n8n/workflows/agent.json`
3. Configure as credenciais do Ollama (URL base) e do Qdrant (URL + nome da coleção)
4. Ative os dois workflows

## 5. Indexe seus documentos

Coloque arquivos na pasta `./shared` (montada no n8n como `/data/shared`) e dispare manualmente o workflow de ingestão, ou configure um monitor de pasta para indexação automática.

## 6. Teste

Envie uma mensagem para seu número dedicado de WhatsApp. O agente deve responder usando tanto o LLM local quanto qualquer contexto relevante recuperado dos seus documentos indexados.

## Dicas de desempenho (apenas CPU)

- Prefira modelos quantizados na faixa de 3B a 7B (ex: quantização Q4_K_M)
- Mantenha a janela de contexto (`ctx-size`) modesta para reduzir a pressão de memória
- Monitore o uso de recursos com `docker stats` junto com `htop` para o processo nativo do Ollama
- Se as respostas estiverem muito lentas, tente um modelo menor antes de aumentar o contexto
