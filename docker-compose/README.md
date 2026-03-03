# PlainID Edge Docker Compose

> [!NOTE]
> This is an example recipe. It is intended as a starting point and must
> be customized before use in any real environment.

## Overview

For more details about PlainID Edge component and how to configure it, refer to
the documentation: <https://docs.plainid.io/docs/plainid-edge>.

The recipe brings up multiple services on a shared Docker network:

| Service                       | Port           |
| ----------------------------- | -------------- |
| `vector-db-classifier-engine` | `8000`         |
| `mcp-gateway`                 | `5235`         |
| `enrichment-agent`            | `8082 → 8080`  |
| `discovery-agent`             | `8080`, `8081` |

## How It Works

### Directory Layout

```text
docker-compose/
├── docker-compose.yaml
└── config/
    ├── discovery-agent/
    │   ├── .env             # environment variables (including secrets)
    │   └── config.yaml      # application config
    ├── enrichment-agent/
    │   ├── .env
    │   └── config.yaml
    ├── mcp-gateway/
    │   ├── .env
    │   └── config.yaml
    └── vector-db-classifier-engine/
        ├── .env
        └── config.yaml
```

### Config Files

Each service has a `config.yaml` mounted read-only into the container at `/app/config/`.
The application reads this file on startup and substitutes `${VAR}` placeholders
with values from the process environment.

### Environment Variables

Each service directory contains a `.env` file loaded by Compose via `env_file`.
It supplies configuration and secret values to the container at runtime.

Values in `.env` are substituted into `config.yaml` by the application at startup.

### Secrets

**In this example**, secret values (API keys, client credentials) are stored as
plain entries in the `.env` file alongside regular configuration.
Replace every `REPLACE_WITH_*` placeholder with the real value before starting
the stack.

## Deploying

### Prerequisites

- Access to PlainID's Docker Hub repositories (image pull credentials).
- A Pinecone API key.
- **(Optional)** An OpenAI API key for `vector-db-classifier-engine`.

### Starting the Stack

1. Edit each `config/<service>/.env` and `config/<service>/config.yaml` file:

   **`config/vector-db-classifier-engine/.env`**

   ```text
   OPENAI_API_KEY=<your-openai-api-key>
   ```

   **`config/enrichment-agent/.env`**

   ```text
   JWT_VALIDATION_ENABLED=false
   CLASSIFICATION_SERVICE_URL=http://vector-db-classifier-engine:8000
   PINECONE_API_KEY=<your-pinecone-api-key>
   ```

   **`config/mcp-gateway/.env`**

   ```text
   PLAINID_CLIENT_ID=<your-client-id>
   PLAINID_CLIENT_SECRET=<your-client-secret>
   PLAINID_RUNTIME_HOST=<plainid-runtime-hostname>
   PLAINID_RUNTIME_PORT=8080
   ```

   **`config/mcp-gateway/config.yaml`** — configure the `mcpServers` block with
   the MCP endpoints relevant to your deployment. The provided example uses
   [Context7](https://mcp.context7.com) as a placeholder.

   **`config/discovery-agent/.env`**

   ```text
   PLAINID_API_URL=https://<plainid-api-endpoint>
   PLAINID_DISCOVERY_URL=https://<plainid-lambda-url>
   POP_ID=MCP
   ENVIRONMENT_ID=<your-environment-id>
   PLAINID_MCP_GATEWAY_URL=http://mcp-gateway:5235
   PLAINID_CLIENT_ID=<your-client-id>
   PLAINID_CLIENT_SECRET=<your-client-secret>
   ```

2. Start the stack (in background):

   ```bash
   docker compose up -d
   ```

3. Check logs:

   ```bash
   docker compose logs -f
   ```

### Stopping the Stack

Running compose stack can be brought down using:

```bash
docker compose down
```

It could also be deleted:

```bash
docker compose rm
```
