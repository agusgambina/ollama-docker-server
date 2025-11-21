# Ollama Server

A Docker-based setup for running [Ollama](https://ollama.ai/) - a local AI model server.

## Prerequisites

- Docker
- Docker Compose

## Setup

1. **Create the environment file**

You can create a `.env` file in the project root:

```bash
OLLAMA_PORT=11434
WEBUI_PORT=3000
```

You can change the port numbers if needed.

2. **Start the Ollama service**

```bash
docker-compose up -d
```

This will:

- Pull the Ollama Docker image
- Pull the Open WebUI Docker image
- Start both containers in detached mode
- Expose the Ollama API on port 11434 (or As specified in `.env`)
- Expose the Open WebUI on port 3000 (or As specified in `.env`)
- Create a `local_data` folder to persist your models and UI data

3. **Verify the services are running**

```bash
docker ps
```

You should see both the `ollama` and `open-webui` containers running.

4. **Access the Web UI**

Open your browser and navigate to:

```
http://localhost:3000
```

On first launch, you'll be prompted to create an admin account. this interface provides:

- A ChatGPT-like interface for interacting with your models
- Model management
- Conversation history
- User management
- And more!

## Downloading Models

### Open LLMs

### Pull a Model

To download a model, use the `ollama pull` command:

```bash
docker exec -it ollama ollama pull llama2
```

### Available Models (Code-oriented)

1. **Meta's Llama 2**

Meta's latest and greatest language model:
[Code-oriented models](https://ollama.ai/library?category=code)

2. **Code Generation Model**

A model that can generate code from a prompt:
[Code-oriented models](https://ollama.ai/library?category=code)

3. **Mistral 7B**

Another powerful language model, designed for code writing:
[Code-oriented models](https://ollama.ai/library?category=code)

4. **Mixtral 8x7B**

A more advanced variant of Mistral specifically for code writing:
[Code-oriented models](https://ollama.ai/library?category=code)

5. **Microsoft Phi-2**

An AI model focused on language processing and natural language generation:
[Code-oriented models](https://ollama.ai/library?category=code)

6. **Google Gemma**

A machine learning model for creating code snippets:
[Code-oriented models](https://ollama.ai/library?category=code)

### Available Models (Non-code)

1. **Coding Tools**

Tools for writing, debugging, and managing code:
[Non-code models](https://ollama.ai/library?category=non-code)

2. **Marketing Copy Generator**

A model capable of generating compelling marketing copy:
[Non-code models](https://ollama.ai/library?category=non-code)

3. **Content Writer**

An AI that can generate content, like blog posts or articles:
[Non-code models](https://ollama.ai/library?category=non-code)

4. **Grammar and Style Correction Model**

A model capable of correcting grammar and style errors in text:
[Non-code models](https://ollama.ai/library?category=non-code)

5. **Resume Writer**

A model that can help you write compelling resumes:
[Non-code models](https://ollama.ai/library?category=non-code)

6. **Script Generator**

Creates scripts for different types of media:
[Non-code models](https://ollama.ai/library?category=non-code)

## Using Open WebUI

Open WebUI provides a user-friendly interface for interacting with your Ollama models.

### Features

- **ChatGPT-style Interface**: Familiar chat interface with streaming responses
- **Model Selection**: Easy switching between downloaded models
- **Conversation History**: All your chats are saved and searchable
- **Markdown Support**: Rich text formatting in responses
- **Code Highlighting**: Syntax highlighting for code blocks
- **Model Management**: Pull and manage models directly from the UI
- **Multi-user Support**: Create accounts for different users
- **Dark Mode**: Easy on the eyes

### First-Time Setup

1. Navigate to `http://localhost:3000`
2. Create an admin account (first user becomes admin)
3. Select a model from the dropdown (you'll need to have models downloaded)
4. Start chatting!

## Using Ollama

### Interactive Chat

Start an interactive chat session with a model:

```bash
docker exec -it ollama ollama run llama2
```

Type your messages and press Enter. Type `/bye` to exit.

### API Usage

Ollama provides a REST API at `http://localhost:${OLLAMA_PORT}` (default: 11434).

**Generate a completion:**

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama2",
  "prompt": "Why is the sky blue?",
  "stream": false
}'
```

**Chat with a model:**

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "llama2",
  "messages": [
    {
      "role": "user",
      "content": "Hello! How are you?"
    }
  ],
  "stream": false
}'
```

## Managing the Service

### Stop the service

```bash
docker-compose down
```

### View logs

```bash
docker-compose logs -f ollama
```

### Restart the service

```bash
docker-compose restart
```

## Data Persistence

All downloaded models and data are stored in the `local_data` folder in your project directory. this ensures your models persist across container restarts.

To completely remove all data:

```bash
docker-compose down
rm -rf local_data
```

## Configuration

You can customize the following in your `.env` file:

- `OLLAMA_PORT` - The port to expose the Ollama API on your host machine (default: 11434)
- `WEBUI_PORT` - The port to expose the Open WebUI on your host machine (default: 3000)

## Troubleshooting

### Port already in use

If port 11434 is already in use, change `OLLAMA_PORT` in your `.env` file to a different port.

### Out of memory

Some larger models require significant RAM. If you encounter memory issues:
- Try smaller model variants (e.g., `llama2:7b` instead of `llama2:13b`)
- Increase Docker's memory limit in Docker Desktop settings

## Resources

- [Ollama Documentation](https://github.com/ollama/ollama)
- [Ollama Model Library](https://ollama.ai/library)
- [Ollama API Documentation](https://github.com/ollama/ollama/blob/main/docs/api.md)
