# Development Port Convention

A local development port convention for individuals, small companies, and big organizations.

It covers popular frameworks, databases, messaging, observability, platform, and AI services, and it follows a few rules so that a port number tells you what is running on it.

---

# 1. Principles

1. **Two kinds of service, two rules.**
   - **Shared services** run one instance per machine: databases, brokers, observability, platform, AI runtimes, dev utilities. They keep their **vendor default port**. Connection strings, Docker images, and docs all assume it.
   - **Project processes** belong to one project: web apps, APIs, dev servers, notebooks, Storybook. They get a port from the **project block** for their stack.
2. **Set ports explicitly.** Every project sets its port in its own config (`PORT`, framework config, or `--port`). Framework defaults such as 3000, 5173, or 8000 are fallbacks for a project that runs alone, not part of the convention.
3. **One owner per port per host.** Two services never share a port on one machine. When two shared services want the same default, the one used most keeps it, and the other uses the fallback block (section 6).
4. **Never use ports below 1024** for applications (they need admin rights on Linux and macOS), and **never use 49152-65535** (the OS ephemeral range for outgoing connections).
5. **Containers publish only what the host needs.** A container port is internal to its Docker network. Only the host-facing port follows this convention.
6. **Port ranges belong to a stack, not a team.** The stack determines the range, so a port number tells you the language or runtime.
7. **Debugger ports are reserved.** Section 2.1 lists them. Projects and shared services never use them.

---

# 2. Port Map

