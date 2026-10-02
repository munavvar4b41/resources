# AI System

Local AI stack with Ollama, Qdrant, SearXNG, and Open WebUI.

## How to use

1. Create `searxng/settings.yml` from the settings file below. Replace `change-this-secret` with a long random string.
2. Copy the compose file below to the project root as `docker-compose.yml`.
3. Replace every `{project}` with a short name, such as `acme`. Container names cannot contain `{` or `}`.
4. On a machine without an NVIDIA GPU, delete the `deploy` block from the `ollama` service. That block reserves NVIDIA devices.
5. Run `docker compose up -d`.
6. Pull a model, then open Open WebUI and create the admin account:

```bash
docker compose exec ollama ollama pull llama3.2
```

The settings file uses `:Z` because only SearXNG mounts it. Create the file before the first start. If the path is missing, Compose creates a directory and SearXNG cannot read it.

JSON output is enabled in SearXNG, and the rate limiter is off, so Open WebUI can query it on the compose network. `server.bind_address` is `0.0.0.0` so the other containers can connect.

Open WebUI stores admin settings in its data volume. On a new volume it reads the environment below. After the first save in the admin panel, that saved configuration is what the next start uses.

## URLs

- Open WebUI: http://localhost:3100
- Ollama: http://localhost:11434
- Qdrant: http://localhost:6333
- SearXNG: http://localhost:8081

## Docker compose file

```yaml
services:
  ollama:
    image: docker.io/ollama/ollama:latest
    container_name: "{project}-ollama"
    restart: unless-stopped
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

  qdrant:
    image: docker.io/qdrant/qdrant:latest
    container_name: "{project}-qdrant"
    restart: unless-stopped
    ports:
      - "6333:6333"
    volumes:
      - qdrant:/qdrant/storage

  searxng:
    image: docker.io/searxng/searxng:latest
    container_name: "{project}-searxng"
    restart: unless-stopped
    ports:
      - "8081:8080"
    volumes:
      - ./searxng/settings.yml:/etc/searxng/settings.yml:Z

  openwebui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: "{project}-openwebui"
    restart: unless-stopped
    ports:
      - "3100:8080"
    environment:
      OLLAMA_BASE_URL: http://ollama:11434
      VECTOR_DB: qdrant
      QDRANT_URI: http://qdrant:6333
      ENABLE_WEB_SEARCH: "True"
      WEB_SEARCH_ENGINE: searxng
      WEB_SEARCH_RESULT_COUNT: "5"
      SEARXNG_QUERY_URL: "http://searxng:8080/search?q=<query>"
    volumes:
      - openwebui:/app/backend/data
    depends_on:
      - ollama
      - qdrant
      - searxng

volumes:
  ollama:
  qdrant:
  openwebui:
```

Keep `<query>` in `SEARXNG_QUERY_URL` as written. Open WebUI replaces that token when it searches.

## searxng/settings.yml

```yaml
use_default_settings: true

server:
  bind_address: "0.0.0.0"
  secret_key: "change-this-secret"
  limiter: false

search:
  formats:
    - html
    - json
```
