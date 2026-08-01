# Arquitetura

## Visão geral

Este assistente é construído como um conjunto de serviços fracamente acoplados, orquestrados pelo n8n, com toda a inferência de IA acontecendo localmente através do Ollama.

## Componentes

### 1. Gateway de WhatsApp (Evolution API)

Atua como um cliente não-oficial de WhatsApp, conectado via pareamento por QR code a um número de telefone dedicado. Encaminha mensagens recebidas para o n8n via webhook, e expõe um endpoint para enviar mensagens de volta.

Escolhido em vez da API oficial WhatsApp Cloud pela simplicidade de configuração em um contexto pessoal/de baixo volume — não exige verificação de conta Business. Pode ser substituído pela API oficial da Meta sem alterar o restante da stack, já que o n8n interage apenas com um contrato de webhook + endpoint HTTP.

### 2. Orquestração (n8n)

Dois tipos principais de workflow:

- **Workflow de ingestão**: monitora uma pasta compartilhada em busca de novos documentos, faz o chunking, gera embeddings via Ollama e armazena os vetores no Qdrant.
- **Workflow do agente**: recebe mensagens de WhatsApp via webhook, repassa para um nó AI Agent configurado com:
  - Ollama como modelo de chat
  - Qdrant como ferramenta de recuperação (RAG)
  - Resposta enviada de volta através da Evolution API

### 3. Inferência de LLM (Ollama)

Executa tanto o modelo de chat quanto o modelo de embeddings. Implantado nativamente no host (em vez de em Docker) para melhor desempenho em configurações apenas com CPU — evita o overhead de acesso containerizado aos recursos do host.

A escolha do modelo é ajustada para inferência em CPU com 16GB de RAM: modelos pequenos e quantizados (faixa de 3B a 7B) em vez de modelos maiores que exigiriam GPU ou causariam troca de memória (swap) excessiva.

### 4. Banco vetorial (Qdrant)

Armazena os embeddings da base de conhecimento pessoal para retrieval-augmented generation. Mantém as respostas fundamentadas nos documentos-fonte reais, em vez de depender puramente da memória do modelo.

## Fluxo de dados

1. Documentos são colocados na pasta compartilhada e indexados uma vez através do workflow de ingestão.
2. Uma mensagem de WhatsApp chega → webhook da Evolution API → n8n.
3. O nó AI Agent consulta o Qdrant em busca de contexto relevante, depois consulta o Ollama com o prompt aumentado.
4. A resposta é enviada de volta ao WhatsApp através da Evolution API.

## Notas sobre hardware

Testado em: Intel i5 (8ª geração), 16GB DDR4, sem GPU dedicada. Toda a inferência roda em CPU. O tamanho do modelo e a janela de contexto foram deliberadamente mantidos pequenos para manter a latência razoável sem GPU.
