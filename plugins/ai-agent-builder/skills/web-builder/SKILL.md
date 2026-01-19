---
name: ai-agent-builder
description: Build multi-agent AI systems with Claude Agent SDK. Use this skill for creating agentic applications with orchestrator patterns, Exa search integration, SSE streaming, and Vercel Sandbox deployment. Supports progressive learning from simple agents to production multi-agent pipelines.
license: MIT
---

# Web Builder Skill

Use this skill when helping students build modern web applications with AI capabilities. This skill provides guidance for a unified tech stack optimized for building production-ready agentic applications, from simple single-agent queries to complex multi-agent orchestration with Vercel Sandbox deployment.

## Unified Tech Stack

When building web applications with this skill, use the following stack:

| Layer | Technology | Purpose |
|-------|------------|---------|
| Runtime | Node.js | JavaScript runtime |
| Framework | Next.js 14+ (App Router) | Full-stack React framework |
| UI Components | shadcn/ui | Accessible, customizable components |
| Styling | Tailwind CSS | Utility-first CSS |
| Database | Supabase PostgreSQL | Managed PostgreSQL with real-time |
| Authentication | Supabase Auth | User authentication |
| AI | Claude Agent SDK (@anthropic-ai/claude-agent-sdk) | AI agent capabilities |
| AI Tools | Exa (exa-js) | Neural web search for agents |
| Agent Runtime | Vercel Sandbox (@vercel/sandbox) | Isolated container for production agents |
| Deployment | Vercel | Hosting and edge functions |
| Language | TypeScript | Type safety |

---

## Project Scaffolding

### Initial Setup

When creating a new project, guide students through these steps:

1. **Create Next.js project with TypeScript:**
   ```bash
   npx create-next-app@latest my-app --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
   ```

2. **Install shadcn/ui:**
   ```bash
   npx shadcn@latest init
   ```
   - Select "New York" style for modern aesthetics
   - Use CSS variables for theming

3. **Install Supabase:**
   ```bash
   npm install @supabase/supabase-js @supabase/ssr
   ```

4. **Install Claude Agent SDK:**
   ```bash
   npm install @anthropic-ai/claude-agent-sdk
   ```

5. **Install Exa for web search (optional):**
   ```bash
   npm install exa-js
   ```

6. **Install Vercel Sandbox (for production agent deployment):**
   ```bash
   npm install @vercel/sandbox
   ```

7. **Additional dependencies:**
   ```bash
   npm install zod react-hook-form @hookform/resolvers
   ```

### Environment Variables

Guide students to create `.env.local` with:

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=your-project-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# Anthropic
ANTHROPIC_API_KEY=your-api-key

# Exa (for web search - get key at https://exa.ai/)
EXA_API_KEY=your-exa-api-key

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

For **Vercel production deployment** with agent sandbox support, add these to your Vercel project settings:

```env
# Vercel Sandbox (required for production agent execution)
VERCEL_TOKEN=your-vercel-api-token
VERCEL_PROJECT_ID=your-project-id
VERCEL_TEAM_ID=your-team-id  # Optional, only for team projects
```

**Getting Vercel Sandbox credentials:**
1. `VERCEL_TOKEN`: Create at https://vercel.com/account/tokens
2. `VERCEL_PROJECT_ID`: Found in Project Settings → General
3. `VERCEL_TEAM_ID`: Found in Team Settings (only needed for team projects)

### Recommended Folder Structure

```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/            # Auth route group
│   │   ├── login/
│   │   └── signup/
│   ├── (dashboard)/       # Protected routes
│   │   └── dashboard/
│   ├── api/               # API routes
│   │   ├── chat/
│   │   └── webhooks/
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── ui/                # shadcn components
│   ├── forms/             # Form components
│   └── shared/            # Shared components
├── lib/
│   ├── supabase/
│   │   ├── client.ts      # Browser client
│   │   ├── server.ts      # Server client
│   │   └── middleware.ts  # Auth middleware
│   ├── agent/
│   │   ├── tools.ts       # Custom tools with tool() and createSdkMcpServer()
│   │   ├── config.ts      # Orchestrator configuration
│   │   ├── subagents.ts   # Subagent definitions (planner, executor, writer)
│   │   ├── orchestrator.ts # Pipeline runner
│   │   └── prompts.ts     # System prompts for agents
│   ├── sandbox/
│   │   ├── index.ts       # Environment detection
│   │   └── runner.ts      # Vercel Sandbox execution
│   └── utils.ts
├── hooks/                 # Custom React hooks
├── types/                 # TypeScript types
└── middleware.ts          # Next.js middleware
```

---

## Implementation Patterns

### Supabase Client Setup

**Browser Client (`lib/supabase/client.ts`):**
- Create a singleton client using `createBrowserClient` from `@supabase/ssr`
- Use `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY`

**Server Client (`lib/supabase/server.ts`):**
- Create client using `createServerClient` from `@supabase/ssr`
- Access cookies via `cookies()` from `next/headers`
- Use for Server Components and Route Handlers

**Middleware (`middleware.ts`):**
- Refresh auth tokens on each request
- Protect routes by checking session
- Redirect unauthenticated users to login

### Authentication with Supabase Auth

**Sign Up Flow:**
- Use Supabase `auth.signUp()` with email/password
- Handle email confirmation if enabled
- Create user profile in a separate `profiles` table via trigger

**Sign In Flow:**
- Use `auth.signInWithPassword()` for email/password
- Support OAuth providers via `auth.signInWithOAuth()`
- Store session automatically via Supabase SSR helpers

