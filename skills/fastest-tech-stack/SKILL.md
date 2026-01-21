---
name: fastest-tech-stack
description: Full-stack starter kit with Next.js frontend, Python backend, and Supabase. Monorepo on Vercel. The fastest way to ship.
---

# Fastest Tech Stack

Creates a complete full-stack application with:
- **Frontend:** Next.js (TypeScript, Tailwind) → Vercel
- **Backend:** Python (FastAPI-style) → Vercel serverless functions
- **Database:** Supabase (Postgres + Auth + Storage)
- **Structure:** Monorepo (single GitHub repository, single Vercel deployment)

## How to use

- `/fastest-tech-stack <project-name>` — Create and deploy full stack with project name
- `/fastest-tech-stack` — Interactive mode, prompts for project name

## IMPORTANT: Execution Rules

**DO NOT LEAVE THE USER HANGING.** Always complete these steps:

1. **Create all files** (monorepo, api/, web/, supabase/)
2. **Apply migrations** (`bunx supabase db push`)
3. **Create GitHub repo** (`gh repo create`)
4. **Deploy to Vercel** (`vercel --prod`)
5. **Provide testing commands** (curl endpoints, open URLs)

**Package manager:** Always use `bun` and `bunx` — NEVER `npm` or `npx`

---

## Prerequisites

Ensure these CLIs are installed and authenticated:

```bash
# Bun (preferred package manager)
bun --version

# Python 3.9+
python3 --version

# GitHub CLI
gh auth status

# Supabase CLI
bunx supabase --version

# Vercel CLI
vercel --version
```

**Login to services:**
```bash
gh auth login
bunx supabase login
vercel login
```

---

## Configuration

You'll need API keys for these services:

| Service | Get Key | Environment Variable |
|---------|---------|---------------------|
| Supabase | https://supabase.com/dashboard | `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_KEY` |
| Cerebras | https://cloud.cerebras.ai | `CEREBRAS_API_KEY` |
| Exa | https://exa.ai | `EXA_API_KEY` |
| Google OAuth | https://console.cloud.google.com | `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` |

**Cerebras Model:** Use `gpt-oss-120b` (supports tool calling)

---

## Authentication Setup

Default auth providers: **Google OAuth** + **Magic Link (passwordless email)**

### Google OAuth Setup

1. Go to: https://supabase.com/dashboard/project/<PROJECT_REF>/auth/providers
2. Enable **Google** provider
3. Add your Google OAuth credentials
4. Copy **Redirect URL** from Supabase (format: `https://<ref>.supabase.co/auth/v1/callback`)
5. Add redirect URL to GCP Console → APIs & Services → Credentials → OAuth 2.0 Client

### GCP OAuth Setup

In Google Cloud Console → APIs & Services → Credentials → Create OAuth Client ID:

**Authorized JavaScript origins:**
```
http://localhost:3000
http://localhost:54321
https://your-app.vercel.app
```

**Authorized redirect URIs:**
```
http://localhost:54321/auth/v1/callback
https://<PROJECT_REF>.supabase.co/auth/v1/callback
```

### Magic Link (Email)

Enabled by default in Supabase. No configuration needed.

### Local Development (config.toml)

For local dev, add to `supabase/config.toml`:

```toml
[auth]
site_url = "http://localhost:3000"
additional_redirect_urls = ["http://localhost:3000/**"]

[auth.email]
enable_signup = true
double_confirm_changes = true
enable_confirmations = false

[auth.external.google]
enabled = true
client_id = "env(GOOGLE_CLIENT_ID)"
secret = "env(GOOGLE_CLIENT_SECRET)"
redirect_uri = "http://localhost:54321/auth/v1/callback"
```

Create `supabase/.env` for local secrets:
```bash
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"
```

---

## Monorepo Structure

```
$PROJECT_NAME/
├── api/                      # Python serverless functions (Vercel)
│   ├── health.py             # /api/health endpoint
│   ├── chat.py               # /api/chat endpoint
│   └── _lib/                 # Shared code
│       ├── __init__.py
│       ├── supabase.py
│       └── cerebras.py
├── requirements.txt          # Python dependencies
├── web/                      # Next.js frontend (Vercel)
│   ├── src/
│   │   ├── app/
│   │   │   └── auth/
│   │   │       └── callback/
│   │   │           └── route.ts   # OAuth callback
│   │   ├── lib/
│   │   │   ├── config.ts
│   │   │   ├── api.ts
│   │   │   ├── auth.ts            # Google + Magic Link
│   │   │   └── supabase/
│   │   │       ├── client.ts
│   │   │       └── server.ts
│   │   └── types/
│   ├── package.json
│   └── .env.local            # Gitignored
├── supabase/                 # Supabase config & migrations
│   ├── config.toml           # Auth providers config
│   ├── .env                  # Local Google OAuth secrets
│   └── migrations/
├── .gitignore
├── README.md
└── package.json              # Root scripts
```

