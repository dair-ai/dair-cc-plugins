---
name: ai-agent-builder
description: Build modern AI-powered web applications using the Claude Agent SDK. Use this skill when creating full-stack apps with Next.js, Supabase, and AI agent capabilities including custom tools, Exa search integration, and agentic workflows.
license: MIT
---

# Web Builder Skill

Use this skill when helping students build modern web applications with AI capabilities. This skill provides guidance for a unified tech stack optimized for building production-ready applications.

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

6. **Additional dependencies:**
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
│   │   └── config.ts      # Agent configuration (allowedTools, permissions)
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
```

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