**Protected Routes:**
- Check session in middleware
- Use route groups like `(dashboard)` for protected areas
- Redirect to login if no session

**Sign Out:**
- Call `auth.signOut()` from client
- Clear session and redirect to home

### Database Schema Design

**Best Practices:**
- Enable Row Level Security (RLS) on all tables
- Create policies for each operation (SELECT, INSERT, UPDATE, DELETE)
- Use `auth.uid()` to scope queries to current user
- Create a `profiles` table linked to `auth.users`

**Common Schema Patterns:**
- Use UUIDs for primary keys
- Add `created_at` and `updated_at` timestamps
- Use foreign keys with cascading deletes where appropriate
- Create indexes on frequently queried columns

### API Routes with Route Handlers

**Pattern for Route Handlers (`app/api/*/route.ts`):**
- Export named functions: `GET`, `POST`, `PUT`, `DELETE`
- Use `NextRequest` and return `NextResponse`
- Validate input with Zod schemas
- Handle errors with appropriate status codes

**Authentication in API Routes:**
- Create Supabase server client
- Get session via `supabase.auth.getSession()`
- Return 401 if no session for protected routes

### Server Actions

**When to Use:**
- Form submissions
- Data mutations
- Operations that need server-side execution

**Pattern:**
- Create in separate files with `"use server"` directive
- Validate input with Zod
- Revalidate paths after mutations with `revalidatePath()`
- Return typed responses for client handling

### shadcn/ui Component Usage

**Adding Components:**
```bash
npx shadcn@latest add button card form input
```

**Form Pattern:**
- Use `react-hook-form` with `zodResolver`
- Combine with shadcn Form components
- Handle loading and error states

**Styling Approach:**
- Use Tailwind classes for layout and spacing
- Use CSS variables for theming
- Extend components via `cn()` utility for conditional classes

### Error Handling

**Client-Side:**
- Use try/catch with async operations
- Display errors via toast notifications (shadcn Sonner)
- Provide user-friendly error messages

**Server-Side:**
- Log errors for debugging
- Return appropriate HTTP status codes
- Never expose internal errors to clients

### Loading States

**Patterns:**
- Use React Suspense with loading.tsx files
- Show skeleton components during data fetching
- Disable buttons and show spinners during mutations
- Use optimistic updates where appropriate

---

## Claude Agent SDK Patterns

The Claude Agent SDK (`@anthropic-ai/claude-agent-sdk`) is different from the standard Anthropic SDK. It provides **automatic tool execution**, **session management**, and **built-in tools** - you don't need to implement tool loops manually.

### Key Differences from Standard Anthropic SDK

| Feature | Agent SDK | Standard Anthropic SDK |
|---------|-----------|----------------------|
| Tool Execution | Automatic (built-in tools) | Manual (you implement) |
| Tool Loop | Handled by SDK | You must implement |
| Built-in Tools | Yes (Read, Write, Edit, Bash, Glob, etc.) | No |
| Session Management | First-class feature | Must manage manually |
| MCP Support | Native | Limited |

### Core Functions

**`query()`** - For single-session/one-off tasks:
```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

// Simple one-off task - Claude handles tool execution automatically
for await (const message of query({
  prompt: "What files are in this directory?",
  options: {
    allowedTools: ["Bash", "Glob"],
    permissionMode: "bypassPermissions"
  }
})) {
  console.log(message);
}
```

### Creating Custom Tools

Use the `tool()` function with `createSdkMcpServer()` to define custom tools:

**Example: Weather Tool (`lib/ai/tools.ts`):**
```typescript
import { tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

// Define tool with Zod schema
const getWeatherTool = tool(
  "get_weather",
  "Get current temperature for a location using coordinates",
  {
    latitude: z.number().describe("Latitude coordinate"),
    longitude: z.number().describe("Longitude coordinate")
  },
  async (args) => {
    const response = await fetch(
      `https://api.open-meteo.com/v1/forecast?latitude=${args.latitude}&longitude=${args.longitude}&current=temperature_2m`
    );
    const data = await response.json();

    return {
      content: [{
        type: "text",
        text: `Temperature: ${data.current.temperature_2m}°C`
      }]
    };
  }
);

// Create MCP server with custom tools
export const customToolsServer = createSdkMcpServer({
  name: "my-custom-tools",
  version: "1.0.0",
  tools: [getWeatherTool]
});
```

**Using Custom Tools in a Query:**
```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";
import { customToolsServer } from "@/lib/ai/tools";

for await (const message of query({
  prompt: "What's the weather in San Francisco?",
  options: {
    mcpServers: {
      "my-custom-tools": customToolsServer
    },
    allowedTools: ["mcp__my-custom-tools__get_weather"]
  }
})) {
  // Process messages
}
```

### Exa Search Integration

Exa provides neural and keyword search for finding web content, research papers, and articles. It's ideal for AI applications that need to retrieve up-to-date information.

**Setup (`lib/agent/tools.ts`):**
```typescript
import { createSdkMcpServer, tool } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";
import Exa from "exa-js";

// Initialize Exa client
const getExaClient = () => {
  const apiKey = process.env.EXA_API_KEY;
  if (!apiKey) {
    throw new Error("EXA_API_KEY environment variable is not set");
  }
  return new Exa(apiKey);
};