---

## Execution Steps

### Step 1: Gather Information

Ask the user for:
1. **Project name** (kebab-case, e.g., `my-app`)
2. **GitHub org/user** (default: current user)
3. **Visibility** (public/private)
4. **Supabase org ID** (optional, for auto-creation)

### Step 2: Create Monorepo

```bash
mkdir -p ~/projects/$PROJECT_NAME
cd ~/projects/$PROJECT_NAME

# Initialize git
git init

# Create root package.json for scripts
cat > package.json << 'EOF'
{
  "name": "$PROJECT_NAME",
  "private": true,
  "scripts": {
    "dev": "bun run dev:web",
    "dev:web": "cd web && bun run dev",
    "build": "cd web && bun run build",
    "deploy": "vercel --prod"
  }
}
EOF
```

### Step 3: Create Supabase Project

```bash
cd ~/projects/$PROJECT_NAME

# Create new Supabase project (replace ORG_ID with your org)
bunx supabase projects create $PROJECT_NAME \
  --org-id <YOUR_ORG_ID> \
  --db-password "$(openssl rand -base64 32)" \
  --region us-east-1

# Wait for project to be ready (check status)
bunx supabase projects list

# Initialize local supabase config
bunx supabase init

# Link to the new project (use the ref from projects list)
bunx supabase link --project-ref <PROJECT_REF>
```

**Get credentials:**
```bash
bunx supabase projects api-keys --project-ref <PROJECT_REF>
```

Store these values:
- `SUPABASE_URL` — `https://<ref>.supabase.co`
- `SUPABASE_ANON_KEY` — Public anon key
- `SUPABASE_SERVICE_KEY` — Service role key (backend only)

**Note:** Project creation takes ~2 minutes.

### Step 4: Create Python Backend (Vercel Functions)

Each `api/*.py` file becomes a `/api/*` endpoint automatically.

**requirements.txt** (at project root):
```
httpx>=0.25.0
python-dotenv>=1.0.0
```

**api/health.py:**
```python
from http.server import BaseHTTPRequestHandler
import json

class handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-Type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps({
            "status": "ok",
            "service": "api"
        }).encode())
```

**api/chat.py:**
```python
from http.server import BaseHTTPRequestHandler
import json
import os
from api._lib.cerebras import complete

class handler(BaseHTTPRequestHandler):
    def do_POST(self):
        # Parse request body
        content_length = int(self.headers.get('Content-Length', 0))
        body = json.loads(self.rfile.read(content_length)) if content_length else {}

        message = body.get('message', '')
        if not message:
            self.send_response(400)
            self.send_header('Content-Type', 'application/json')
            self.end_headers()
            self.wfile.write(json.dumps({"error": "message required"}).encode())
            return

        # Call Cerebras
        response = complete(message)

        self.send_response(200)
        self.send_header('Content-Type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps({"response": response}).encode())
```

**api/_lib/__init__.py:**
```python
# Shared library code
```

**api/_lib/config.py:**
```python
import os

# Supabase
SUPABASE_URL = os.environ.get('SUPABASE_URL', '')
SUPABASE_ANON_KEY = os.environ.get('SUPABASE_ANON_KEY', '')
SUPABASE_SERVICE_KEY = os.environ.get('SUPABASE_SERVICE_KEY', '')

# AI APIs
CEREBRAS_KEY = os.environ.get('CEREBRAS_API_KEY', '')
CEREBRAS_MODEL = os.environ.get('CEREBRAS_MODEL', 'gpt-oss-120b')
EXA_KEY = os.environ.get('EXA_API_KEY', '')
```

**api/_lib/cerebras.py:**
```python
import httpx
from api._lib.config import CEREBRAS_KEY, CEREBRAS_MODEL

def complete(prompt: str, model: str = None, api_key: str = None) -> str:
    """Simple completion using Cerebras API."""
    key = api_key or CEREBRAS_KEY
    model = model or CEREBRAS_MODEL

    if not key:
        raise ValueError("CEREBRAS_API_KEY not set")

    response = httpx.post(
        "https://api.cerebras.ai/v1/chat/completions",
        headers={"Authorization": f"Bearer {key}"},
        json={
            "model": model,
            "messages": [{"role": "user", "content": prompt}]
        },
        timeout=30.0
    )
    response.raise_for_status()
    data = response.json()

    return data["choices"][0]["message"]["content"]
```

