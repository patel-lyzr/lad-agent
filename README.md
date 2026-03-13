# lad-wrapper

Lyzr Agent Deploy wrapper agent — analyzes LangGraph/CrewAI/Strands projects and generates BedrockAgentCoreApp wrapper + Dockerfile for deployment to Bedrock AgentCore Runtime.

## Run

```bash
npx @open-gitagent/gitagent run -r https://github.com/lyzr/lad-agent
```

## What It Can Do

- Detects framework (LangGraph, CrewAI, Strands) from project imports
- Generates `agent.py` with `BedrockAgentCoreApp` wrapper
- Generates `Dockerfile` for container deployment
- Patches `requirements.txt` with `bedrock-agentcore`
- Works with multi-agent workflows (entire graph/crew runs as one unit)

## Structure

```
lad-agent/
├── agent.yaml
├── SOUL.md
├── RULES.md
├── skills/
│   └── wrap-agent/
│       └── SKILL.md
└── knowledge/
    ├── index.yaml
    └── agentcore-contract.md
```

## Built with

[gitagent](https://github.com/open-gitagent/gitagent) — a git-native, framework-agnostic open standard for AI agents.
