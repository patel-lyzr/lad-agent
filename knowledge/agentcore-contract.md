# Bedrock AgentCore Runtime Contract

## The entrypoint pattern

Every agent deployed to AgentCore Runtime must have:

```python
from bedrock_agentcore import BedrockAgentCoreApp

app = BedrockAgentCoreApp()

@app.entrypoint
def invoke(payload):
    # payload is a dict from the HTTP request body
    # Must return a dict
    return {"result": "..."}

if __name__ == "__main__":
    app.run()
```

## Request format
- `payload` is a JSON dict
- Convention: `payload.get("prompt", "")` for the user message
- Can include any additional keys the agent needs

## Response format
- Must return a dict
- Convention: `{"result": "the agent output"}`
- Can include any additional keys

## Container requirements
- Must listen on port 8080 (set via `ENV PORT=8080`)
- Must be linux/amd64 architecture
- `bedrock-agentcore` package handles the HTTP server
- Environment variables are injected by AgentCore Runtime at container start

## Dockerfile pattern
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ENV PORT=8080
EXPOSE 8080
CMD ["python", "agent.py"]
```
