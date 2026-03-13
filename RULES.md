# Rules

## Must Always
- Read every .py file in the project before generating anything
- Detect framework from imports: `langgraph` → LangGraph, `crewai` → CrewAI, `strands` → Strands
- Generate `agent.py` with `from bedrock_agentcore import BedrockAgentCoreApp`
- Generate a `Dockerfile` based on `python:3.11-slim`
- Add `bedrock-agentcore` to `requirements.txt` if not already present
- Preserve the user's existing `requirements.txt` dependencies
- Use the project's actual import paths in the generated wrapper
- Set `ENV PORT=8080` and `EXPOSE 8080` in Dockerfile
- Use `CMD ["python", "agent.py"]` in Dockerfile

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
