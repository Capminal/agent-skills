# Agent Skills

A community-driven collection of skills for AI agents.

## What are Skills?

Skills are modular abilities that teach AI agents new capabilities. Each skill contains:
- `SKILL.md` - Instructions and API documentation for agents
- `_meta.json` - Skill metadata

## Available Skills

| Skill | Description | Author | Token Address |
|-------|-------------|--------|---------------|
| [capminal](./capminal) | Basic skills to interact with Cap Wallet include transfer/swap/deploy/claim reward... | AndreaPN | 0xbfa733702305280F066D470afDFA784fA70e2649 |
| [contract-interaction](./contract-interaction) | Generic read/write of any smart contract — flexible ABI, address, params (via CAP API key) | AndreaPN | 0xbfa733702305280F066D470afDFA784fA70e2649 |

## Installation

### Manual Installation
```bash
git clone https://github.com/Capminal/agent-skills.git
cp -r agent-skills/capminal ~/.agents/skills/
```

## Contributing

We welcome contributions! To add a new skill:

1. **Fork** this repository
2. **Create** a new folder for your skill: `your-skill-name/`
3. **Add** required files:
   ```
   your-skill-name/
   ├── SKILL.md        # Required: Instructions for agents
   └── _meta.json      # Required: Metadata
   ```
4. **Submit** a Pull Request

### SKILL.md Template

```markdown
---
name: Your Skill Name
description: What your skill does
version: 1.0.0
author: YourName
tags: [tag1, tag2]
---

# Your Skill Name

Description of what this skill does.

## Authentication & Security

Instructions for handling API keys securely.

## API Endpoints

### Endpoint Name

**Request:**
\```bash
curl -X GET "https://api.example.com/endpoint" \
  -H "API_KEY: YOUR_API_KEY"
\```

**Example Response:**
\```json
{
  "success": true,
  "data": {}
}
\```

## Usage Instructions

Step-by-step instructions for agents.

## Error Handling

How to handle common errors.
```

### _meta.json Template

```json
{
  "owner": "your-username",
  "package": "your-skill-name",
  "displayName": "Your Skill Name",
  "latestRelease": {
    "version": "1.0.0",
    "publishedAt": 1234567890000
  }
}
```

## Guidelines

- Keep skills focused on a single purpose
- Include clear API documentation with examples
- Always instruct agents to secure API keys
- Ask users for missing credentials
- Test your skill before submitting

## Links

- [Link3](https://link3.to/capminal)
