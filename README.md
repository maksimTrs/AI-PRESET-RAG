# AI-PRESET: Complete Local AI Stack

A Docker Compose setup for running a complete local AI stack with **n8n**, **Ollama (LLM)**, **PostgreSQL with pgvector**, **Docling (Document Parser)**, and **Open WebUI**. Includes automatic function import for seamless n8n-Open WebUI integration.

> ⚠️ **IMPORTANT**: The default LLM models require approximately **7 GB of disk space** (~5 GB for Qwen3.5-9B + ~1.2 GB for BGE-M3). Optional heavier models may require up to **50+ GB**. Ensure you have sufficient storage before starting the stack.

## 🚀 Quick Start

### Prerequisites

- **Docker Desktop** (Windows/Mac) or **Docker Engine + Compose** (Linux)
- **For GPU**: NVIDIA drivers

### Setup

1. **Create environment file**:

```bash
cp .env.example .env
# Edit .env with your passwords
```

2. **Start the stack**:

```bash
# CPU only
docker compose --profile cpu up -d

# With NVIDIA GPU
docker compose --profile gpu-nvidia up -d
```

## 📋 Project Structure

```
AI-PRESET/
├── .env.example                           # Environment template (copy to .env)
├── .env                                   # Your local environment variables (not in git)
├── docker-compose.yml                     # Main configuration
├── n8n_pipe_function.py                   # Pipe function source (human-readable)
├── open-webui-functions-for-import.json   # Same function wrapped for Open WebUI auto-import
├── scripts/
│   └── open-webui-entrypoint.sh          # Auto-creates admin user & imports functions on startup
├── n8n/backup/                           # n8n workflows (auto-imported on startup)
└── shared/                               # Shared volume between n8n and other services
```

> **Note**: `n8n_pipe_function.py` and `open-webui-functions-for-import.json` contain the same function code. The `.py` file is for reading/editing, the `.json` is what gets imported into Open WebUI at startup. If you edit the `.py`, you must manually update the JSON as well.

## 🌐 Access Services

| Service        | URL                                     | Purpose                |
| -------------- | --------------------------------------- | ---------------------- |
| **Open WebUI** | [localhost:3000](http://localhost:3000) | ChatGPT-like interface |
| **n8n**        | [localhost:5678](http://localhost:5678) | Workflow automation    |
| **Docling**    | [localhost:5001](http://localhost:5001) | Document parsing & OCR |
| **pgAdmin**    | [localhost:5050](http://localhost:5050) | Database management    |
| **Ollama API** | localhost:11434                         | LLM API endpoint       |

## ⚙️ Configuration

### Open WebUI Setup

Admin user and n8n pipe function are **auto-created on first startup** via `scripts/open-webui-entrypoint.sh`.

1. Login with admin credentials from `.env` file
2. **Configure Ollama**: Settings → Models → Add endpoint: `http://ollama:11434`
3. **Configure n8n Function**: Admin → Functions → Find imported function → Configure Valves:
   - **N8N Url**: `http://n8n:5678/webhook-test/[your-webhook-url]`
   - **N8N Bearer Token**: (if required)
   - **Input Field**: `chatInput`
   - **Response Field**: `output`

### n8n Setup

1. Create owner account
2. **Add Ollama credentials**:
   - **Chat Model**: Base URL `http://ollama:11434/v1`, Model `qwen3.5:9b`
   - **Embeddings**: Base URL `http://ollama:11434`, Model `bge-m3`

### Docling Setup (Document Processing)

**Basic HTTP Request Configuration for n8n:**

- **Method**: POST
- **URL**: `http://docling-serve:5001/v1alpha/convert/file`
- **Headers**: `accept: application/json`
- **Body Type**: Form-Data
- **Parameters**:
  - `files`: [Binary File] - Your document
  - `options`: [Form Data] - `{"include_images": true, "do_ocr": true}`

**Supported formats**: PDF, DOCX, XLSX, PPTX, Images, HTML, Markdown
**Features**: Advanced PDF parsing, table extraction, OCR, layout analysis

### pgAdmin Setup

1. Login with credentials from `.env`
2. **Connect to PostgreSQL**:
   - Host: `postgres`, Port: `5432`
   - Username/Password: from `.env` file

## 🤖 Available Models

**Default Models** (automatically downloaded, ~7 GB total):

- `qwen3.5:9b` - Natively multimodal (text + vision + tool calling + thinking), beats previous-gen Qwen3-30B and Qwen3-VL on most benchmarks while being 4x smaller (~5 GB)
- `bge-m3` - State-of-the-art embedding model with multi-language support, optimized for RAG applications (~1.2 GB)

**Optional Upgrade Models** (commented out, uncomment in docker-compose.yml for better RAG quality):

- `qwen3.5:35b-a3b` - Qwen MoE (35B total, 3B active), native vision, 262K context — direct upgrade from 9B (~22 GB)
- `glm-4.7-flash` - Zhipu GLM, 30B dense, 198K context, #1 Chatbot Arena — best chat quality for RAG answers (~19 GB)

**Optional Lightweight Model** (fast secondary model for simple RAG queries):

- `gemma3n:4b` - Google Gemma 3n, mobile-first multimodal, 128K context — fast fallback for easy questions (~3 GB)

**To customize models**: Edit `docker-compose.yml` → `x-init-ollama` section → Uncomment or add `ollama pull` commands

## 🔗 n8n-Open WebUI Integration

### How It Works

1. User sends message in Open WebUI
2. n8n pipe function intercepts and forwards to n8n webhook
3. n8n processes via workflow and returns response
4. Response displayed in Open WebUI

### Function Features

- ✅ **Auto-imported** during startup
- ✅ **Real-time status** indicators
- ✅ **Session management** for chat context
- ✅ **Error handling** and reporting

### Use Cases

- **RAG Applications**: Document processing with Docling + vector search
- **Document Analysis**: PDF parsing, table extraction, OCR processing
- **Database Integration**: Dynamic data retrieval
- **Multi-API Orchestration**: Complex workflow automation
- **Custom AI Agents**: Specialized task automation

## 🛠️ Docker Compose Architecture

### Services

- **open-webui**: ChatGPT-like interface with auto-imported functions
- **n8n**: Workflow automation with auto-imported workflows
- **docling**: Document parsing and OCR service
- **postgres**: PostgreSQL with pgvector extension
- **pgadmin**: Database management interface
- **ollama-cpu/gpu**: LLM inference engine
- **ollama-pull**: Automatic model downloader

### Key Features

- **Automatic Setup**: Functions and workflows imported on startup
- **Network Isolation**: Internal Docker networks for security
- **Volume Persistence**: Data survives container restarts
- **Profile Support**: CPU-only or GPU-accelerated execution

## 🔧 Troubleshooting

| Issue                    | Solution                                                      |
| ------------------------ | ------------------------------------------------------------- |
| **Port conflicts**       | Edit `ports:` in `docker-compose.yml`                         |
| **Service failures**     | Check logs: `docker compose logs [service-name]`              |
| **GPU issues**           | Verify NVIDIA drivers and container toolkit                   |
| **Model downloads**      | Monitor: `docker compose logs -f ollama-pull-llama-[cpu/gpu]` |
| **Function not working** | Check n8n webhook URL in Open WebUI function settings         |

## 🔒 Security Notes

- **Never commit** `.env` file with real credentials
- **Change default passwords** before production use
- **Local use only** - implement proper security for network exposure
- **Keep updated** - regularly update Docker images

---

**Need more details?** Check the original documentation or the [n8n integration article](https://www.pondhouse-data.com/blog/integrating-n8n-with-open-webui).
