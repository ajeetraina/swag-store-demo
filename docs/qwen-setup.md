# Qwen Model Setup Instructions

This guide explains how to set up the Qwen model on your host system using Docker Desktop's Model Runner, following the pen-shop-demo architecture.

## Prerequisites

- Docker Desktop with Model Runner enabled
- At least 8GB RAM available for the model
- 10GB free disk space for model weights

## Setup Steps

### 1. Enable Docker Desktop Model Runner

1. Open Docker Desktop
2. Go to Settings > Features in development
3. Enable "Model Runner" feature
4. Restart Docker Desktop

### 2. Download and Run Qwen Model

```bash
# Pull and run Qwen 2.5 7B Instruct model
docker model run qwen2.5:7b-instruct

# Alternative: Use the latest Qwen model
docker model run qwen:latest

# For systems with more RAM, use the larger model
docker model run qwen2.5:14b-instruct
```

### 3. Verify Model is Running

The model will be available at `http://localhost:11434` on your host system.

Test the model:
```bash
# Test with curl
curl -X POST http://localhost:11434/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b-instruct",
    "prompt": "Hello, how are you?",
    "stream": false
  }'

# Test with simple chat
curl -X POST http://localhost:11434/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b-instruct",
    "messages": [
      {
        "role": "user", 
        "content": "What is Docker?"
      }
    ]
  }'
```

### 4. Configure SwagStore to Use Host Model

The SwagStore containers will access the Qwen model via `host.docker.internal:11434`.

Environment variables in docker-compose.yml:
```yaml
environment:
  QWEN_MODEL_URL: http://host.docker.internal:11434
  QWEN_MODEL_NAME: qwen2.5:7b-instruct
```

### 5. Model Configuration Options

You can customize the model behavior by setting these environment variables:

```bash
# In your .env file
QWEN_MODEL_NAME=qwen2.5:7b-instruct
QWEN_MAX_TOKENS=4096
QWEN_TEMPERATURE=0.7
QWEN_TOP_P=0.9
QWEN_CONTEXT_LENGTH=8192
```

## Model Management Commands

```bash
# List running models
docker model list

# Stop a model
docker model stop qwen2.5:7b-instruct

# Remove a model
docker model rm qwen2.5:7b-instruct

# View model logs
docker model logs qwen2.5:7b-instruct

# Check model status
docker model ps
```

## Security Considerations

1. **Network Access**: The model runs on localhost only by default
2. **Resource Limits**: Monitor CPU and memory usage
3. **API Security**: Consider adding authentication for production use
4. **Model Updates**: Regularly update to latest secure versions

## Troubleshooting

### Model Won't Start
```bash
# Check available resources
docker system df
docker system events

# Verify Docker Desktop Model Runner is enabled
docker version | grep -i model
```

### Connection Issues
```bash
# Test from host
curl -v http://localhost:11434/api/version

# Test from container
docker run --rm curlimages/curl \
  curl -v http://host.docker.internal:11434/api/version
```

### Performance Issues
```bash
# Monitor model resource usage
docker stats $(docker ps -q --filter ancestor=qwen)

# Check model configuration
docker model inspect qwen2.5:7b-instruct
```

## Integration with MCP Gateway

The SwagStore uses the Qwen model through the MCP Gateway for:

- Customer service chat
- Product recommendations
- Inventory queries
- Security threat analysis

The MCP Gateway (port 8081) acts as a secure intermediary between the frontend and the Qwen model, providing:

- Input sanitization
- Output filtering  
- Rate limiting
- Audit logging
- Security monitoring

## Advanced Configuration

For production deployments, consider:

1. **Model Optimization**: Use quantized models for better performance
2. **Load Balancing**: Run multiple model instances
3. **Monitoring**: Add Prometheus metrics
4. **Backup**: Regular model state backups
5. **Updates**: Automated security updates

## Support

- [Docker Model Runner Documentation](https://docs.docker.com/desktop/model-runner/)
- [Qwen Model Documentation](https://qwenlm.github.io/)
- [MCP Gateway Documentation](https://docs.docker.com/ai/mcp-gateway/)

---

**Note**: This setup follows the pen-shop-demo architecture where the AI model runs on the host system for better performance and resource management, while MCP servers run in isolated containers.