// Create Exa search tools
export const exaSearchTools = createSdkMcpServer({
  name: "exa-search",
  version: "1.0.0",
  tools: [
    // Neural/Keyword Search
    tool(
      "search",
      "Search the web using neural or keyword search. Neural search uses semantic understanding, keyword search matches exact terms.",
      {
        query: z.string().describe("Search query. Natural language for neural, operators (AND/OR/quotes) for keyword"),
        type: z.enum(["neural", "keyword"]).default("neural").describe("Search type"),
        num_results: z.number().min(1).max(20).default(5).describe("Number of results"),
        include_domains: z.array(z.string()).optional().describe("Only include these domains"),
        exclude_domains: z.array(z.string()).optional().describe("Exclude these domains"),
        start_published_date: z.string().optional().describe("Filter: published after (YYYY-MM-DD)"),
        end_published_date: z.string().optional().describe("Filter: published before (YYYY-MM-DD)"),
        use_autoprompt: z.boolean().default(true).describe("Let Exa optimize the query"),
        include_text: z.boolean().default(false).describe("Include text snippets")
      },
      async (args) => {
        const exa = getExaClient();

        const options: any = {
          type: args.type,
          numResults: args.num_results,
          useAutoprompt: args.use_autoprompt
        };

        if (args.include_domains?.length) options.includeDomains = args.include_domains;
        if (args.exclude_domains?.length) options.excludeDomains = args.exclude_domains;
        if (args.start_published_date) options.startPublishedDate = args.start_published_date;
        if (args.end_published_date) options.endPublishedDate = args.end_published_date;
        if (args.include_text) options.contents = { text: { maxCharacters: 1000 } };

        const results = await exa.searchAndContents(args.query, options);

        return {
          content: [{
            type: "text",
            text: JSON.stringify({
              total: results.results.length,
              results: results.results.map((r: any) => ({
                title: r.title,
                url: r.url,
                author: r.author || "Unknown",
                published_date: r.publishedDate || "Unknown",
                text: r.text || null
              }))
            }, null, 2)
          }]
        };
      }
    ),

    // Get full content from URLs
    tool(
      "get_contents",
      "Get full content from specific URLs or search result IDs",
      {
        ids: z.array(z.string()).min(1).max(10).describe("URLs or result IDs to fetch"),
        text_length_words: z.number().min(100).max(2000).default(500).describe("Words to retrieve per document")
      },
      async (args) => {
        const exa = getExaClient();

        const contents = await exa.getContents(args.ids, {
          text: { maxCharacters: args.text_length_words * 5 }
        });

        return {
          content: [{
            type: "text",
            text: JSON.stringify({
              documents: contents.results.map((doc: any) => ({
                url: doc.url,
                title: doc.title,
                author: doc.author || "Unknown",
                text: doc.text
              }))
            }, null, 2)
          }]
        };
      }
    ),

    // Find similar content
    tool(
      "find_similar",
      "Find content similar to a given URL",
      {
        url: z.string().url().describe("URL to find similar content for"),
        num_results: z.number().min(1).max(20).default(5).describe("Number of results"),
        exclude_source_domain: z.boolean().default(false).describe("Exclude results from same domain")
      },
      async (args) => {
        const exa = getExaClient();

        const results = await exa.findSimilar(args.url, {
          numResults: args.num_results,
          excludeSourceDomain: args.exclude_source_domain
        });

        return {
          content: [{
            type: "text",
            text: JSON.stringify({
              source_url: args.url,
              similar: results.results.map((r: any) => ({
                title: r.title,
                url: r.url,
                author: r.author || "Unknown"
              }))
            }, null, 2)
          }]
        };
      }
    )
  ]
});
```

**Using Exa in a Query:**
```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";
import { exaSearchTools } from "@/lib/agent/tools";

for await (const message of query({
  prompt: "Find recent papers on transformer architectures",
  options: {
    mcpServers: {
      "exa-search": exaSearchTools
    },
    allowedTools: [
      "mcp__exa-search__search",
      "mcp__exa-search__get_contents",
      "mcp__exa-search__find_similar"
    ]
  }
})) {
  console.log(message);
}
```

**Exa Search Best Practices:**

| Use Case | Search Type | Tips |
|----------|-------------|------|
| Research papers | `neural` | Use `autoprompt: true`, filter with `include_domains: ["arxiv.org"]` |
| Exact term matching | `keyword` | Use operators: `"exact phrase"`, `term1 AND term2` |
| Recent content | `neural` | Set `start_published_date` to filter by date |
| Deep dive | `get_contents` | Fetch full text after initial search |
| Related work | `find_similar` | Expand research from a key paper |

**Example: Research Agent with Exa:**
```typescript
// lib/agent/config.ts
import { exaSearchTools } from "./tools";

export const researchAgentConfig = {
  model: "claude-sonnet-4-5",
  systemPrompt: `You are a research assistant with access to Exa search.
    Use neural search for semantic queries and get_contents to read full articles.
    Always cite sources with URLs.`,
  mcpServers: {
    "exa-search": exaSearchTools
  },
  allowedTools: [
    "mcp__exa-search__search",
    "mcp__exa-search__get_contents",
    "mcp__exa-search__find_similar",
    "Read",
    "Write"
  ],
  permissionMode: "bypassPermissions"
};
```

### Permission Modes

Control how the SDK handles tool execution:

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

// 1. acceptEdits - Auto-approve file changes (trusted dev workflows)
for await (const message of query({
  prompt: "Fix the bug in auth.ts",
  options: {
    allowedTools: ["Read", "Edit", "Write"],
    permissionMode: "acceptEdits"
  }
})) { /* ... */ }

// 2. bypassPermissions - No prompts (CI/CD, automation)
for await (const message of query({
  prompt: "Run the test suite",
  options: {
    allowedTools: ["Read", "Edit", "Bash"],
    permissionMode: "bypassPermissions"
  }
})) { /* ... */ }

// 3. default - Custom approval handler
for await (const message of query({
  prompt: "Modify the database schema",
  options: {
    permissionMode: "default",
    canUseTool: async (toolName, inputData) => {
      // Block dangerous commands
      if (toolName === "Bash" && inputData.command?.includes("rm -rf")) {
        return false;
      }
      return true;
    }
  }
})) { /* ... */ }
```

