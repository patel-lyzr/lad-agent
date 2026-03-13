---
name: wrap-agent
description: Scans a project directory, detects the agent framework, and generates BedrockAgentCoreApp wrapper + Dockerfile
license: MIT
allowed-tools: Read Edit Grep Glob Bash Write
metadata:
  author: lyzr
  version: "1.0.0"
  category: developer-tools
---

# Wrap Agent for Bedrock AgentCore

## Step 1 — Scan the project

Read all `.py` files in the target directory. Look at imports to detect the framework:

| Import pattern | Framework |
|---|---|
| `from langgraph` or `import langgraph` | LangGraph |
| `from crewai` or `import crewai` | CrewAI |
| `from strands` or `import strands` | Strands |

## Step 2 — Find the workflow entry point

### LangGraph
Find the file containing `.compile()` — this produces the runnable graph. Trace back to find:
- The `StateGraph(State)` definition
- The module path to import the compiled graph
- Whether State uses `messages` (chat pattern) or custom keys

### CrewAI
Find the file containing `Crew(` — this defines the crew. Trace back to find:
- All `Agent()` definitions
- All `Task()` definitions
- The `Process` type (sequential, hierarchical)
- The module path to import or reconstruct the crew

### Strands
Find the file containing `Agent(` from strands — trace back to find:
- Tool definitions
- The module path to import the agent

## Step 3 — Generate `agent.py`

Create a wrapper that:
1. Imports `BedrockAgentCoreApp` from `bedrock_agentcore`
2. Imports the user's compiled graph / crew / agent
3. Defines `@app.entrypoint` handler that:
   - Extracts `prompt` from `payload`
   - Calls the workflow with the prompt
   - Returns `{"result": <output>}`

### LangGraph template
```python
from bedrock_agentcore import BedrockAgentCoreApp
from <module> import <graph_var>

app = BedrockAgentCoreApp()

@app.entrypoint
def invoke(payload):
    prompt = payload.get("prompt", "")
    result = <graph_var>.invoke({"messages": [{"role": "user", "content": prompt}]})
    return {"result": result["messages"][-1].content}

if __name__ == "__main__":
    app.run()
```

### CrewAI template
```python
from bedrock_agentcore import BedrockAgentCoreApp
from <module> import <crew_builder>

app = BedrockAgentCoreApp()

@app.entrypoint
def invoke(payload):
    prompt = payload.get("prompt", "")
    crew = <crew_builder>(prompt)
    result = crew.kickoff()
    return {"result": result.raw}

if __name__ == "__main__":
    app.run()
```

### Strands template
```python
from bedrock_agentcore import BedrockAgentCoreApp
from <module> import <agent_var>

app = BedrockAgentCoreApp()

@app.entrypoint
def invoke(payload):
    prompt = payload.get("prompt", "")
    result = <agent_var>(prompt)
    return {"result": str(result)}

if __name__ == "__main__":
    app.run()
```

## Step 4 — Generate `Dockerfile`

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

## Step 5 — Patch `requirements.txt`

If `bedrock-agentcore` is not in the existing requirements.txt, append it.
If no requirements.txt exists, create one with `bedrock-agentcore` plus detected dependencies.

## Step 6 — Report

Output:
- Framework detected
- Files generated (agent.py, Dockerfile, requirements.txt changes)
- The `lad deploy` command to run next