**api/_lib/supabase.py:**
```python
import httpx
from api._lib.config import SUPABASE_URL, SUPABASE_SERVICE_KEY

def query(table: str, params: dict = None) -> list:
    """Query a Supabase table."""
    url = f"{SUPABASE_URL}/rest/v1/{table}"
    headers = {
        "apikey": SUPABASE_SERVICE_KEY,
        "Authorization": f"Bearer {SUPABASE_SERVICE_KEY}"
    }
    response = httpx.get(url, headers=headers, params=params or {})
    response.raise_for_status()
    return response.json()

def insert(table: str, data: dict) -> dict:
    """Insert into a Supabase table."""
    url = f"{SUPABASE_URL}/rest/v1/{table}"
    headers = {
        "apikey": SUPABASE_SERVICE_KEY,
        "Authorization": f"Bearer {SUPABASE_SERVICE_KEY}",
        "Content-Type": "application/json",
        "Prefer": "return=representation"
    }
    response = httpx.post(url, headers=headers, json=data)
    response.raise_for_status()
    return response.json()
```

**api/_lib/exa.py:**
```python
import httpx
from api._lib.config import EXA_KEY

def search(query: str, num_results: int = 5) -> list:
    """Search with Exa API."""
    if not EXA_KEY:
        raise ValueError("EXA_API_KEY not set")

    response = httpx.post(
        "https://api.exa.ai/search",
        headers={"x-api-key": EXA_KEY},
        json={
            "query": query,
            "numResults": num_results,
            "contents": {"text": {"maxCharacters": 2000}}
        },
        timeout=30.0
    )
    response.raise_for_status()
    return response.json().get("results", [])
```

### Step 5: Create Next.js Frontend

```bash
cd ~/projects/$PROJECT_NAME

# Create Next.js app in web/ directory
bunx create-next-app@latest web \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*" \
  --no-turbopack

cd web
bun add @supabase/supabase-js @supabase/ssr
```

**web/src/lib/config.ts:**
```typescript
export const config = {
  supabase: {
    url: process.env.NEXT_PUBLIC_SUPABASE_URL!,
    anonKey: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
  },
  api: {
    url: process.env.NEXT_PUBLIC_API_URL || '',
  },
} as const;
```

**web/src/lib/supabase/client.ts:**
```typescript
import { createBrowserClient } from '@supabase/ssr';
import { config } from '@/lib/config';

export function createClient() {
  return createBrowserClient(
    config.supabase.url,
    config.supabase.anonKey
  );
}
```

**web/src/lib/supabase/server.ts:**
```typescript
import { createServerClient } from '@supabase/ssr';
import { cookies } from 'next/headers';
import { config } from '@/lib/config';

export async function createClient() {
  const cookieStore = await cookies();

  return createServerClient(
    config.supabase.url,
    config.supabase.anonKey,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll();
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            );
          } catch {
            // Server Component
          }
        },
      },
    }
  );
}
```

**web/src/lib/auth.ts:** (Google + Magic Link helpers)
```typescript
import { createClient } from '@/lib/supabase/client';

const supabase = createClient();

// Google OAuth sign in
export async function signInWithGoogle() {
  const { data, error } = await supabase.auth.signInWithOAuth({
    provider: 'google',
    options: {
      redirectTo: `${window.location.origin}/auth/callback`,
    },
  });
  return { data, error };
}

// Magic link (passwordless email)
export async function signInWithMagicLink(email: string) {
  const { data, error } = await supabase.auth.signInWithOtp({
    email,
    options: {
      emailRedirectTo: `${window.location.origin}/auth/callback`,
    },
  });
  return { data, error };
}

// Sign out
export async function signOut() {
  const { error } = await supabase.auth.signOut();
  return { error };
}

// Get current user
export async function getUser() {
  const { data: { user }, error } = await supabase.auth.getUser();
  return { user, error };
}
```

**web/src/app/auth/callback/route.ts:** (OAuth callback handler)
```typescript
import { createClient } from '@/lib/supabase/server';
import { NextResponse } from 'next/server';

export async function GET(request: Request) {
  const { searchParams, origin } = new URL(request.url);
  const code = searchParams.get('code');
  const next = searchParams.get('next') ?? '/';

  if (code) {
    const supabase = await createClient();
    const { error } = await supabase.auth.exchangeCodeForSession(code);
    if (!error) {
      return NextResponse.redirect(`${origin}${next}`);
    }
  }

  return NextResponse.redirect(`${origin}/auth/error`);
}
```