### Session Management

Sessions allow Claude to remember context across multiple queries:

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

let sessionId: string | undefined;

// First query - capture session ID
for await (const message of query({
  prompt: "Read the authentication module",
  options: {
    allowedTools: ["Read", "Glob"],
    model: "claude-sonnet-4-5"
  }
})) {
  if (message.type === 'system' && message.subtype === 'init') {
    sessionId = message.session_id;
  }
  console.log(message);
}

// Resume session - Claude remembers context from previous query
for await (const message of query({
  prompt: "Now find all places that call it",
  options: {
    resume: sessionId,
    model: "claude-sonnet-4-5"
  }
})) {
  console.log(message);
}

// Fork session - Creates a new branch to explore different approach
for await (const message of query({
  prompt: "What if we used a different auth strategy?",
  options: {
    resume: sessionId,
    forkSession: true  // Creates new session branch
  }
})) {
  console.log(message);
}
```

### Next.js API Route Integration

**Route Handler (`app/api/agent/route.ts`):**
```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";
import { NextRequest } from "next/server";

export async function POST(request: NextRequest) {
  const { prompt, sessionId } = await request.json();

  const encoder = new TextEncoder();
  const stream = new ReadableStream({
    async start(controller) {
      try {
        for await (const message of query({
          prompt,
          options: {
            allowedTools: ["Read", "Glob", "Grep"],
            permissionMode: "bypassPermissions",
            resume: sessionId,
            model: "claude-sonnet-4-5"
          }
        })) {
          controller.enqueue(
            encoder.encode(`data: ${JSON.stringify(message)}\n\n`)
          );
        }
        controller.enqueue(encoder.encode("data: [DONE]\n\n"));
        controller.close();
      } catch (error) {
        controller.error(error);
      }
    }
  });

  return new Response(stream, {
    headers: {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache",
      "Connection": "keep-alive"
    }
  });
}
```

**Client-Side Hook (`hooks/useAgent.ts`):**
```typescript
import { useState, useCallback } from "react";

interface AgentMessage {
  type: string;
  content?: string;
  session_id?: string;
}

export function useAgent() {
  const [messages, setMessages] = useState<AgentMessage[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [sessionId, setSessionId] = useState<string | undefined>();

  const sendMessage = useCallback(async (prompt: string) => {
    setIsLoading(true);

    const response = await fetch("/api/agent", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ prompt, sessionId })
    });

    const reader = response.body?.getReader();
    const decoder = new TextDecoder();

    while (reader) {
      const { done, value } = await reader.read();
      if (done) break;

      const chunk = decoder.decode(value);
      const lines = chunk.split("\n").filter(line => line.startsWith("data: "));

      for (const line of lines) {
        const data = line.slice(6);
        if (data === "[DONE]") continue;

        const message = JSON.parse(data);
        setMessages(prev => [...prev, message]);

        // Capture session ID for future queries
        if (message.type === "system" && message.session_id) {
          setSessionId(message.session_id);
        }
      }
    }

    setIsLoading(false);
  }, [sessionId]);

  return { messages, isLoading, sessionId, sendMessage };
}
```

### Tool Categories and Access

**Read-only analysis:**
```typescript
allowedTools: ["Read", "Glob", "Grep"]
```

**Code modification:**
```typescript
allowedTools: ["Read", "Edit", "Write", "Glob"]
```

**Full automation:**
```typescript
allowedTools: ["Read", "Edit", "Write", "Bash", "Glob", "Grep"]
```

**Web integration:**
```typescript
allowedTools: ["Read", "Edit", "WebSearch", "WebFetch"]
```

### System Prompts

```typescript
// Custom system prompt
for await (const message of query({
  prompt: "Review this code",
  options: {
    systemPrompt: "You are a senior TypeScript developer. Always suggest improvements."
  }
})) { /* ... */ }

// Use Claude Code's preset system prompt with additions
for await (const message of query({
  prompt: "Fix the bug",
  options: {
    systemPrompt: {
      type: "preset",
      preset: "claude_code",
      append: "Always add type annotations to new functions."
    }
  }
})) { /* ... */ }
```

### Error Handling

```typescript
import {
  query,
  CLINotFoundError,
  ProcessError,
  CLIJSONDecodeError
} from "@anthropic-ai/claude-agent-sdk";

try {
  for await (const message of query({ prompt: "Hello" })) {
    console.log(message);
  }
} catch (error) {
  if (error instanceof CLINotFoundError) {
    console.error("Please install Claude Code: npm install -g @anthropic-ai/claude-code");
  } else if (error instanceof ProcessError) {
    console.error(`Process failed with exit code: ${error.exitCode}`);
  } else if (error instanceof CLIJSONDecodeError) {
    console.error(`Failed to parse response: ${error}`);
  }
}
```

### Model Selection

```typescript
// claude-sonnet-4-5: Balanced performance and cost (recommended for most tasks)
options: { model: "claude-sonnet-4-5" }

