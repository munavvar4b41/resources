# AI Apps

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

  openwebui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports:
      - "3100:8080"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    volumes:
      - openwebui:/app/backend/data
    depends_on:
      - ollama
    restart: unless-stopped

volumes:
  ollama:
  qdrant:
  openwebui:
```