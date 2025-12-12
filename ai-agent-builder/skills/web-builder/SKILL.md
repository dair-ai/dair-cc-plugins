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
| AI | Claude Agent SDK (@anthropic-ai/claude-code) | AI agent capabilities |
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
   npm install @anthropic-ai/sdk
   ```

5. **Additional dependencies:**
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
│   ├── ai/
│   │   ├── client.ts      # Anthropic client
│   │   └── tools.ts       # Agent tools
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

### SDK Setup and Configuration

**Initialize the Client (`lib/ai/client.ts`):**
- Import `Anthropic` from `@anthropic-ai/sdk`
- Create singleton instance with API key from environment
- Configure default model (claude-sonnet-4-20250514 recommended for most tasks)

**Model Selection:**
- `claude-sonnet-4-20250514`: Balanced performance and cost for most use cases
- `claude-opus-4-20250514`: Complex reasoning and analysis
- Use environment variables to configure model selection

### Creating Custom Tools

**Tool Definition Structure:**
Tools allow Claude to perform actions and retrieve information. Define tools with:

- `name`: Unique identifier (snake_case)
- `description`: Clear explanation of what the tool does and when to use it
- `input_schema`: JSON Schema defining required parameters

**Tool Categories:**
1. **Data retrieval tools**: Fetch from database, APIs, or external services
2. **Action tools**: Perform operations like sending emails, creating records
3. **Computation tools**: Process data, calculate values, transform information

**Implementation Pattern:**
- Define tool schema separately from execution logic
- Create a tool handler function that maps tool names to implementations
- Validate tool inputs before execution
- Return structured results that Claude can interpret

### Streaming Responses in Next.js

**Why Streaming:**
- Better user experience with immediate feedback
- Reduces perceived latency
- Enables real-time typing effect

**Route Handler Pattern:**
- Use `client.messages.stream()` method
- Return a `ReadableStream` from the route handler
- Set appropriate headers for streaming (`Content-Type: text/event-stream`)

**Client-Side Consumption:**
- Use `fetch` with response body reader
- Parse Server-Sent Events (SSE) format
- Update UI incrementally as chunks arrive
- Handle stream completion and errors

**With Tool Use:**
- Stream may pause during tool execution
- Handle `tool_use` events in the stream
- Execute tools and continue the conversation
- Consider showing tool execution status to users

### Conversation Management

**Message History:**
- Store messages in state or database
- Include both user and assistant messages
- Maintain proper message format for API

**Message Format:**
```typescript
type Message = {
  role: "user" | "assistant";
  content: string | ContentBlock[];
};
```

**Context Management:**
- Set system prompt for consistent behavior
- Include relevant context in system prompt
- Consider summarizing long conversations to manage token usage

**State Patterns:**
- Use React state for ephemeral conversations
- Use Supabase for persistent chat history
- Consider session-based storage for semi-persistent chats

### Multi-Turn Interactions

**Conversation Loop:**
1. Send user message with history
2. Receive assistant response
3. Check for tool use in response
4. If tool use: execute tool, add result, continue
5. If no tool use: display response, await next user input

**Tool Use Loop:**
- Assistant may request multiple tools in sequence
- Each tool result becomes a new message
- Continue until assistant provides final response
- Set reasonable limits on tool iterations

**Handling Tool Results:**
- Format tool results as `tool_result` content blocks
- Include `tool_use_id` to link result to request
- Handle tool errors gracefully with error messages

### Error Handling for AI Operations

**API Errors:**
- Handle rate limits with exponential backoff
- Catch authentication errors and prompt for valid key
- Handle network errors with retry logic

**Content Errors:**
- Handle content filtering responses
- Provide fallback responses for blocked content
- Log issues for review

**Tool Errors:**
- Catch and format tool execution errors
- Return error information to Claude for handling
- Allow Claude to retry or use alternative approaches

**User-Facing Errors:**
- Show friendly error messages
- Provide retry options
- Log detailed errors server-side

### Rate Limiting and Usage Considerations

**Client-Side Throttling:**
- Debounce user input to prevent rapid requests
- Queue requests during high activity
- Show rate limit warnings to users

**Server-Side Controls:**
- Implement request rate limiting per user
- Track token usage per user/session
- Set daily/monthly limits if needed

**Cost Management:**
- Monitor API usage via Anthropic console
- Use caching for repeated queries
- Choose appropriate model for each task
- Consider prompt optimization to reduce tokens

**Production Considerations:**
- Use environment-specific API keys
- Implement usage logging and monitoring
- Set up alerts for unusual usage patterns
- Plan for scaling based on user growth

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
