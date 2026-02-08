---
name: reclaim-zkp
description: Generate a production-ready React + Vite + Tailwind ZKP verification app using Reclaim Protocol
user_invocable: true
---

# Reclaim ZKP App Generator

Generate a complete, production-ready React application for zero-knowledge proof verification using Reclaim Protocol's JS SDK.

## Invocation

When the user invokes `/reclaim-zkp`, follow this workflow:

### Step 1: Collect Credentials & Optional Brand URL

Ask the user for the following 5 values (use AskUserQuestion or conversational prompts):

1. **APP_ID** — starts with `0x` followed by 40 hex characters (e.g., `0xFF01cc85cf34cfDE492d70b8AccE5d215690c808`)
2. **APP_SECRET** — starts with `0x` followed by 64 hex characters (e.g., `0x74430a0644d88534889100e843afa6e701a301ed16d5108e61ebad709a340273`)
3. **PROVIDER_ID** — UUID format (e.g., `f9f383fd-32d9-4c54-942f-5e9fda349762`)
4. **Provider Name** — human-readable name (e.g., "Gmail", "GitHub", "Instagram"). Defaults to "Gmail" if not provided.
5. **Target URL** (optional) — a website URL whose brand colors/design the app should match (e.g., `stripe.com`, `google.com`). If not provided, the default cyberpunk theme is used.

Refer to `PROVIDERS.md` in this skill directory for common provider examples to help the user.

### Step 2: Validate Credentials

Validate formats before proceeding:
- `APP_ID`: Must match `/^0x[0-9a-fA-F]{40}$/`
- `APP_SECRET`: Must match `/^0x[0-9a-fA-F]{64}$/`
- `PROVIDER_ID`: Must match `/^[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12}$/`

If validation fails, tell the user which field is invalid and ask them to correct it.

### Step 3: Determine Output Directory

- Default: `reclaim-{provider-name-lowercase}-zkp/` in the current working directory
- Ask the user if they want a different directory name
- Create the directory if it doesn't exist

### Step 3.5: Brand Analysis (only if Target URL was provided)

Skip this step entirely if no Target URL was provided. The default cyberpunk theme will be used.

If a Target URL was provided:

1. **Fetch the target URL** using WebFetch. Prepend `https://` if the user didn't include a protocol.
2. **Extract brand attributes** from the fetched page:
   - **Primary accent color**: Look at buttons, links, highlighted elements, logo colors
   - **Secondary accent color**: If not obvious, derive by shifting the primary hue by ±30°
   - **Background color**: Determine if the site is light or dark mode
   - **Text colors**: Primary text, secondary/muted text
   - **Font families**: If Google Fonts are used, note them. Otherwise use sensible defaults (Inter for sans-serif, JetBrains Mono for mono)
   - **Border radius style**: Sharp (0-2px), moderate (4-8px), or rounded (12px+)
3. **Construct a Brand Theme Object** — map extracted values to the ~23 `@theme` CSS variables (see the `@theme` block in `template/src/index.css` for the full list). Use the primary accent as the base for `--color-neon`, `--color-neon-dim`, `--color-neon-dark`, and all glow variants. Derive background shades from the site's background color.
4. **Handle failures gracefully**: If the URL fetch fails or returns minimal useful data, tell the user and offer two options:
   - Provide 1-2 hex colors manually (primary accent + optional background)
   - Fall back to the default cyberpunk theme

