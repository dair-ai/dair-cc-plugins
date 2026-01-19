# AI Agent Builder Plugin

A Claude Code plugin that helps you build multi-agent AI systems with the Claude Agent SDK. From simple single-agent queries to complex orchestrated pipelines with Vercel Sandbox deployment.

## Installation

Add the DAIR.AI marketplace and install the plugin:

```bash
/plugin marketplace add dair-ai/dair-cc-plugins
/plugin install ai-agent-builder@dair-cc-plugins
```

## Usage

Once installed, invoke the web-builder skill when you want to build an agentic application:

```
/skill web-builder
```

Or simply ask Claude Code to help you build an AI agent - the skill will be activated automatically when relevant.

### Example Prompts

**Build a research agent:**
```
Help me build a research agent that uses Exa search to find and synthesize information
```

**Create a multi-agent pipeline:**
```
Build a multi-agent system with a planner, searcher, and report writer
```

**Add Vercel Sandbox for production:**
```
Add Vercel Sandbox support so my agent can run in production
```

**Simple agent query:**
```
Add a chat interface that uses Claude Agent SDK to answer questions
```

## Tech Stack

The plugin guides you through building with:

| Layer | Technology | Purpose |
|-------|------------|---------|
| Framework | Next.js 14+ (App Router) | Full-stack React framework |
| UI Components | shadcn/ui | Accessible, customizable components |
| Styling | Tailwind CSS | Utility-first CSS |
| Database | Supabase PostgreSQL | Managed PostgreSQL with real-time |
| Authentication | Supabase Auth | User authentication |
| AI | Claude Agent SDK | AI agent capabilities with built-in tools |
| AI Tools | Exa (exa-js) | Neural web search for agents |
| Agent Runtime | Vercel Sandbox | Isolated container for production agents |
| Deployment | Vercel | Hosting and edge functions |
| Language | TypeScript | Type safety |

## What the Skill Provides

### Multi-Agent Orchestration
- **Orchestrator pattern** - Coordinate multiple specialized subagents
- **Subagent definitions** - Planner, executor, and writer agents with specific roles
- **Stage tracking** - Real-time progress through pipeline stages
- **Data passing** - Complete context transfer between agents

### Vercel Sandbox Integration
- **Environment detection** - Auto-route between local and production
- **Sandbox execution** - Run agents in isolated containers
- **Script generation** - Dynamic agent code for sandbox
- **Progressive learning** - Build locally first, add sandbox for production

### SSE Streaming for Pipelines
- **Message types** - stage_change, status, result, error
- **Real-time updates** - Stream progress to frontend
- **Stage announcements** - Parse orchestrator stage transitions

### State Management
- **Pipeline hooks** - useResearchAgent for complex state
- **Stage tracking** - pending/active/completed status
- **Source extraction** - Parse tool results for discovered sources

### Claude Agent SDK Patterns
- `query()` function for agentic tasks
- Custom tools with `tool()` and `createSdkMcpServer()`
- Exa search integration (neural search, get_contents, find_similar)
- Session management (resume, fork)
- Permission modes for tool execution control

### Project Scaffolding
- Next.js project setup with TypeScript and Tailwind
- shadcn/ui component library configuration
- Supabase integration for database and auth
- Recommended folder structure for agent projects

### Deployment
- Vercel CLI commands and workflows
- Vercel Sandbox configuration
- Environment variable management
- Production checklist

## Progressive Learning Path

Students can build in stages:

1. **Stage 1**: Simple single-agent queries with Claude Agent SDK
2. **Stage 2**: Add custom tools and Exa search integration
3. **Stage 3**: Implement multi-agent orchestration
4. **Stage 4**: Add Vercel Sandbox for production deployment

## License

MIT