**web/src/lib/api.ts:**
```typescript
import { config } from '@/lib/config';

class ApiClient {
  private baseUrl: string;

  constructor() {
    this.baseUrl = config.api.url;
  }

  async get<T>(path: string): Promise<T> {
    const res = await fetch(`${this.baseUrl}${path}`, {
      headers: { 'Content-Type': 'application/json' },
    });
    if (!res.ok) throw new Error(`API error: ${res.status}`);
    return res.json();
  }

  async post<T>(path: string, body: unknown): Promise<T> {
    const res = await fetch(`${this.baseUrl}${path}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(body),
    });
    if (!res.ok) throw new Error(`API error: ${res.status}`);
    return res.json();
  }
}

export const api = new ApiClient();
```

**web/.env.example:**
```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...

# Backend API (same Vercel domain, leave empty)
NEXT_PUBLIC_API_URL=
```

### Step 6: Create .gitignore

```bash
cd ~/projects/$PROJECT_NAME

cat > .gitignore << 'EOF'
# Dependencies
node_modules/
web/node_modules/

# Bun
bun.lockb

# Python
__pycache__/
*.pyc
.venv/

# Secrets (NEVER commit)
.env
web/.env.local
supabase/.env

# Build
web/.next/
web/out/

# IDE
.idea/
.vscode/
*.swp

# OS
.DS_Store
Thumbs.db

# Vercel
.vercel/
EOF
```

### Step 7: Create GitHub Repo

```bash
cd ~/projects/$PROJECT_NAME

# Create single repo
gh repo create $PROJECT_NAME --private --source=. --remote=origin --push
```

### Step 8: Deploy to Vercel

Both frontend and backend deploy to Vercel (monorepo setup).

```bash
cd ~/projects/$PROJECT_NAME

# Link project to Vercel
vercel link --yes

# Set environment variables for Python functions
vercel env add SUPABASE_URL production
vercel env add SUPABASE_ANON_KEY production
vercel env add SUPABASE_SERVICE_KEY production
vercel env add CEREBRAS_API_KEY production
vercel env add EXA_API_KEY production

# Set environment variables for Next.js
vercel env add NEXT_PUBLIC_SUPABASE_URL production
vercel env add NEXT_PUBLIC_SUPABASE_ANON_KEY production

# Deploy
vercel --prod
```

**Test the deployed endpoints:**
```bash
# Test Python API
curl https://$PROJECT_NAME.vercel.app/api/health
# Expected: {"status":"ok","service":"api"}

# Test frontend
open https://$PROJECT_NAME.vercel.app
```

### Step 9: Generate Supabase Types

```bash
cd ~/projects/$PROJECT_NAME
bunx supabase gen types typescript --project-id <PROJECT_REF> > web/src/types/database.ts
```

---

## Development Workflow

```bash
cd ~/projects/$PROJECT_NAME

# Run frontend
bun run dev

# Or use Vercel dev (includes Python functions)
vercel dev

# Deploy
bun run deploy
```

---

## Environment Variables Reference

### Backend (Vercel Environment Variables)

| Variable | Description | Required |
|----------|-------------|----------|
| `SUPABASE_URL` | Supabase project URL | Yes |
| `SUPABASE_ANON_KEY` | Supabase public key | Yes |
| `SUPABASE_SERVICE_KEY` | Supabase service key | Yes |
| `CEREBRAS_API_KEY` | Cerebras API key | Yes |
| `CEREBRAS_MODEL` | LLM model | No (default: `gpt-oss-120b`) |
| `EXA_API_KEY` | Exa search API key | Optional |

### Frontend (web/.env.local)

| Variable | Description | Required |
|----------|-------------|----------|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL | Yes |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase public key | Yes |
| `NEXT_PUBLIC_API_URL` | Backend URL (empty for same domain) | No |

---

## CLI Commands Quick Reference

### Root Scripts
```bash
bun run dev        # Run Next.js dev server
bun run build      # Build for production
bun run deploy     # Deploy to Vercel
```

### Vercel CLI
```bash
vercel dev         # Local dev with env vars
vercel --prod      # Deploy production
vercel env pull    # Pull env vars to .env.local
vercel logs        # View function logs
```

### Supabase CLI
```bash
bunx supabase start                    # Local Supabase
bunx supabase db push                  # Push migrations
bunx supabase gen types typescript     # Generate types
```

---

## Quick Code Templates

### Supabase Migration (Chat App)

`supabase/migrations/001_create_chat_tables.sql`:
```sql
-- Conversations
create table public.conversations (
  id uuid default gen_random_uuid() primary key,
  title text,
  user_id uuid references auth.users(id) on delete cascade,
  created_at timestamptz default now() not null,
  updated_at timestamptz default now() not null
);

