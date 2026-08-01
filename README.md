# Assistente de IA Local — RAG + WhatsApp + n8n

Um assistente de IA pessoal totalmente self-hosted, construído com **LLMs locais**, **Retrieval-Augmented Generation (RAG)** e **automação de workflows** — sem depender de nenhuma API paga na nuvem. O assistente roda inteiramente em hardware local (funciona sem GPU) e é acessível via WhatsApp.

![Arquitetura](assets/architecture.png)

## ✨ Visão geral

Este projeto combina uma plataforma de automação low-code self-hosted com um runtime de LLM local e um banco de dados vetorial para criar um assistente pessoal capaz de:

- Responder perguntas usando um modelo de linguagem rodando localmente (nenhum dado sai da sua máquina)
- Recuperar contexto relevante de uma base de conhecimento pessoal (PDFs, notas, documentos) via RAG
- Ser acessado de forma conversacional pelo WhatsApp
- Executar tarefas agendadas/automatizadas (lembretes, resumos, buscas)

Toda a stack roda em hardware de consumo — sem necessidade de GPU, testado em uma configuração apenas com CPU e 16GB de RAM.

## 🧱 Stack

| Componente | Função |
|---|---|
| [n8n](https://n8n.io/) | Orquestração de workflows e lógica do agente (low-code) |
| [Ollama](https://ollama.com/) | Inferência de LLM local (chat + embeddings) |
| [Qdrant](https://qdrant.tech/) | Banco de dados vetorial para RAG (recuperação de documentos) |
| [Evolution API](https://github.com/EvolutionAPI/evolution-api) | Gateway não-oficial de WhatsApp (baseado em webhook) |
| Docker Compose | Orquestração dos containers |

## 🏗️ Arquitetura

```
WhatsApp (número dedicado)
      │
      ▼
Evolution API (gateway WhatsApp, webhook)
      │
      ▼
n8n Webhook Trigger
      │
      ▼
n8n AI Agent Node
      ├── Ollama Chat Model (LLM local)
      └── Qdrant Vector Store (busca RAG)
                ▲
                │
        Ollama Embeddings
                ▲
                │
   Workflow de Ingestão de Documentos (PDFs, notas, docs)
```

## 🚀 Como começar

### Pré-requisitos

- Docker e Docker Compose
- ~16GB de RAM (inferência apenas em CPU é suportada; GPU não é necessária)
- Um número de telefone dedicado para a integração com WhatsApp (recomendado, para isolar do uso pessoal)

### 1. Clone e configure

```bash
git clone https://github.com/<seu-usuario>/local-ai-assistant.git
cd local-ai-assistant
cp .env.example .env
# edite o .env com seus próprios valores
```

### 2. Suba a stack

```bash
docker compose up -d
```

Isso sobe o n8n, o Qdrant e a Evolution API. Espera-se que o Ollama rode separadamente (nativamente ou em seu próprio container) — veja [docs/setup.md](docs/setup.md).

### 3. Baixe os modelos

```bash
ollama pull llama3.2        # modelo de chat
ollama pull nomic-embed-text  # modelo de embeddings
```

### 4. Conecte o WhatsApp

1. Abra a instância da Evolution API e escaneie o QR code com o número dedicado
2. Importe os workflows de `n8n/workflows/` para sua instância do n8n
3. Configure a URL do webhook na Evolution API apontando para o webhook do n8n

### 5. Indexe sua base de conhecimento

Rode o workflow de ingestão (`n8n/workflows/ingestion.json`) para indexar seus documentos no Qdrant. Coloque os arquivos na pasta compartilhada e dispare o workflow.

## 📂 Estrutura do repositório

```
.
├── docker-compose.yml       # Qdrant + Evolution API + n8n
├── .env.example             # Template de variáveis de ambiente
├── n8n/
│   └── workflows/           # Workflows do n8n exportados em JSON
├── docs/
│   ├── architecture.md      # Notas detalhadas de arquitetura
│   └── setup.md             # Passo a passo completo de instalação
└── assets/                  # Diagramas e capturas de tela
```

## 💡 Por que local-first?

- **Privacidade**: nenhum dado pessoal ou conversa é enviado para APIs de terceiros
- **Custo**: zero custo recorrente de API — apenas o chip SIM (opcional) para o WhatsApp
- **Aprendizado**: experiência prática com orquestração de LLM, busca vetorial e automação de workflows

## 🛣️ Roadmap

- [ ] Suporte a múltiplas bases de conhecimento (coleções por tópico no Qdrant)
- [ ] Adicionar memória/persistência de conversa
- [ ] Explorar modelos quantizados mais leves para inferência mais rápida em CPU
- [ ] Adicionar workflows de resumo agendado

## 📜 Licença

MIT — veja [LICENSE](LICENSE)
