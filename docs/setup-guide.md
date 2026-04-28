# Setup Guide

This walks you through everything from zero to running `/aeo-master`. If you've never used Claude Code or a terminal before, start at Step 1. If you already have Claude Code running, skip to Step 3.

---

## Step 1: Install Claude Code

Open the Terminal app on your Mac (search "Terminal" in Spotlight, or press Cmd+Space and type "Terminal").

Paste this and press Enter:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Verify it worked:

```bash
claude --version
```

If you see a version number, you're good.

**Account requirement:** You need a paid Anthropic account (Claude Pro at $20/month, Claude Max, Teams, or Enterprise). The free Claude.ai plan does not include Claude Code.

---

## Step 2: Clone This Repo

In Terminal:

```bash
cd ~/Desktop
git clone https://github.com/Telescope-Portfolio/aeo-master.git
cd aeo-master
```

---

## Step 3: Set Up Scrunch (Required)

Scrunch tracks your brand's visibility across AI platforms. You need a Scrunch account with your brand configured and prompts being tracked before running this skill.

**Get your API token:**
1. Log into your Scrunch dashboard
2. Go to Settings > API
3. Copy your API token

**Set the environment variable.** Add this line to your shell profile (`~/.zshrc` on Mac):

```bash
echo 'export SCRUNCH_API_TOKEN="your-token-here"' >> ~/.zshrc
source ~/.zshrc
```

Replace `your-token-here` with your actual token.

---

## Step 4: Set Up Firecrawl (Required)

Firecrawl scrapes your website pages for the content audit.

