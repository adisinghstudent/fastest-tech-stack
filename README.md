# Fastest Tech Stack

Agent skill for creating full-stack applications with the fastest modern stack.

## Stack

| Layer | Technology | Deployment |
|-------|------------|------------|
| Frontend | Next.js + TypeScript + Tailwind | Vercel |
| Backend | Python (serverless functions) | Vercel |
| Database | Supabase (Postgres + Auth) | Supabase Cloud |
| Auth | Google OAuth + Magic Link | Supabase Auth |

## Install

```bash
npx add-skill adisinghstudent/fastest-tech-stack
```

Or install for a specific agent:

```bash
npx add-skill adisinghstudent/fastest-tech-stack -a claude-code
npx add-skill adisinghstudent/fastest-tech-stack -a cursor
npx add-skill adisinghstudent/fastest-tech-stack -a codex
```

## Usage

After installation, use the skill in your agent:

```
/fastest-tech-stack my-app
```

This will:
1. Create a monorepo with Next.js frontend and Python backend
2. Set up Supabase project with auth
3. Deploy to Vercel
4. Provide test commands

## What You Get

```
my-app/
├── api/                      # Python serverless functions
│   ├── health.py             # /api/health endpoint
│   ├── chat.py               # /api/chat (Cerebras AI)
│   └── _lib/                 # Shared code
├── requirements.txt
├── web/                      # Next.js frontend
│   └── src/
│       └── lib/
│           ├── auth.ts       # Google + Magic Link
│           └── supabase/     # SSR client
├── supabase/                 # Migrations & config
└── package.json
```

## Required API Keys

| Service | Get Key |
|---------|---------|
| Supabase | https://supabase.com/dashboard |
| Cerebras | https://cloud.cerebras.ai |
| Exa (optional) | https://exa.ai |
| Google OAuth | https://console.cloud.google.com |

## License

MIT
