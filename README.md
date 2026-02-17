# Agent Orchestration System

**TypeScript framework for building autonomous, collaborative AI agents**

Key capabilities:

- **Autonomous Agents**: Agents gather information via tools, making independent decisions without massive context dumps
- **Deep Reasoning**: Multi-provider thinking support (Claude, OpenAI, OpenRouter) for complex planning and problem-solving
- **Agent Collaboration**: Agents delegate to specialized sub-agents, forming dynamic teams for complex tasks
- **Multi-Provider Support**: Switch between Anthropic, OpenAI, OpenRouter, or custom providers with simple configuration
- **Production-Ready**: Built-in security, retry logic, session persistence, and comprehensive monitoring
- **Cost Efficient**: Smart caching delivers up to 90% cost savings on multi-agent workflows

## 📦 Installation

Install from npm (no authentication required):

```bash
# Install core library
npm install @nielspeter/agent-orchestration-core

# Install CLI globally
npm install -g @nielspeter/agent-orchestration-cli

# Or install both
npm install @nielspeter/agent-orchestration-core @nielspeter/agent-orchestration-cli
```

**Package URLs:**
- Core: https://www.npmjs.com/package/@nielspeter/agent-orchestration-core
- CLI: https://www.npmjs.com/package/@nielspeter/agent-orchestration-cli

## 🎯 Architecture Highlights

### Clean Middleware Pipeline
```typescript
type Middleware = (ctx: MiddlewareContext, next: () => Promise<void>) => Promise<void>;
```

The monolithic 500-line `AgentExecutor` has been refactored into a clean pipeline of focused middleware:
- **ErrorHandlerMiddleware** - Global error boundary
- **AgentLoaderMiddleware** - Loads agents and filters tools
- **ThinkingMiddleware** - Validates and normalizes thinking configuration
- **ContextSetupMiddleware** - Manages conversation context
- **ProviderSelectionMiddleware** - Selects LLM provider (Anthropic, OpenRouter, etc.)
- **SafetyChecksMiddleware** - Enforces limits (depth, iterations, tokens)
- **SmartRetryMiddleware** - Retries on rate limits (429) with exponential backoff
- **LLMCallMiddleware** - Handles LLM communication
- **ToolExecutionMiddleware** - Orchestrates tool execution

### Everything is an Agent
- No special orchestrator class - all agents use the same pipeline
- Agents are defined as markdown files with YAML frontmatter
- Orchestration emerges through the `Delegate` tool for delegation

### Pull Architecture with Caching
When agent A delegates to agent B:
1. B receives **minimal context** (~5-500 tokens) - just the task prompt
2. B uses tools (Read, Write, List, Grep, Delegate) to **pull** information it needs
3. Anthropic's cache makes "redundant" reads efficient (90% cost savings)
4. Clean separation - each agent has independent context

## 🔄 Core Patterns

### The Agentic Loop (ReAct Pattern)
Each agent automatically implements the **Reason → Act → Observe** loop:
1. **Reason**: Agent analyzes prompt and decides what to do
2. **Act**: Agent calls tools to gather information or take action
3. **Observe**: Agent processes tool results
4. **Repeat**: Continue until task is complete (no more tool calls)

This iterative refinement allows agents to:
- Build understanding incrementally
- Correct mistakes
- Ground responses in actual data
- Never hallucinate file contents

See [Agentic Loop Pattern](docs/agentic-loop-pattern.md) for details.

### Iteration vs Delegation
- **Iteration**: Same agent refining its response (limited by MAX_ITERATIONS)
- **Delegation**: Calling another agent via Delegate tool (limited by MAX_DEPTH)

## 🚀 Quick Start

```bash
# Install dependencies
npm install

# Set up API keys (at least one required)
cp .env.example .env
# Edit .env with your ANTHROPIC_API_KEY or OPENROUTER_API_KEY

# Optional: Configure providers
cp providers-config.example.json providers-config.json

# Build the project
npm run build

# Run tests
npm test              # Run all tests
npm run test:unit     # Unit tests only (no API)
npm run test:integration # Integration tests (requires API key)

# Use CLI
npm run cli -- -p "Hello, world!"       # CLI tool
echo "Analyze this" | npm run cli       # stdin support

# Run examples
npx tsx packages/examples/quickstart.ts          # Simple quickstart
npx tsx packages/examples/orchestration.ts       # Agent orchestration
npx tsx packages/examples/configuration.ts       # Config file usage
npx tsx packages/examples/code-first-config.ts   # Code-first configuration (no files)
npx tsx packages/examples/logging.ts             # Logging features
npx tsx packages/examples/mcp-integration.ts     # MCP server support
npx tsx packages/examples/werewolf-game.ts       # Autonomous multi-agent game
npx tsx packages/examples/coding-team.ts         # Collaborative coding agents
```

