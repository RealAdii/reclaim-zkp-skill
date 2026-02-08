# /reclaim-zkp — Claude Code Skill

A Claude Code skill that generates production-ready React apps for zero-knowledge proof verification using [Reclaim Protocol](https://reclaimprotocol.org).

Type `/reclaim-zkp` in Claude Code, provide your credentials, and get a complete working app in seconds.

## What it generates

A full Vite + React + Tailwind CSS v4 project with:

- Cyberpunk-themed UI (dark mode, neon accents, grid background)
- Reclaim Protocol JS SDK integration
- In-page iframe verification flow (no popups)
- Proof result display with parsed fields
- Collapsible raw JSON viewer
- Credentials stored in `.env` (gitignored, never committed)

## Installation

```bash
# Clone this repo into your Claude Code skills directory
git clone https://github.com/realadii/reclaim-zkp-skill.git ~/.claude/skills/reclaim-zkp
```

Restart Claude Code. The `/reclaim-zkp` command will now be available.

## Usage

1. Get your credentials from [dev.reclaimprotocol.org](https://dev.reclaimprotocol.org):
   - **APP_ID** (0x + 40 hex chars)
   - **APP_SECRET** (0x + 64 hex chars)
   - **PROVIDER_ID** (UUID)

2. In Claude Code, type:
   ```
   /reclaim-zkp
   ```

3. Provide your credentials when prompted.

4. Claude generates the project, runs `npm install`, and you're ready:
   ```bash
   cd reclaim-gmail-zkp  # or whatever provider name
   npm run dev
   ```

5. Open http://localhost:5173

## Generated project structure

```
reclaim-{provider}-zkp/
├── .env                  # Your credentials (gitignored)
├── .env.example          # Placeholder reference
├── .gitignore
├── index.html
├── package.json
├── vite.config.js
└── src/
    ├── main.jsx
    ├── index.css         # Tailwind v4 @theme + custom utilities
    ├── config.js          # Reads from import.meta.env
    ├── App.jsx            # State machine + SDK orchestration
    └── components/
        ├── Header.jsx
        ├── VerifyButton.jsx
        ├── VerificationIframe.jsx
        ├── ProofResult.jsx
        └── RawProofJson.jsx
```

## Tech stack

- Vite 6 + @vitejs/plugin-react
- React 19
- Tailwind CSS v4 (CSS-first config)
- @reclaimprotocol/js-sdk
- JetBrains Mono + Space Grotesk fonts

## Supported providers

Any provider available on [dev.reclaimprotocol.org](https://dev.reclaimprotocol.org) — Gmail, GitHub, Instagram, Twitter/X, LinkedIn, Steam, Spotify, Amazon, and more.

## Security

- Credentials live in `.env` which is gitignored
- No credentials are hardcoded in source files
- Note: since this is a client-side app, credentials are embedded in the built JS bundle at runtime (visible in browser devtools). This is inherent to Reclaim Protocol's client-side SDK design.

## License

MIT