// claude-opus-4-5: Complex reasoning and analysis
options: { model: "claude-opus-4-5" }

// claude-haiku-4-5: Fast and cost-effective (good for subagents)
options: { model: "claude-haiku-4-5-20251001" }
```

---

## Multi-Agent Orchestration Pattern

For complex tasks like research, content generation, or multi-step workflows, use an **orchestrator pattern** where a main agent delegates to specialized subagents.

### Architecture Overview

```
User Input
  ↓
Orchestrator Agent (coordinates the pipeline)
  ├→ Stage 1: Planner Subagent (creates plan/queries)
  ├→ Stage 2: Executor Subagent (executes searches/actions)
  └→ Stage 3: Writer Subagent (generates final output)
  ↓
Final Result
```

### Subagent Definitions

Define specialized subagents with specific roles, tools, and models:

```typescript
// lib/agent/subagents.ts
export const PLANNER_SUBAGENT = {
  name: "planner",
  description: "Analyzes the research topic and creates optimized search queries",
  model: "claude-haiku-4-5-20251001",
  tools: [], // No tools - planning only
  systemPrompt: `You are a research planning specialist. Your task is to analyze
a research topic and create a structured search plan.

OUTPUT FORMAT (JSON):
{
  "date_range": {
    "start": "YYYY-MM-DD",
    "end": "YYYY-MM-DD"
  },
  "search_queries": [
    {
      "query": "specific search query",
      "type": "neural" | "keyword",
      "priority": 1-4,
      "rationale": "why this query is important"
    }
  ]
}

Create 4-6 diverse queries covering different aspects of the topic.`
};

export const WEB_SEARCH_SUBAGENT = {
  name: "web-search",
  description: "Executes search plan and gathers sources using Exa",
  model: "claude-haiku-4-5-20251001",
  tools: [
    "mcp__exa-research__search",
    "mcp__exa-research__get_contents"
  ],
  systemPrompt: `You are a web research specialist. Execute the provided search
plan systematically.

PROCESS:
1. Run each search query from the plan
2. Identify the 6 most relevant URLs from results
3. Fetch full content from those URLs using get_contents
4. Return all gathered information

Always complete ALL searches before moving to content fetching.`
};

export const REPORT_WRITER_SUBAGENT = {
  name: "report-writer",
  description: "Creates comprehensive research report from gathered sources",
  model: "claude-haiku-4-5-20251001",
  tools: [], // No tools - writing only
  systemPrompt: `You are a research report writer. Create a comprehensive,
well-structured report from the provided research findings.

OUTPUT FORMAT (Markdown):
## Summary
Brief 2-3 paragraph overview

## Key Findings
- Finding 1 with [source](url)
- Finding 2 with [source](url)

## Detailed Analysis
In-depth discussion organized by theme

## Conclusion
Key takeaways and implications

## References
All sources cited with URLs

Always cite sources inline with markdown links.`
};
```

### Orchestrator Configuration

The orchestrator coordinates subagents and tracks pipeline progress:

```typescript
// lib/agent/config.ts
import { exaSearchTools } from "./tools";

export const ORCHESTRATOR_CONFIG = {
  model: "claude-haiku-4-5-20251001",
  systemPrompt: `You are a research orchestrator managing a 3-stage pipeline.

CRITICAL RULES:
1. Execute stages in EXACT order: Planner → WebSearch → ReportWriter
2. Announce each stage with: "STAGE: [stage-name]"
3. Pass complete data between stages
4. Never skip stages or execute out of order

PIPELINE:
1. STAGE: Planner - Create search plan from topic
2. STAGE: WebSearch - Execute searches, gather sources
3. STAGE: ReportWriter - Generate final report

Start by announcing "STAGE: Planner" and delegating to the planning subagent.`,

  mcpServers: {
    "exa-research": exaSearchTools
  },

  allowedTools: [
    "mcp__exa-research__search",
    "mcp__exa-research__get_contents",
    "Task" // For delegating to subagents
  ],

  disallowedTools: [
    "WebFetch",
    "WebSearch" // Use Exa instead
  ],

  permissionMode: "bypassPermissions"
};
```

### Running the Orchestrated Pipeline

```typescript
// lib/agent/orchestrator.ts
import { query } from "@anthropic-ai/claude-agent-sdk";
import { ORCHESTRATOR_CONFIG } from "./config";
import {
  PLANNER_SUBAGENT,
  WEB_SEARCH_SUBAGENT,
  REPORT_WRITER_SUBAGENT
} from "./subagents";

