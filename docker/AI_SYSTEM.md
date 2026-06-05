# AI System

This is a docker compose file for a local AI development environment. It includes ollama, qdrant, and openwebui.

## How to use

1. Copy the below docker compose code to your project directory and name it `docker-compose.yml`
2. Run `docker compose up -d`
3. Access the applications at the following URLs:
   - Ollama: http://localhost:11434
   - Qdrant: http://localhost:6333
   - OpenWebUI: http://localhost:3100

## Docker compose file

```docker
services:

  ollama:
    image: ollama/ollama
    container_name: ollama
    environment:
      OLLAMA_KEEP_ALIVE: 1m
      OLLAMA_NUM_PARALLEL: 1
      OLLAMA_MAX_LOADED_MODELS: 1
    ports:
      - "11434:11434"
    volumes:
      - ollama:/root/.ollama
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    restart: unless-stopped

  qdrant:
    image: qdrant/qdrant
    container_name: qdrant
    ports:
      - "6333:6333"
    volumes:
      - qdrant:/qdrant/storage
    restart: unless-stopped

  searxng:
    image: searxng/searxng
    container_name: searxng
    ports:
      - "8081:8080"
    restart: unless-stopped
    volumes:
      - ./searxng/settings.yml:/etc/searxng/settings.yml:Z

  openwebui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: openwebui
    ports:
      - "3100:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
      - ENABLE_RAG_WEB_SEARCH=true
      - RAG_WEB_SEARCH_ENGINE=searxng
      - SEARXNG_QUERY_URL=http://searxng:8080/search?q=<query>
    depends_on:
      - ollama
      - searxng
    restart: unless-stopped
    volumes:
      - openwebui:/app/backend/data

volumes:
  ollama:
  qdrant:
  openwebui:
```