**Color derivation guidelines:**
- `--color-neon`: Primary brand accent (e.g., Stripe purple `#635BFF`)
- `--color-neon-dim`: 80% saturation/brightness of primary
- `--color-neon-dark`: 60% saturation/brightness of primary
- `--color-neon-glow`: Primary at 25% opacity
- `--color-neon-glow-strong`: Primary at 35% opacity
- `--color-neon-hover`: Lighten primary by 15%
- `--color-neon-glow-subtle`: Primary at 6% opacity
- `--color-neon-glow-text`: Primary at 35% opacity
- `--color-neon-glow-bg`: Primary at 10% opacity
- `--color-cyan`: Secondary accent or complementary color
- `--color-bg-primary`: Site background (dark: `#0A0A0F`-ish, light: `#FFFFFF`-ish)
- `--color-bg-secondary`, `--color-bg-card`, `--color-bg-elevated`: Slight variations of bg-primary
- `--color-bg-primary-glass`: bg-primary at 90% opacity
- `--color-bg-primary-30`: bg-primary at 30% opacity
- `--color-border`: Subtle border matching the background tone
- `--color-border-glow`: Border tinted with primary accent
- `--color-text-primary`, `--color-text-secondary`, `--color-text-muted`: Text hierarchy with proper contrast
- `--color-error`, `--color-error-bg`, `--color-error-border`: Keep red-based for errors
- **WCAG contrast**: Ensure text colors have at least 4.5:1 contrast ratio against their backgrounds. For light-mode themes, use dark text on light backgrounds.

**Light-mode adjustments:** If the target site uses a light background:
- Use subtle box-shadows instead of neon glow effects (the `glow-neon` utility will automatically adapt since it uses CSS variables)
- Ensure `--color-text-primary` is dark (e.g., `#1A1A2E`)
- Use lighter, more pastel glow variants instead of vivid neon

### Step 4: Generate the Project

1. **Copy the template**: Copy the entire `template/` directory from this skill folder to the output directory
2. **Create `.env`** in the output directory with the user's credentials:
   ```
   VITE_APP_ID={app_id}
   VITE_APP_SECRET={app_secret}
   VITE_PROVIDER_ID={provider_id}
   VITE_PROVIDER_NAME={provider_name}
   ```
3. **Substitute tokens** in these files:
   - `src/config.js`: Replace `{{PROVIDER_NAME}}` (used as fallback display name)
   - `index.html`: Replace `{{PROVIDER_NAME}}` in the `<title>` tag
   - `package.json`: Replace `{{PROJECT_NAME}}` with the output directory name (e.g., `reclaim-gmail-zkp`)
   - `.env.example`: Replace `{{APP_ID}}`, `{{APP_SECRET}}`, `{{PROVIDER_ID}}`, `{{PROVIDER_NAME}}` with placeholder descriptions
4. **Apply brand theme** (only if Step 3.5 produced a Brand Theme Object):
   - **Rewrite `src/index.css`** in the generated project (NOT the template):
     - Replace the Google Fonts `@import` URL if different fonts were identified
     - Replace all values in the `@theme { }` block with the Brand Theme Object values
     - Replace `--font-mono` and `--font-display` if brand fonts were identified
     - Keep all `@utility`, `@keyframes`, and base styles unchanged — they reference CSS variables and will automatically adapt
   - All JSX components use CSS variable references, so no component files need modification

### Step 5: Install Dependencies

Run `npm install` in the output directory.

### Step 6: Report Success

Tell the user:
```
Project generated at ./{output-dir}/

To start development:
  cd {output-dir}
  npm run dev

The app will be available at http://localhost:5173
```

### Step 7: Optional Deployment

If the user asks for deployment, offer GitHub Pages:

1. `cd {output-dir} && git init && git add -A && git commit -m "Initial commit: Reclaim ZKP verification app"`
2. `gh repo create {output-dir} --public --source=. --push`
3. Add to `package.json` scripts: `"deploy": "vite build && gh-pages -d dist"`
4. `npm install -D gh-pages && npm run deploy`

**Warning**: `.env` is gitignored so credentials won't be committed. However, remind the user that credentials are still embedded in the built JS bundle (client-side app) and visible in browser devtools at runtime.

## Tech Stack

- Vite 7 + @vitejs/plugin-react
- React 19 (JSX, no TypeScript)
- Tailwind CSS v4 (CSS-first config via @theme)
- @reclaimprotocol/js-sdk
- Fonts: JetBrains Mono + Space Grotesk
