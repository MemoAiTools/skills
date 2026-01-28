# MemoAI Skill

Persistent memory system for AI agents. Search and record learnings using semantic search. Perfect for remembering bug fixes, solutions, architecture decisions, implementation patterns, best practices, workarounds, and technical decisions.

**Use when:**

- Debugging similar issues
- Implementing features
- Searching past learnings
- Recording new knowledge
- Sharing team knowledge across coding sessions

**Requirements:**

- MemoAI Account
- `MEMOAI_PROJECT` environment variable
- MCP client support

---

## Installation

### Quick Install (Recommended)

Use the [AI Agent Skills](https://github.com/skillcreatorai/Ai-Agent-Skills) CLI to install to all your agents at once:

```bash
# Install from GitHub (recommended)
npx ai-agent-skills install MemoAiTools/skills/memoai

# Or install to a specific agent only
npx ai-agent-skills install MemoAiTools/skills/memoai --agent cursor
npx ai-agent-skills install MemoAiTools/skills/memoai --agent claude
npx ai-agent-skills install MemoAiTools/skills/memoai --agent opencode
```

**One command installs to ALL compatible agents:**

- Claude Code
- Cursor
- Codex
- Amp
- VS Code / GitHub Copilot
- Gemini CLI
- Goose
- OpenCode
- Letta

### Manual Installation

If you prefer to install manually, first clone the repository:

```bash
git clone https://github.com/MemoAiTools/skills.git
cd skills
```

Then copy the skill to your agent's directory:

#### Claude Code

```bash
cp -r memoai ~/.claude/skills/
```

#### Cursor

```bash
cp -r memoai .cursor/skills/
```

#### OpenCode

```bash
cp -r memoai ~/.opencode/skill/
```

#### VS Code / GitHub Copilot

```bash
cp -r memoai .github/skills/
```

#### Other Agents

See [AI Agent Skills documentation](https://github.com/skillcreatorai/Ai-Agent-Skills#supported-agents) for other installation paths.

---

## Skill Details

[View full documentation →](memoai/SKILL.md)

**Available MCP Tools:**

- `memoai_memo_search` - Search past learnings with semantic similarity
- `memoai_memo_record` - Record new knowledge for future retrieval

**References:**

- [Common Workflows](memoai/references/WORKFLOWS.md) - Step-by-step usage examples
- [API Documentation](memoai/references/API.md) - Technical schemas and debugging

---

## License

MIT

---

## Links

- [Agent Skills Specification](https://agentskills.io)
- [AI Agent Skills CLI](https://github.com/skillcreatorai/Ai-Agent-Skills)