export async function* runResearchPipeline(topic: string) {
  const subagents = [
    PLANNER_SUBAGENT,
    WEB_SEARCH_SUBAGENT,
    REPORT_WRITER_SUBAGENT
  ];

  for await (const message of query({
    prompt: `Research this topic thoroughly: ${topic}`,
    options: {
      model: ORCHESTRATOR_CONFIG.model,
      systemPrompt: ORCHESTRATOR_CONFIG.systemPrompt,
      mcpServers: ORCHESTRATOR_CONFIG.mcpServers,
      allowedTools: ORCHESTRATOR_CONFIG.allowedTools,
      disallowedTools: ORCHESTRATOR_CONFIG.disallowedTools,
      permissionMode: ORCHESTRATOR_CONFIG.permissionMode,
      subagents
    }
  })) {
    // Detect stage changes from orchestrator announcements
    if (message.type === "assistant" && message.content) {
      const text = typeof message.content === "string"
        ? message.content
        : message.content.find((b: any) => b.type === "text")?.text || "";

      const stageMatch = text.match(/STAGE:\s*(\w+)/i);
      if (stageMatch) {
        yield {
          type: "stage_change",
          stage: stageMatch[1].toLowerCase(),
          timestamp: Date.now()
        };
      }
    }

    yield message;
  }
}
```

### Stage Tracking Best Practices

| Practice | Description |
|----------|-------------|
| **Explicit Announcements** | Orchestrator says "STAGE: X" for parsing |
| **Sequential Execution** | Never parallelize dependent stages |
| **Complete Data Passing** | Pass full context between stages |
| **Subagent Isolation** | Each subagent has specific tools only |
| **Model Selection** | Use haiku for subagents, sonnet for complex orchestration |

---

## SSE Streaming for Agent Pipelines

Real-time progress tracking requires structured Server-Sent Events (SSE) with multiple message types.

### Message Types

Define consistent message types for the frontend:

```typescript
// types/research.ts
export type PipelineStage = "orchestrator" | "planner" | "web-search" | "report-writer";

export interface StageChangeMessage {
  type: "stage_change";
  stage: PipelineStage;
  timestamp: number;
  description?: string;
}

export interface StatusMessage {
  type: "status";
  content: string;
}

export interface ResultMessage {
  type: "result";
  result: string;
  session_id?: string;
}

export interface ErrorMessage {
  type: "error";
  content: string;
}

export type PipelineMessage =
  | StageChangeMessage
  | StatusMessage
  | ResultMessage
  | ErrorMessage;
```

### API Route with Stage Tracking

```typescript
// app/api/research/route.ts
import { NextRequest } from "next/server";
import { runResearchPipeline } from "@/lib/agent/orchestrator";

export const maxDuration = 300; // 5 minutes for long research

export async function POST(request: NextRequest) {
  const { topic } = await request.json();

  const encoder = new TextEncoder();
  const stream = new ReadableStream({
    async start(controller) {
      const sendMessage = (msg: object) => {
        controller.enqueue(
          encoder.encode(`data: ${JSON.stringify(msg)}\n\n`)
        );
      };

      try {
        sendMessage({ type: "status", content: "Starting research pipeline..." });

        for await (const message of runResearchPipeline(topic)) {
          // Forward stage changes
          if (message.type === "stage_change") {
            sendMessage(message);
            continue;
          }

          // Extract text content for final result
          if (message.type === "assistant") {
            const content = message.content;
            if (Array.isArray(content)) {
              for (const block of content) {
                if (block.type === "text") {
                  // Check if this looks like the final report
                  if (block.text.includes("## Summary") ||
                      block.text.includes("## Key Findings")) {
                    sendMessage({
                      type: "result",
                      result: block.text,
                      session_id: message.session_id
                    });
                  }
                }
              }
            }
          }

          // Forward tool usage as status
          if (message.type === "tool_use") {
            sendMessage({
              type: "status",
              content: `Using tool: ${message.name}`
            });
          }
        }

        controller.enqueue(encoder.encode("data: [DONE]\n\n"));
      } catch (error) {
        sendMessage({
          type: "error",
          content: error instanceof Error ? error.message : "Unknown error"
        });
      } finally {
        controller.close();
      }
    }
  });

  return new Response(stream, {
    headers: {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache",
      "Connection": "keep-alive"
    }
  });
}
```

---

## State Management for Agent Pipelines

Complex agent interactions require structured state management on the frontend.

### Pipeline State Hook

```typescript
// hooks/useResearchAgent.ts
import { useState, useCallback, useRef } from "react";
import type { PipelineStage, PipelineMessage } from "@/types/research";

interface StageProgress {
  stage: PipelineStage;
  status: "pending" | "active" | "completed";
  startTime?: number;
  endTime?: number;
}

interface ResearchSource {
  title: string;
  url: string;
  author?: string;
  snippet?: string;
}

interface ResearchState {
  status: "idle" | "researching" | "completed" | "error";
  currentStage: PipelineStage | null;
  stages: StageProgress[];
  sources: ResearchSource[];
  report: string;
  sessionId?: string;
  error?: string;
}

const INITIAL_STAGES: StageProgress[] = [
  { stage: "planner", status: "pending" },
  { stage: "web-search", status: "pending" },
  { stage: "report-writer", status: "pending" }
];