| Range | Use |
|-------------|-----------------------------------------------------------|
| 1024-11999 | Shared services at vendor defaults (section 5) |
| 12000-12999 | JavaScript / TypeScript project blocks |
| 13000-13999 | Python project blocks |
| 14000-14999 | Go, Rust, C / C++ project blocks |
| 15000-15999 | JVM project blocks (Java, Kotlin, Scala, Groovy) |
| 16000-16999 | Ruby, PHP, Elixir / Erlang, Dart / Flutter project blocks |
| 17000-17999 | .NET project blocks (C#, F#, VB) |
| 18000-18999 | Mobile and desktop dev servers (Metro, Ionic, Tauri) |
| 19000-19999 | Reserved for future stacks (19530 Milvus and 19999 Netdata are used by shared services) |
| 20000-20999 | Team-shared dev services (mock servers, gateways, stubs) |
| 21000-24999 | Reserved |
| 25000-25999 | Fallback for shared-service conflicts (section 6) |
| 26000-49151 | Ad hoc use, OS-assigned, other tools |
| 49152-65535 | OS ephemeral range. Do not use. |

## 2.1 Reserved Debugger Ports

Debuggers attach to their own ports. These values are reserved: no project block, shared service, or fallback port may use them, even when nothing is running.

| Debugger | Default | Notes |
|----------|--------:|-------|
| Node.js / Bun / Deno (inspector) | 9229 | Extra processes: 9230, 9231, ... |
| Chrome / Edge (remote debugging) | 9222 | |
| PHP (Xdebug) | 9003 | |
| Python (debugpy) | 5678 | |
| Java / Kotlin / Scala (JDWP) | 5005 | |
| Go (Delve) | 2345 | |
| Ruby (rdbg) | 12345 | Lies inside the JavaScript block range; project tools index 5 of slot 3 must not be used |
| .NET (vsdbg) | Random | Set a fixed port when you need one |

---

# 3. Project Blocks

## 3.1 Format

```
Port = Stack base + (Project x 100) + Role + Index
```

| Part | Values | Meaning |
|------|--------|---------|
| Stack base | 12000, 13000, 14000, ... | From the stack table in section 2 |
| Project | 0-9 | Slot within the stack for one project on one machine |
| Role | +00, +10, +20, +30, +40 | Kind of process (table below) |
| Index | 0-9 | Second and later instances of a role. 0 is the primary one |

| Role offset | Role | Examples |
|-------------|------|----------|
| +00 | Web frontend / UI | Main app, admin panel, dashboard, docs site |
| +10 | Gateway / BFF | Backend-for-frontend, API gateway, SSR server |
| +20 | API | REST, GraphQL, gRPC, ML inference API |
| +30 | Internal services | Auth, WebSocket, workers, job runners, schedulers |
| +40 | Tools | Storybook, Jupyter, MkDocs, MLflow, Flower, mocks |
| +50-+99 | Reserved | |

**Mapping rule:** for the same project and index, the API is the frontend port + 20, and the internal services are the frontend port + 30. For example, if the web app is `12000`, the API is `12020` and auth is `12030`.

**Slot ownership:** a project slot (0-9) only has to be unique on one developer's machine, so two developers can both use slot 0 for different projects. Record the slot in the repo's `ports.md` (or the team wiki) so teammates agree. Do not run two projects with the same slot on one machine.

## 3.2 Examples

Project Alpha (Angular, slot 0):

| Process | Port |
|---------|-----:|
| Web app | 12000 |
| Admin panel | 12001 |
| API | 12020 |
| Auth | 12030 |
| WebSocket | 12031 |
| Storybook | 12040 |

Project Beta (React, slot 1, with a Python ML service at slot 0):

| Process | Port |
|---------|-----:|
| Web app (React, slot 1) | 12100 |
| API (Node, slot 1) | 12120 |
| Auth (slot 1) | 12130 |
| WebSocket (slot 1) | 12131 |
| Storybook (slot 1) | 12140 |
| ML API (FastAPI, Python slot 0) | 13020 |
| Jupyter (Python slot 0) | 13040 |

Both projects share the shared services at their defaults:

| Shared service | Port |
|----------------|-----:|
| PostgreSQL | 5432 |
| Redis | 6379 |
| Grafana | 3000 |
| Prometheus | 9090 |

---

# 4. Application Stacks (Project Blocks)

The default ports in the tables are for reference only. In a project block the project sets its own port.

## 4.1 JavaScript / TypeScript: 12000-12999

### Web frontends and static sites

| Framework | Default |
|-----------|--------:|
| React (Create React App) | 3000 |
| Next.js | 3000 |
| Remix | 3000 |
| Nuxt | 3000 |
| Vue CLI | 8080 |
| Vite (React, Vue, Svelte, Solid, Qwik) | 5173 |
| SvelteKit | 5173 |
| SolidStart | 3000 |
| Angular | 4200 |
| Ember | 4200 |
| Astro | 4321 |
| Gatsby | 8000 |
| Docusaurus | 3000 |
| VitePress | 5173 |
| Eleventy | 8080 |
| Parcel | 1234 |

### Backends and full-stack servers

| Framework / runtime | Default |
|---------------------|--------:|
| Express | 3000 |
| NestJS | 3000 |
| Fastify | 3000 |
| Koa | 3000 |
| Hono (Node, Bun) | 3000 |
| Hono on Cloudflare Workers (`wrangler dev`) | 8787 |
| Bun server | 3000 |
| Deno, Deno Fresh | 8000 |
| AdonisJS | 3333 |
| Strapi | 1337 |
| Payload CMS | 3000 |
| Directus | 8055 |
| Socket.IO | 3000 |
| FeathersJS | 3030 |
| Meteor | 3000 |

### Tools

| Tool | Default |
|------|--------:|
| Storybook | 6006 |

## 4.2 Python: 13000-13999

### Web frameworks and APIs

| Framework | Default |
|-----------|--------:|
| Django | 8000 |
| FastAPI (uvicorn) | 8000 |
| Starlette (uvicorn) | 8000 |
| Litestar | 8000 |
| Sanic | 8000 |
| Flask | 5000 |
| Quart | 5000 |
| Tornado | 8888 |
| aiohttp | 8080 |
| Chainlit | 8000 |
| Reflex (backend) | 8000 |
| Reflex (frontend) | 3000 |

### Data and ML UIs

| Tool | Default |
|------|--------:|
| Streamlit | 8501 |
| Gradio | 7860 |
| Dash (Plotly) | 8050 |
| Jupyter Notebook / JupyterLab | 8888 |
| MLflow tracking server | 5000 |
| TensorBoard | 6006 |
| Ray dashboard | 8265 |
| Label Studio | 8080 |
| Prefect UI | 4200 |
| Dagster UI | 3000 |
| Airflow webserver | 8080 |

### Docs and tasks

| Tool | Default |
|------|--------:|
| MkDocs | 8000 |
| Celery Flower | 5555 |

## 4.3 Go, Rust, C / C++: 14000-14999

| Framework / runtime | Default |
|---------------------|--------:|
| net/http (examples) | 8080 |
| Gin | 8080 |
| Echo | 1323 |
| Fiber | 3000 |
| Chi | 3000 (examples) |
| Buffalo | 3000 |
| Beego | 8080 |
| Warp | 3030 |
| Hugo (dev server) | 1313 |
| Actix Web | 8080 |
| Axum | 3000 (examples) |
| Rocket | 8000 |

## 4.4 JVM: 15000-15999

| Framework | Default |
|-----------|--------:|
| Spring Boot | 8080 |
| Quarkus | 8080 |
| Micronaut | 8080 |
| Ktor | 8080 |
| Vert.x | 8080 |
| Helidon | 8080 |
| Dropwizard (application / admin) | 8080 / 8081 |
| Jetty | 8080 |
| Tomcat | 8080 |
| Grails | 8080 |
| Akka HTTP | 8080 (examples) |
| Javalin | 7000 |
| Play Framework (Scala) | 9000 |

## 4.5 Ruby, PHP, Elixir / Erlang, Dart: 16000-16999

| Framework | Default |
|-----------|--------:|
| Rails | 3000 |
| Sinatra | 4567 |
| Hanami | 2300 |
| Jekyll | 4000 |
| Laravel (`artisan serve`) | 8000 |
| Symfony (`server:start`) | 8000 |
| PHP built-in server | 8000 |
| Phoenix | 4000 |
| Dart `shelf` (examples) | 8080 |
| Flutter web | Random by default. Set with `--web-port`. |

**Note:** port 16010 (HBase UI) is a shared service in this range. Do not use the project slot 0 gateway (`16010`) on a machine that runs HBase.

## 4.6 .NET: 17000-17999

| Framework | Default |
|-----------|--------:|
| ASP.NET Core (HTTP) | 5000 |
| ASP.NET Core (HTTPS) | 5001 |

**Note:** `launchSettings.json` generates random ports in the 5xxx and 7xxx ranges. Replace them with the project block value.

## 4.7 Mobile and desktop dev servers: 18000-18999

| Tool | Default |
|------|--------:|
| React Native / Expo Metro | 8081 |
| Ionic (`ionic serve`) | 8100 |
| Tauri (dev server) | 1420 |

## 4.8 Team-shared dev services: 20000-20999

One mock server, gateway, or stub per team per stack. Record the port in the team's `ports.md`.

| Tool | Default | Notes |
|------|--------:|-------|
| WireMock | 8080 | Team-shared mock |
| Mockoon | 3000 | Team-shared mock |

---

# 5. Shared Services (Vendor Defaults)

Run one instance of each on a machine. Use the vendor default. For a second instance on the same machine, run it in a separate Docker network instead of changing the port.

## 5.1 Databases

| Database | Port | Notes |
|----------|-----:|-------|
| PostgreSQL | 5432 | Also TimescaleDB, pgvector, and most Postgres-compatible tools |
| YugabyteDB (YSQL) | 5433 | |
| MySQL | 3306 | |
| MariaDB | 3306 | Same as MySQL. Run one. |
| SQL Server | 1433 | |
| Oracle | 1521 | |
| Db2 | 50000 | |
| Firebird | 3050 | |
| CockroachDB (SQL) | 26257 | UI is 8080 (see section 6) |
| TiDB | 4000 | Status 10080 |
| MongoDB | 27017 | |
| Couchbase | 8091 | UI. Data port 11210. |
| CouchDB | 5984 | |
| Cassandra | 9042 | ScyllaDB uses the same port |
| DynamoDB Local | 8000 | |
| Firestore emulator | 8080 | |
| Redis | 6379 | Valkey, KeyDB, and Dragonfly also use 6379 |
| Memcached | 11211 | |
| etcd | 2379 | Peer port 2380 |
| ClickHouse (HTTP) | 8123 | |
| ClickHouse (native) | 9000 | |
| InfluxDB | 8086 | |
| QuestDB (HTTP) | 9000 | PostgreSQL wire 8812 |
| Neo4j (HTTP) | 7474 | Bolt 7687 |
| ArangoDB | 8529 | |
| Elasticsearch | 9200 | |
| OpenSearch | 9200 | |
| Solr | 8983 | |
| Meilisearch | 7700 | |
| Typesense | 8108 | |
| Qdrant (HTTP) | 6333 | gRPC 6334 |
| Weaviate | 8080 | gRPC 50051 |
| Milvus | 19530 | |
| Chroma | 8000 | |
| Trino / Presto | 8080 | |
| Apache Druid (router) | 8888 | |
| Apache Pinot (controller) | 9000 | |
| StarRocks (FE HTTP) | 8030 | MySQL protocol 9030 |

## 5.2 Messaging and Streaming

| Service | Port | Notes |
|---------|-----:|-------|
| Apache Kafka | 9092 | Redpanda uses the same port |
| Redpanda admin | 9644 | |
| Kafka Connect | 8083 | |
| Schema Registry | 8081 | |
| ZooKeeper | 2181 | |
| RabbitMQ | 5672 | |
| RabbitMQ management UI | 15672 | |
| ActiveMQ (OpenWire) | 61616 | Web console 8161. Artemis uses the same ports. |
| NATS | 4222 | Monitoring 8222 |
| MQTT (Mosquitto, EMQX) | 1883 | TLS 8883 |
| Apache Pulsar (broker) | 6650 | Admin 8080 |
| Temporal (server) | 7233 | UI 8233 |

## 5.3 Observability

| Service | Port | Notes |
|---------|-----:|-------|
| Prometheus | 9090 | |
| Alertmanager | 9093 | |
| Pushgateway | 9091 | |
| Node Exporter | 9100 | |
| cAdvisor | 8080 | |
| Grafana | 3000 | |
| Loki | 3100 | |
| Tempo | 3200 | |
| Mimir | 9009 | |
| Pyroscope | 4040 | |
| Jaeger UI | 16686 | |
| OpenTelemetry Collector (OTLP gRPC) | 4317 | HTTP 4318. Jaeger OTLP uses the same ports, so run one collector. |
| Zipkin | 9411 | |
| Kibana | 5601 | |
| Logstash (Beats input) | 5044 | |
| Fluentd / Fluent Bit (forward) | 24224 | |
| Vector (API) | 8686 | |
| Datadog Agent (APM) | 8126 | DogStatsD 8125 |
| Sentry (self-hosted) | 9000 | |
| Netdata | 19999 | |
| Uptime Kuma | 3001 | |

## 5.4 Platform and DevOps

| Service | Port | Notes |
|---------|-----:|-------|
| Kubernetes API | 6443 | |
| kubelet | 10250 | |
| Docker Registry | 5000 | |
| Harbor | 80 / 443 | |
| Nexus Repository | 8081 | |
| JFrog Artifactory | 8081 | |
| Verdaccio (npm) | 4873 | |
| devpi (PyPI) | 3141 | |
| Consul | 8500 | DNS 8600 |
| Vault | 8200 | |
| Nomad | 4646 | |
| HashiCorp Boundary | 9200 | Controller API |
| Keycloak | 8080 | HTTPS 8443 |
| Dex | 5556 | |
| Authelia | 9091 | |
| Traefik | 80 / 443 | Dashboard 8080 |
| Nginx | 80 / 443 | |
| Caddy | 80 / 443 | Admin 2019 |
| HAProxy (stats) | 8404 | |
| Kong (proxy) | 8000 | Admin 8001, TLS 8443 |
| Envoy | 10000 | Admin 9901 |
| Portainer | 9443 | |
| Gitea | 3000 | SSH 2222 |
| GitLab | 80 / 443 | SSH 22 |
| Jenkins | 8080 | |
| Argo CD | 8080 | |
| Concourse | 8080 | |
| SonarQube | 9000 | |
| Camunda | 8080 | |
| Apache Superset | 8088 | |
| Metabase | 3000 | |
| Redash | 5000 | |

## 5.5 Storage, Mail, Testing, and DB Admin

| Service | Port | Notes |
|---------|-----:|-------|
| MinIO (S3 API) | 9000 | Console 9001 |
| Mailpit | 8025 | Web UI. SMTP 1025. |
| MailHog | 8025 | Web UI. SMTP 1025. Run one of Mailpit or MailHog. |
| LocalStack | 4566 | All AWS services |
| Selenium Grid | 4444 | |
| Adminer | 8080 | |
| phpMyAdmin | 8080 | |
| pgAdmin | 5050 | |
| Mongo Express | 8081 | |
| Redis Commander | 8081 | |
| RedisInsight | 5540 | |

## 5.6 AI and LLM Runtimes

| Service | Port | Notes |
|---------|-----:|-------|
| Ollama | 11434 | |
| llama.cpp server (`llama-server`) | 8080 | |
| llama-cpp-python server | 8000 | OpenAI-compatible |
| LocalAI | 8080 | |
| LM Studio | 1234 | |
| Jan | 1337 | Local API server |
| GPT4All (API server) | 4891 | |
| KoboldCpp | 5001 | |
| text-generation-webui (oobabooga) | 7860 | |
| vLLM | 8000 | OpenAI-compatible server |
| Hugging Face TGI | 80 | Container port. Often mapped to 8080. |
| Hugging Face Text Embeddings Inference | 80 | Container port. Often mapped to 8080. |
| Infinity (embeddings) | 7997 | |
| Xinference | 9997 | |
| NVIDIA Triton | 8000 | gRPC 8001, metrics 8002 |
| NVIDIA NIM | 8000 | |
| Ray Serve | 8000 | |
| BentoML | 3000 | |
| LiteLLM proxy | 4000 | OpenAI-compatible gateway |
| Open WebUI | 3000 | Container port 8080 |
| AnythingLLM | 3001 | |
| Flowise | 3000 | |
| Langflow | 7860 | |
| Dify (web) | 3000 | Container port 80 |
| Dify (API) | 5001 | |
| Langfuse | 3000 | LLM observability |
| n8n | 5678 | Reserved debugger port (section 2.1). Set a fallback port (section 6). |
| ComfyUI | 8188 | |
| Stable Diffusion WebUI (AUTOMATIC1111) | 7860 | |

---

# 6. Conflicts and Fallback

Some shared services share a vendor default. On one machine, the service used most keeps the port. Each other service takes a free port from the fallback block **25000-25999** and records it in the machine's `ports.md`.

| Default | Claimed by (shared services) | Rule |
|--------:|------------------------------|------|
| 3000 | Grafana, Gitea, Metabase, Open WebUI, Flowise, Dify (web), Langfuse, BentoML | One keeps 3000. Others use the fallback. |
| 3001 | AnythingLLM, Uptime Kuma | One keeps 3001. Other uses the fallback. |
| 4000 | TiDB, LiteLLM proxy | One keeps 4000. Other uses the fallback. |
| 5001 | Dify (API), KoboldCpp | One keeps 5001. Other uses the fallback. |
| 5678 | n8n | Reserved debugger port (section 2.1). n8n always uses the fallback. |
| 7860 | text-generation-webui, Stable Diffusion WebUI, Langflow | One keeps 7860. Others use the fallback. |
| 8000 | vLLM, Kong proxy, DynamoDB Local, Chroma, Triton, NVIDIA NIM, Ray Serve, llama-cpp-python | One keeps 8000. Others use the fallback. |
| 8080 | Traefik, Jenkins, Keycloak, Argo CD, Concourse, Camunda, Airflow, Kafka UI, Adminer, cAdvisor, Pulsar admin, Firestore emulator, Weaviate, Trino, CockroachDB UI, llama.cpp, LocalAI, WireMock | Most overloaded default. Most machines run one. Others use the fallback. |
| 8081 | Nexus, Schema Registry, Mongo Express, Redis Commander, Artifactory | One keeps 8081. Others use the fallback. |
| 9000 | SonarQube, MinIO API, Sentry, QuestDB, Pinot | One keeps 9000. Others use the fallback. MinIO and SonarQube are the most common pair. |
| 9090 | Prometheus, Invoke (Stable Diffusion UI) | One keeps 9090. Other uses the fallback. |
| 9091 | Pushgateway, Authelia | One keeps 9091. Other uses the fallback. |
| 9092 | Kafka, Redpanda | One keeps 9092. Other uses the fallback. |
| 9200 | Elasticsearch, OpenSearch, Boundary | One keeps 9200. Others use the fallback. |
| 3306 | MySQL, MariaDB | Run one. |
| 5000 | Docker Registry, Redash | One keeps 5000. Other uses the fallback. |
| 8025 | Mailpit, MailHog | Run one. |

**Fallback examples:** SonarQube on a machine that also runs MinIO: MinIO keeps `9000` and SonarQube uses `25000`. Portainer on a machine that also runs SonarQube: `25001`.

---

# 7. Host Rules

## 7.1 Checking and reserving ports

- Check a port before you use it: `Get-NetTCPConnection -State Listen -LocalPort 12000` (PowerShell) or `lsof -i :12000` (macOS and Linux).
- **Windows reserved ranges:** Hyper-V and WSL reserve TCP blocks. A port inside one fails with "access denied" or "address in use," even when nothing is running. Check them with `netsh interface ipv4 show excludedportrange protocol=tcp`. If a convention port falls inside a reserved block, use another index in the same project block or move the shared service to the fallback block.

## 7.2 Containers (Docker Compose)

- Services talk to each other over the Compose network by service name and container port (`postgres:5432`). These do not use host ports.
- Publish a host port only when the developer's host needs it, and use the convention's port: `"5432:5432"`, `"12000:3000"` (container port 3000 published as project port 12000).

## 7.3 Kubernetes and cloud

- Developers do not choose ports in clusters. Services, Ingress, and service mesh routing handle them. The convention applies to local development only.

## 7.4 Shared dev environments

- In a shared environment, address services by DNS name, not by port. Use the convention only for what each developer runs locally.

## 7.5 Organizations

- A team owns its stacks' project slots on its machines. Keep `ports.md` at the repo root.
- The organization owns the stack ranges in section 2. Changing one is a breaking change for everyone, so change it through the platform team.
- Team-shared services (section 4.8) are registered in the team's `ports.md`. Do not claim a port that another team has listed.

---

# 8. Rule of Thumb

When you start a new service:

1. **Shared service** (database, broker, observability, platform, AI runtime): use the vendor default from section 5. If it's taken on this machine, use the fallback from section 6.
2. **Project process** (app, API, tool): pick a free project slot (0-9) for your stack, then take the port for the role:
   - Web frontend: `Stack base + slot x 100 + 00 + index`
   - Gateway: `... + 10 + index`
   - API: `... + 20 + index`
   - Internal service: `... + 30 + index`
   - Tool: `... + 40 + index`
3. Record the slot and ports in the repo's `ports.md`.
