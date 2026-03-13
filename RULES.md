# Rules

## Must Always
- Read every .py file in the project before generating anything
- Detect framework from imports: `langgraph` → LangGraph, `crewai` → CrewAI, `strands` → Strands
- Detect all required environment variables (see Environment Variable Detection below)
- Generate `agent.py` with `from bedrock_agentcore import BedrockAgentCoreApp`
- Generate a `Dockerfile` based on `python:3.11-slim`
- If the project uses `pyproject.toml`: `COPY . .` then `pip install .` (never `-e .`)
- If the project uses `requirements.txt`: `COPY requirements.txt .` then `pip install -r requirements.txt` then `COPY . .`
- Add `bedrock-agentcore` to `requirements.txt` if not already present
- Preserve the user's existing `requirements.txt` dependencies
- Use the project's actual import paths in the generated wrapper
- Set `ENV PORT=8080` and `EXPOSE 8080` in Dockerfile
- Use `CMD ["python", "agent.py"]` in Dockerfile
- Always include the full `lad deploy` command with all required `-e` flags in the report

## Must Never
- Modify any existing file in the user's project (except appending to requirements.txt)
- Hardcode API keys or secrets in generated files
- Generate code that restructures or refactors the user's workflow
- Assume a specific LLM provider — the user's code handles that
- Add unnecessary dependencies beyond `bedrock-agentcore`

## Output Constraints
- `agent.py` must be under 50 lines
- `Dockerfile` must be under 20 lines
- Only output the generated file contents — no explanations unless there's an error
- If the framework can't be detected, report the error clearly

## Detection Rules

### LangGraph
- Look for: `from langgraph.graph import StateGraph` or `StateGraph(`
- Find the `.compile()` call — that's the graph object
- Wrapper calls `graph.invoke({"messages": [...]})` and extracts last message

### CrewAI
- Look for: `from crewai import Crew` or `Crew(`
- Find the `Crew(agents=[...], tasks=[...])` definition
- Wrapper calls `crew.kickoff()` and returns `.raw`

### Strands
- Look for: `from strands import Agent` or `strands.Agent(`
- Find the `Agent(...)` instantiation
- Wrapper calls `agent(prompt)` and returns the result

### Generic Python
- If no framework detected, look for a `main()` or `run()` function
- Wrap it as a callable that takes a prompt string

## Environment Variable Detection

Scan ALL .py files and config files (.env, .env.example, *.yaml, *.toml) for environment variables. Look for:

### Patterns to match
- `os.environ["KEY"]` or `os.environ.get("KEY")`
- `os.getenv("KEY")`
- `environ["KEY"]` or `environ.get("KEY")`
- `.env` file entries: `KEY=value`
- `.env.example` file entries
- YAML/TOML config referencing `${KEY}` or `$KEY`

### Common env vars by framework
- **OpenAI**: `OPENAI_API_KEY`
- **Anthropic/Bedrock**: `ANTHROPIC_API_KEY`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`
- **Tavily**: `TAVILY_API_KEY`
- **LangSmith**: `LANGCHAIN_API_KEY`, `LANGCHAIN_TRACING_V2`, `LANGCHAIN_PROJECT`
- **Azure**: `AZURE_API_KEY`, `AZURE_API_BASE`, `AZURE_API_VERSION`
- **Generic**: `DATABASE_URL`, `REDIS_URL`, `API_KEY`

### Classification
For each detected env var, classify as:
- **required** — used directly without a fallback default (e.g., `os.environ["KEY"]`, `os.getenv("KEY")` with no default)
- **optional** — has a fallback default (e.g., `os.environ.get("KEY", "default")`, `os.getenv("KEY", "fallback")`)

### Report format
Always include in the output report:

```
Environment variables:
  Required:
    - OPENAI_API_KEY (model.py:12)
    - TAVILY_API_KEY (tools/search.py:5)
  Optional:
    - LANGCHAIN_TRACING_V2 (config.py:8, default: "false")

Deploy command:
  lad deploy . --name <agent_name> \
    -e OPENAI_API_KEY=<your-key> \
    -e TAVILY_API_KEY=<your-key>
```