export function useResearchAgent() {
  const [state, setState] = useState<ResearchState>({
    status: "idle",
    currentStage: null,
    stages: INITIAL_STAGES,
    sources: [],
    report: "",
  });

  const abortRef = useRef<AbortController | null>(null);

  const startResearch = useCallback(async (topic: string) => {
    abortRef.current = new AbortController();

    setState(prev => ({
      ...prev,
      status: "researching",
      stages: INITIAL_STAGES,
      sources: [],
      report: "",
      error: undefined
    }));

    try {
      const response = await fetch("/api/research", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ topic }),
        signal: abortRef.current.signal
      });

      const reader = response.body?.getReader();
      const decoder = new TextDecoder();

      while (reader) {
        const { done, value } = await reader.read();
        if (done) break;

        const chunk = decoder.decode(value);
        const lines = chunk.split("\n").filter(l => l.startsWith("data: "));

        for (const line of lines) {
          const data = line.slice(6);
          if (data === "[DONE]") continue;

          const message: PipelineMessage = JSON.parse(data);

          switch (message.type) {
            case "stage_change":
              setState(prev => ({
                ...prev,
                currentStage: message.stage,
                stages: prev.stages.map(s => {
                  if (s.stage === message.stage) {
                    return { ...s, status: "active", startTime: Date.now() };
                  }
                  if (s.status === "active") {
                    return { ...s, status: "completed", endTime: Date.now() };
                  }
                  return s;
                })
              }));
              break;

            case "result":
              setState(prev => ({
                ...prev,
                status: "completed",
                report: message.result,
                sessionId: message.session_id,
                stages: prev.stages.map(s =>
                  s.status === "active"
                    ? { ...s, status: "completed", endTime: Date.now() }
                    : s
                )
              }));
              break;

            case "error":
              setState(prev => ({
                ...prev,
                status: "error",
                error: message.content
              }));
              break;
          }
        }
      }
    } catch (error) {
      if (error instanceof Error && error.name !== "AbortError") {
        setState(prev => ({
          ...prev,
          status: "error",
          error: error.message
        }));
      }
    }
  }, []);

  const cancelResearch = useCallback(() => {
    abortRef.current?.abort();
    setState(prev => ({ ...prev, status: "idle" }));
  }, []);

  const reset = useCallback(() => {
    setState({
      status: "idle",
      currentStage: null,
      stages: INITIAL_STAGES,
      sources: [],
      report: ""
    });
  }, []);

  return {
    ...state,
    startResearch,
    cancelResearch,
    reset
  };
}
```

### Extracting Sources from Tool Results

Parse tool results to extract discovered sources:

```typescript
// Inside useResearchAgent, add source extraction:
const extractSources = (message: any): ResearchSource[] => {
  if (message.type !== "tool_result") return [];

  try {
    const content = JSON.parse(message.content);
    if (content.results) {
      return content.results.map((r: any) => ({
        title: r.title,
        url: r.url,
        author: r.author,
        snippet: r.text?.slice(0, 200)
      }));
    }
  } catch {
    return [];
  }
  return [];
};

// Deduplicate sources by URL
const addSources = (newSources: ResearchSource[]) => {
  setState(prev => {
    const existingUrls = new Set(prev.sources.map(s => s.url));
    const uniqueNew = newSources.filter(s => !existingUrls.has(s.url));
    return {
      ...prev,
      sources: [...prev.sources, ...uniqueNew]
    };
  });
};
```

---

## Vercel Sandbox Integration

**Why Sandbox?** The Claude Agent SDK spawns subprocesses for MCP servers, but Vercel serverless functions cannot spawn subprocesses. Vercel Sandbox provides isolated containers that can.

### When to Use Sandbox

| Environment | Approach |
|-------------|----------|
| **Local Development** | Direct SDK execution (no sandbox needed) |
| **Vercel Production** | Use Vercel Sandbox for agent execution |

### Environment Detection

```typescript
// lib/sandbox/index.ts
export function isVercelEnvironment(): boolean {
  return Boolean(process.env.VERCEL || process.env.VERCEL_ENV);
}
```

### Key Sandbox Pattern

```typescript
// lib/sandbox/runner.ts
import { Sandbox } from "@vercel/sandbox";

export async function runAgentInSandbox(
  topic: string,
  onMessage: (msg: object) => void
) {
  const sandbox = await Sandbox.create({
    runtime: "node22",
    timeout: 300_000, // 5 minutes
    // Auth - set these in Vercel dashboard
    token: process.env.VERCEL_TOKEN,
    projectId: process.env.VERCEL_PROJECT_ID,
    teamId: process.env.VERCEL_TEAM_ID // Optional, for team projects
  });

  try {
    // 1. Write package.json
    await sandbox.writeFile("/vercel/sandbox/package.json", JSON.stringify({
      name: "research-agent",
      type: "module",
      dependencies: {
        "@anthropic-ai/claude-agent-sdk": "^0.1.72",
        "exa-js": "^2.0.11",
        "zod": "^3.25.0"
      }
    }));

    // 2. Write research script (include your agent code here)
    const script = generateResearchScript(topic);
    await sandbox.writeFile("/vercel/sandbox/index.js", script);

    // 3. Install dependencies
    await sandbox.runCommand({
      cmd: "npm",
      args: ["install"],
      cwd: "/vercel/sandbox"
    });

    // 4. Execute agent
    const result = await sandbox.runCommand({
      cmd: "node",
      args: ["index.js"],
      cwd: "/vercel/sandbox",
      env: {
        ANTHROPIC_API_KEY: process.env.ANTHROPIC_API_KEY!,
        EXA_API_KEY: process.env.EXA_API_KEY!
      }
    });

    // 5. Parse output (use markers for structured messages)
    const lines = result.stdout.split("\n");
    for (const line of lines) {
      if (line.startsWith("__MSG__")) {
        const json = line.replace("__MSG__", "");
        onMessage(JSON.parse(json));
      }
    }
  } finally {
    await sandbox.stop();
  }
}

