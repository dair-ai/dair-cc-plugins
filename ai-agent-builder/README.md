# AI Agent Builder Plugin

A Claude Code plugin that helps you build modern web applications with AI capabilities using a unified tech stack.

## Installation

Add the DAIR.AI marketplace and install the plugin:

```bash
/plugin marketplace add dair-ai/dair-cc-plugins
/plugin install ai-agent-builder@dair-cc-plugins
```

## Usage

Once installed, invoke the web-builder skill when you want to build a web application:

```
/skill web-builder
```

Or simply ask Claude Code to help you build a web app - the skill will be activated automatically when relevant.

### Example Prompts

**Start a new project:**
```
Help me create a new web app for tracking personal expenses
```

**Add authentication:**
```
Add user authentication with email/password and Google OAuth
```

**Build an AI feature:**
```
Add a chat interface that uses Claude to help users categorize their expenses
```

**Deploy the app:**
```
Help me deploy this app to Vercel
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
| Deployment | Vercel | Hosting and edge functions |
| Language | TypeScript | Type safety |

## What the Skill Provides

### Project Scaffolding
- Next.js project setup with TypeScript and Tailwind
- shadcn/ui component library configuration
- Supabase integration for database and auth
- Recommended folder structure

### Implementation Patterns
- Supabase client setup (browser, server, middleware)
- Authentication flows (sign up, sign in, protected routes)
- Database schema design with Row Level Security
- API routes and server actions
- Form handling with react-hook-form and Zod

### Claude Agent SDK Integration
- `query()` function for one-off agentic tasks
- Custom tools with `tool()` and `createSdkMcpServer()`
- Session management (resume, fork)
- Permission modes for tool execution control
- Streaming responses in Next.js API routes
- Built-in tools (Read, Write, Edit, Bash, Glob, Grep, etc.)

### Deployment
- Vercel CLI commands and workflows
- Environment variable management
- Production checklist

## License

MIT