**Get an API key:**
1. Go to [firecrawl.dev/app](https://firecrawl.dev/app) and sign up (free tier gives 500 credits, no credit card needed)
2. Copy your API key from the dashboard (starts with `fc-`)

**Add it to Claude Code.** In Terminal, run:

```bash
claude mcp add firecrawl -e FIRECRAWL_API_KEY=fc-your-key-here -- npx -y firecrawl-mcp
```

Replace `fc-your-key-here` with your actual key.

---

## Step 5: Set Up Your CRM (Strongly Recommended)

The skill pulls win rates, loss reasons, and deal data from your CRM. This is what makes the priority rankings and content specs specific to your business instead of generic.

### If you use Salesforce

**Install the Salesforce CLI:**

```bash
brew install --cask salesforce-cli
```

If you don't have Homebrew, use npm instead: `npm install -g @salesforce/cli`

**Verify it installed:**

```bash
sf --version
```

**Log in to your Salesforce org:**

```bash
sf org login web --alias my-org --set-default
```

This opens your browser. Log in with your normal Salesforce credentials and authorize the app. Go back to Terminal and wait for the success message (can take 30-45 seconds).

**Verify the connection:**

```bash
sf data query --query "SELECT Id, Name FROM Account LIMIT 5"
```

If that returns a table of account names, you're connected.

**Common issues:**
- "command not found" after install: Close Terminal completely and reopen it
- Using a sandbox? Add `--instance-url https://test.salesforce.com` to the login command
- Queries failing with "not authorized": Your Salesforce profile needs API Enabled permission. Ask your Salesforce admin.

### If you use HubSpot

**Create a Private App token:**
1. Log into HubSpot > Settings (gear icon, top right)
2. Integrations > Private Apps > Create a private app
3. Name it "Claude Code AEO" (or whatever you want)
4. Go to the **Scopes** tab and select:
   - `crm.objects.contacts.read`
   - `crm.objects.companies.read`
   - `crm.objects.deals.read`
5. Click Create app
6. **Copy the access token immediately.** You won't see it again.

**Add it to Claude Code:**

```bash
claude mcp add hubspot -- npx -y @hubspot/mcp-server
```

Then edit `~/.claude/settings.local.json` to add your token:

```json
{
  "mcpServers": {
    "hubspot": {
      "command": "npx",
      "args": ["-y", "@hubspot/mcp-server"],
      "env": {
        "PRIVATE_APP_ACCESS_TOKEN": "pat-na1-your-token-here"
      }
    }
  }
}
```

### If you don't have CRM access

Fill in the "Competitive Win/Loss Data" section of `config/company.md` manually. Ask your sales team for: win rates by segment, top 3-5 reasons you lose deals (with rough percentages), and the exact phrases prospects use on calls. Even rough data is much better than nothing.

---

## Step 6: Set Up Call Recording Access (Strongly Recommended)

The skill mines buyer language from your sales call recordings. This is what makes content specs use actual buyer phrasing instead of marketing guesswork.

### If you use Fathom

This is the easiest setup. Run:

```bash
claude mcp add fathom -- npx mcp-remote@latest https://api.fathom.ai/mcp
```

The first time Claude uses Fathom, it opens an OAuth popup in your browser. Click Allow. That's it.

### If you use Gong

Gong requires an admin to generate API credentials.

**Get API credentials:**
1. You need to be a Gong Technical Administrator (or have one do this for you)
2. In Gong, go to Company Settings > Ecosystem > API > API Keys
3. Click Create to generate an Access Key and Access Key Secret
4. **Copy the secret immediately.** It's only shown once.

**Set environment variables.** Add these to your shell profile:

```bash
echo 'export GONG_ACCESS_KEY="your-access-key"' >> ~/.zshrc
echo 'export GONG_ACCESS_SECRET="your-access-secret"' >> ~/.zshrc
source ~/.zshrc
```

The skill calls the Gong REST API directly using these credentials.

**Required API scopes** (ask your Gong admin to enable):
- `api:calls:read:basic`
- `api:calls:read:extensive`
- `api:calls:read:transcript`
- `api:users:read`

### If you don't have call recording access

The skill will use whatever buyer language you provide in the "Buyer Language" and "Known Buyer Objections" sections of `config/company.md`. Talk to a few sales reps and write down the top 5-10 questions and objections they hear most. That's a solid starting point.

---

## Step 7: Fill Out Your Company Config

Open `config/company.md` in any text editor and fill in the blanks. At minimum you need:

- Company name and website URL
- 2-4 competitors
- Your Scrunch workspace ID
- 3-5 differentiators

If you set up CRM and call recording access in Steps 5-6, the skill pulls strategic data automatically. If you didn't, fill in the "Competitive Win/Loss Data" section manually.

---

## Step 8: Run It

Open the project in Claude Code:

```bash
cd ~/Desktop/aeo-master
claude
```

Then type:

```
/aeo-master
```

The skill will:
1. Run a strategic analysis (pulling CRM and call data if configured)
2. Pull your Scrunch AI visibility baseline
3. Scrape and audit your website via Firecrawl
4. Cross-reference everything and produce the report

Output saves to `outputs/aeo-master-YYYY-MM.md`.

---

## Troubleshooting

**"command not found" after installing anything:** Close Terminal completely (Cmd+Q) and reopen it. New tools don't appear in an already-open Terminal session.

**Claude Code says "Scrunch API token not set":** Make sure you ran the `echo 'export...'` command AND `source ~/.zshrc` afterward. Or close and reopen Terminal.

**Firecrawl scrapes are failing:** Check that your API key is correct and you have credits remaining. Free tier is 500 credits. Each page scrape uses 1 credit.

**Salesforce "not authorized" errors on queries:** Your Salesforce profile needs the "API Enabled" permission. Ask your Salesforce admin to enable it for your user.

**HubSpot returns empty results:** Check that your Private App token has the right scopes (`crm.objects.deals.read`, etc.) and hasn't been deactivated.

**Gong "401 Unauthorized":** Double-check your Access Key and Secret are correct. Make sure the required API scopes are enabled in Gong's admin settings.

**The report is thin or mostly generic:** This usually means Scrunch doesn't have enough prompts configured, or the CRM/call recording data wasn't available. The more data sources connected, the better the output.

---

## What's Connected to What

| Tool | What the skill uses it for | Required? |
|------|---------------------------|-----------|
| **Scrunch** | AI visibility data across 8+ platforms | Yes |
| **Firecrawl** | Scrape website pages for content audit | Yes |
| **Salesforce or HubSpot** | Win rates, loss reasons, deal data | No, but strongly recommended |
| **Gong or Fathom** | Buyer language from call transcripts | No, but strongly recommended |