-- Messages
create table public.messages (
  id uuid default gen_random_uuid() primary key,
  conversation_id uuid references public.conversations(id) on delete cascade not null,
  role text not null check (role in ('user', 'assistant', 'system')),
  content text not null,
  created_at timestamptz default now() not null
);

-- Indexes
create index messages_conversation_id_idx on public.messages(conversation_id);
create index conversations_user_id_idx on public.conversations(user_id);

-- RLS
alter table public.conversations enable row level security;
alter table public.messages enable row level security;

-- Policies (authenticated users only)
create policy "users_own_conversations" on public.conversations
  for all using (auth.uid() = user_id);
create policy "users_own_messages" on public.messages
  for all using (
    conversation_id in (
      select id from public.conversations where user_id = auth.uid()
    )
  );

-- Updated_at trigger
create or replace function handle_updated_at() returns trigger as $$
begin
  new.updated_at = now();
  return new;
end;
$$ language plpgsql;

create trigger on_conversation_updated before update on public.conversations
  for each row execute function handle_updated_at();
```

### Python Chat Endpoints

`api/conversations.py`:
```python
from http.server import BaseHTTPRequestHandler
import json
from api._lib.supabase import query, insert

class handler(BaseHTTPRequestHandler):
    def do_GET(self):
        """List all conversations."""
        conversations = query("conversations", {"order": "created_at.desc"})
        self.send_response(200)
        self.send_header('Content-Type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(conversations).encode())

    def do_POST(self):
        """Create new conversation."""
        content_length = int(self.headers.get('Content-Length', 0))
        body = json.loads(self.rfile.read(content_length)) if content_length else {}
        conversation = insert("conversations", {"title": body.get("title")})
        self.send_response(201)
        self.send_header('Content-Type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(conversation).encode())
```

`api/messages.py`:
```python
from http.server import BaseHTTPRequestHandler
import json
from api._lib.supabase import query, insert
from api._lib.cerebras import complete

class handler(BaseHTTPRequestHandler):
    def do_POST(self):
        """Send message and get AI response."""
        content_length = int(self.headers.get('Content-Length', 0))
        body = json.loads(self.rfile.read(content_length)) if content_length else {}

        conversation_id = body.get('conversation_id')
        content = body.get('content', '')

        if not conversation_id or not content:
            self.send_response(400)
            self.send_header('Content-Type', 'application/json')
            self.end_headers()
            self.wfile.write(json.dumps({"error": "conversation_id and content required"}).encode())
            return

        # Save user message
        insert("messages", {"conversation_id": conversation_id, "role": "user", "content": content})

        # Get AI response
        ai_response = complete(content)

        # Save and return assistant message
        assistant_message = insert("messages", {
            "conversation_id": conversation_id,
            "role": "assistant",
            "content": ai_response
        })

        self.send_response(200)
        self.send_header('Content-Type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(assistant_message).encode())
```

---

## Final Steps Checklist

After setup, run these commands to verify everything works:

### 1. Test Frontend Locally
```bash
cd web
bun run dev
# Open: http://localhost:3000
```

### 2. Deploy to Vercel
```bash
# From project root
vercel --prod
# Set env vars when prompted
```

### 3. Verify Production
```bash
# Test Python API endpoint
curl https://<name>.vercel.app/api/health
# Expected: {"status":"ok","service":"api"}

# Test frontend
open https://<name>.vercel.app
```

---

## Troubleshooting

**Vercel doesn't find Next.js:**
- Check monorepo settings in Vercel dashboard
- Ensure `web/` folder has `package.json`

**CORS errors:**
- Vercel functions don't need CORS config (same domain)
- For external APIs, add headers in the function response

**Supabase connection fails:**
- Verify URL includes `https://`
- Check keys are correct
- Ensure project is not paused

**Vercel Python function fails:**
- Check `requirements.txt` at project root
- Ensure `handler` class exists in each `api/*.py`
- Check Vercel build logs for import errors

**Missing API key errors:**
- Set all required env vars in Vercel dashboard
- Use `vercel env pull` to sync locally