function generateResearchScript(topic: string): string {
  // Escape the topic for safe embedding
  const escapedTopic = topic.replace(/\\/g, "\\\\").replace(/`/g, "\\`");

  return `
// Dynamic research script for sandbox execution
import { query } from "@anthropic-ai/claude-agent-sdk";

const topic = \`${escapedTopic}\`;

// Output helper - prefix messages for parsing
const emit = (msg) => console.log("__MSG__" + JSON.stringify(msg));

async function main() {
  emit({ type: "status", content: "Starting research..." });

  for await (const message of query({
    prompt: \`Research: \${topic}\`,
    options: {
      // Your agent configuration here
      permissionMode: "bypassPermissions"
    }
  })) {
    // Forward relevant messages
    if (message.type === "assistant") {
      emit({ type: "progress", content: message });
    }
  }

  emit({ type: "done" });
}

main().catch(err => {
  emit({ type: "error", content: err.message });
  process.exit(1);
});
`;
}
```

### API Route with Environment Routing

```typescript
// app/api/research/route.ts
import { isVercelEnvironment } from "@/lib/sandbox";
import { runAgentInSandbox } from "@/lib/sandbox/runner";
import { runResearchPipeline } from "@/lib/agent/orchestrator";

export async function POST(request: NextRequest) {
  const { topic } = await request.json();

  const encoder = new TextEncoder();
  const stream = new ReadableStream({
    async start(controller) {
      const sendMessage = (msg: object) => {
        controller.enqueue(encoder.encode(`data: ${JSON.stringify(msg)}\n\n`));
      };

      try {
        if (isVercelEnvironment()) {
          // Production: Use Vercel Sandbox
          await runAgentInSandbox(topic, sendMessage);
        } else {
          // Development: Direct SDK execution
          for await (const msg of runResearchPipeline(topic)) {
            sendMessage(msg);
          }
        }

        controller.enqueue(encoder.encode("data: [DONE]\n\n"));
      } catch (error) {
        sendMessage({ type: "error", content: String(error) });
      } finally {
        controller.close();
      }
    }
  });

  return new Response(stream, {
    headers: {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache"
    }
  });
}
```

### Environment Variables for Sandbox

Add these to your Vercel project settings:

```env
# Required for Vercel Sandbox
VERCEL_TOKEN=your-vercel-api-token
VERCEL_PROJECT_ID=your-project-id
VERCEL_TEAM_ID=your-team-id  # Optional, only for team projects

# Agent API Keys (passed to sandbox)
ANTHROPIC_API_KEY=your-api-key
EXA_API_KEY=your-exa-api-key
```

### Progressive Learning Path

Students can build in stages:

1. **Stage 1**: Build and test locally with direct SDK execution
2. **Stage 2**: Add multi-agent orchestration
3. **Stage 3**: Integrate Vercel Sandbox for production deployment

The environment detection automatically routes to the correct execution method.

### Production Considerations

**Rate Limiting:**
- Debounce user input to prevent rapid requests
- Implement request rate limiting per user
- Track token usage per user/session

**Security:**
- Use `canUseTool` handler to block dangerous operations
- Restrict `allowedTools` to minimum necessary
- Never expose API keys to client

**Cost Management:**
- Choose appropriate model for each task
- Use session resumption to maintain context efficiently
- Monitor API usage via Anthropic console

---

## Best Practices

### TypeScript Configuration

**Strict Mode:**
- Enable `strict: true` in tsconfig.json
- Use proper type annotations
- Avoid `any` type; use `unknown` when type is uncertain

**Path Aliases:**
- Configure `@/*` alias for clean imports
- Organize types in dedicated files
- Export types from index files

### Environment Variable Management

**Validation:**
- Use Zod to validate env vars at startup
- Fail fast if required vars are missing
- Type environment variables properly

**Security:**
- Never expose secret keys to client
- Use `NEXT_PUBLIC_` prefix only for public vars
- Rotate keys regularly
- Use different keys per environment

### Vercel Deployment with Vercel CLI

**Install Vercel CLI:**
```bash
npm install -g vercel
```

**Login to Vercel:**
```bash
vercel login
```

**First Deployment (Project Setup):**
```bash
vercel
```
- Follow prompts to link to existing project or create new
- CLI will detect Next.js and configure build settings automatically

**Production Deployment:**
```bash
vercel --prod
```

**Preview Deployments:**
```bash
vercel
```
- Creates a preview URL for testing before production
- Useful for reviewing changes with team

**Environment Variables via CLI:**
```bash
# Add environment variable
vercel env add ANTHROPIC_API_KEY production

# Pull env vars to local .env file
vercel env pull .env.local

# List all env vars
vercel env ls
```

**Useful CLI Commands:**
```bash
# View deployment logs
vercel logs <deployment-url>

# List recent deployments
vercel ls

# Inspect a deployment
vercel inspect <deployment-url>

# Promote preview to production
vercel promote <deployment-url>

# Rollback to previous deployment
vercel rollback
```

**Deployment Checklist:**
1. Run `vercel env pull` to sync environment variables
2. Test locally with `vercel dev` (uses Vercel's development server)
3. Deploy preview with `vercel` and test
4. Deploy to production with `vercel --prod`
5. Verify Supabase connection with production URL
6. Update OAuth redirect URLs if using social auth

**Database:**
- Run migrations on production Supabase before deploying
- Verify RLS policies are in place
- Test with production data patterns

**Performance:**
- Enable ISR/SSG where appropriate
- Configure caching headers
- Use Edge Runtime for latency-sensitive routes

### Security Considerations

**Authentication:**
- Always verify sessions server-side
- Use HTTPS in production
- Implement CSRF protection
- Set secure cookie options

**Data Access:**
- Never trust client-side data
- Validate all inputs on server
- Use RLS as defense in depth
- Audit data access patterns

**API Security:**
- Rate limit all endpoints
- Validate request origins
- Sanitize user inputs
- Log security-relevant events

**AI-Specific:**
- Validate tool inputs before execution
- Limit tool capabilities to necessary scope
- Monitor for prompt injection attempts
- Review generated content before external actions
