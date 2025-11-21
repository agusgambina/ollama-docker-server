# Quick Start Guide

## 🚀 Your Setup is Ready!

### Services Running

✅ **Ollama Server** - Running on `http://localhost:11434`
✅ **Open WebUI** - Running on `http://localhost:3000`

### Available Models

- **mistral:latest** (4.4 GB) - Ready to use!

---

## 🎯 How to Use

### Option 1: Web Interface (Recommended)

1. Open your browser and go to:
   ```
   http://localhost:3000
   ```

2. Select "mistral:latest" from the model dropdown

3. Start chatting!

> **Note**: Authentication is disabled by default. To enable login, set `WEBUI_AUTH=True` in docker-compose.yml

---

### Option 2: API (For Developers)

#### Chat with the model:
```bash
curl http://localhost:11434/api/chat -d '{
  "model": "mistral",
  "messages": [
    {
      "role": "user",
      "content": "Explain quantum computing in simple terms"
    }
  ],
  "stream": false
}'
```

#### Generate completion:
```bash
curl http://localhost:11434/api/generate -d '{
  "model": "mistral",
  "prompt": "Write a haiku about programming",
  "stream": false
}'
```

---

### Option 3: Command Line

```bash
# Interactive chat
docker exec ollama ollama run mistral

# One-off prompt
docker exec ollama ollama run mistral "What is the capital of France?"
```

---

## 📦 Managing Your Setup

### Stop everything:
```bash
docker-compose down
```

### Start everything:
```bash
docker-compose up -d
```

### View logs:
```bash
docker-compose logs -f
```

### View Ollama logs only:
```bash
docker-compose logs -f ollama
```

### View Open WebUI logs only:
```bash
docker-compose logs -f open-webui
```

---

## 🔧 Adding More Models

### Pull a new model:
```bash
docker exec ollama ollama pull llama2
```

### Popular models to try:
- `llama2` - Meta's Llama 2
- `codellama` - Code generation
- `mixtral` - Larger, more powerful model
- `phi` - Microsoft's compact model
- `gemma` - Google's model

### Check downloaded models:
```bash
docker exec ollama ollama list
```

---

## 🌐 Open WebUI Features

- **Multiple Conversations**: Keep different chat threads
- **Model Switching**: Try different models in the same interface
- **Conversation History**: All your chats are saved
- **Markdown Rendering**: Beautiful code blocks and formatting
- **Dark Mode**: Toggle in settings
- **Multi-user**: Create accounts for your team
- **Import/Export**: Save and share conversations

---

## 🆘 Troubleshooting

### Can't access the web UI?
- Check if port 3000 is free: `lsof -i :3000`
- Change port in `.env` file if needed

### Model not showing up?
- Wait a few seconds for Open WebUI to detect new models
- Refresh the page
- Check Ollama is running: `docker ps | grep ollama`

### Out of memory?
- Mistral requires ~8GB RAM
- Restart Docker Desktop and increase memory limit
- Try smaller models like `phi`

---

## 🎉 Next Steps

1. Open http://localhost:3000 in your browser
2. Create your admin account
3. Select "mistral:latest"
4. Start building amazing AI applications!

**Enjoy your local AI playground!** 🚀

