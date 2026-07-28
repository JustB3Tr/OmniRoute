# Setup Guide: OmniRoute on Your Machine

> **TL;DR**: Install → set `.env` secrets → first login → connect a provider → create an
> OmniRoute API key → point Claude Code CLI at it. This is the ordered, PC-focused version
> of that flow; see [Quick Start](./QUICK-START.md) for the 3-minute version.

No API keys are required just to install and boot OmniRoute — its own secrets
(`JWT_SECRET`, `API_KEY_SECRET`) are generated locally on first install, not
fetched from anywhere. You only need provider API keys once you want to
_connect_ an AI provider (Step 6), and even that's optional if you pick a
free/no-key provider.

---

## 0. What you need before you start

| Requirement       | Version / notes                                  | Get it                                 |
| ----------------- | ------------------------------------------------ | -------------------------------------- |
| Node.js           | `>=22.22.2 <23` or `>=24.0.0 <27` (see `.nvmrc`) | https://nodejs.org or `nvm install 24` |
| npm               | ships with Node                                  | —                                      |
| git               | any recent version                               | https://git-scm.com                    |
| Docker (optional) | if you'd rather not install Node locally         | https://docs.docker.com/get-docker/    |

---

## 1. Get the code

```bash
git clone https://github.com/diegosouzapw/OmniRoute.git
cd OmniRoute
```

(Already have this repo checked out? Just `cd` into it.)

---

## 2. Install dependencies

```bash
npm install
```

This also runs `postinstall`, which:

- auto-creates `.env` from `.env.example`
- auto-generates `JWT_SECRET` and `API_KEY_SECRET` into it
- installs the husky git hooks

**File you should now have:** `.env` at the repo root (gitignored — never commit it).
Subsequent `npm install` runs will not overwrite an existing `.env`.

---

## 3. Configure `.env`

Open `.env` and check/set these before your first real run:

```bash
# Required secrets — auto-filled by npm install, leave as-is unless regenerating
JWT_SECRET=...
API_KEY_SECRET=...

# Change this before exposing the dashboard to anyone else
INITIAL_PASSWORD=CHANGEME

# Defaults are fine for local use
PORT=20128
REQUIRE_API_KEY=false
```

To regenerate the secrets yourself instead of trusting the auto-fill:

```bash
openssl rand -base64 48   # → JWT_SECRET
openssl rand -hex 32      # → API_KEY_SECRET
```

Full variable reference: [Environment Reference](../reference/ENVIRONMENT.md).

---

## 4. Start it

Pick one:

```bash
# A) From source, dev mode (hot reload)
npm run dev

# B) From source, production build
npm run build && npm start

# C) Skip the clone entirely — install the published CLI globally
npm install -g omniroute
omniroute
```

Dashboard + API are served on the same port: **http://localhost:20128**.

---

## 5. First login

1. Open http://localhost:20128 in a browser.
2. Log in with the password from `INITIAL_PASSWORD` in `.env` (default `CHANGEME`).
3. Change it immediately: **Settings → Security**.

---

## 6. Connect an AI provider

You need at least one connected provider before you can route real requests.
Two paths — pick whichever fits:

### A. Zero API keys (free, no signup)

Dashboard → **Providers** → connect one of:

- **Kiro AI** — Claude Sonnet/Haiku/Opus, ~50 credits/month
- **OpenCode Free** — GPT-4o, Claude, Gemini, unlimited
- **Pollinations** — GPT, Claude, Gemini, DeepSeek, Llama — no key needed
- **Qwen** / **Qoder** — no auth needed

### B. Bring your own provider key

Sign up on the provider's site, generate an API key, then paste it into
**Dashboard → Providers → [provider] → Add Connection**:

| Provider        | Where to get a key            |
| --------------- | ----------------------------- |
| OpenAI          | https://platform.openai.com   |
| Anthropic       | https://console.anthropic.com |
| Google (Gemini) | https://aistudio.google.com   |
| Groq            | https://console.groq.com      |
| DeepSeek        | https://platform.deepseek.com |
| Cerebras        | https://cerebras.ai           |
| NVIDIA NIM      | https://build.nvidia.com      |

Full catalog of every supported provider and its free-tier limits:
[Provider Reference](../reference/PROVIDER_REFERENCE.md) and
[Free Tiers Guide](./FREE-TIERS-GUIDE.md).

---

## 7. Create your OmniRoute API key

This is the key your tools (Claude Code, curl, etc.) use to talk to _your_
OmniRoute instance — not the same thing as the provider keys from Step 6.

**Dashboard → Endpoints → Create API Key** → copy it somewhere safe.

---

## 8. Verify it works

```bash
curl http://localhost:20128/v1/models -H "Authorization: Bearer YOUR_OMNIROUTE_KEY"
```

You should get back a JSON list of models from whatever you connected in Step 6.

---

## 9. Point Claude Code CLI at it

Claude Code has no `--base-url` flag — it reads env vars once at startup, so
restart `claude` after changing any of them.

**Fastest — let OmniRoute do it:**

```bash
omniroute launch                                                  # local OmniRoute, auto-detected
omniroute launch --remote http://<host>:20128 --api-key <key>     # remote / VPS
```

**Or generate one profile per model:**

```bash
omniroute setup-claude              # writes ~/.claude/profiles/<name>/settings.json
omniroute launch --profile glm52    # launch Claude Code using that profile
```

**Or configure `claude` manually** — add this to `~/.claude/settings.json` (or a
profile's `settings.json`):

```jsonc
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:20128", // no /v1 suffix
    "ANTHROPIC_AUTH_TOKEN": "YOUR_OMNIROUTE_KEY",
    "CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY": "1",
  },
}
```

Deeper reference (model tiers, discovery aliases, remote mode):
[Claude Code CLI Configuration](../guides/CLAUDE-CODE-CONFIGURATION.md).

---

## 10. (Optional) Wire up MCP too

```bash
claude mcp add-server omniroute --type http --url http://localhost:20128/api/mcp/stream
```

---

## Quick troubleshooting

- **"Ambiguous model" errors** → pin a prefixed model id, e.g. `ANTHROPIC_MODEL=cc/claude-opus-4-8`.
- **`/model` picker is empty** → needs Claude Code v2.1.219+ and `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1`.
- **Auth errors from Claude Code** → profiles never store the token; use `omniroute launch --profile` or export `ANTHROPIC_AUTH_TOKEN` yourself.
- **Anything else** → [Troubleshooting](./TROUBLESHOOTING.md), or run `omniroute doctor`.
