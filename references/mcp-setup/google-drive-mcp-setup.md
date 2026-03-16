# Google Drive MCP Setup for Claude Code

**Purpose:** Connect Google Drive, Docs, and Sheets to Claude Code so you can read, create, and edit files directly from your terminal.

**Package:** `@piotr-agier/google-drive-mcp`

## Prerequisites

- Claude Code installed and working
- A Google account with access to your Google Workspace
- Node.js installed (`node --version` to check)
- Access to Google Cloud Console (https://console.cloud.google.com)

## Step 1: Create Google Cloud OAuth Credentials

1. Go to https://console.cloud.google.com
2. Select your GCP project (or create one)
3. Navigate to **APIs & Services > Enabled APIs & services**
4. Enable these three APIs (search for each and click "Enable"):
   - Google Drive API
   - Google Docs API
   - Google Sheets API
5. Navigate to **APIs & Services > Credentials**
6. Click **+ CREATE CREDENTIALS > OAuth client ID**
   - Application type: **Desktop app**
   - Name: `Claude Code MCP`
7. Click Create
8. Copy the **Client ID** and **Client Secret** — you'll need them in Step 2

> If prompted to configure the OAuth consent screen first, set User type to "Internal" and fill in the required fields (app name, support email). Then return to creating credentials.

## Step 2: Create the Credentials File

```bash
mkdir -p ~/.config/google-docs-mcp

cat > ~/.config/google-docs-mcp/credentials.json << 'EOF'
{
  "installed": {
    "client_id": "YOUR_CLIENT_ID_HERE",
    "client_secret": "YOUR_CLIENT_SECRET_HERE",
    "redirect_uris": ["http://localhost"]
  }
}
EOF
```

Replace `YOUR_CLIENT_ID_HERE` and `YOUR_CLIENT_SECRET_HERE` with the values from Step 1.

## Step 3: Add the MCP Server to Claude Code

Run this in your terminal (replace `YOUR_USERNAME` with your macOS username — run `whoami` to find it):

```bash
claude mcp add --scope user google-drive \
  -e GOOGLE_DRIVE_OAUTH_CREDENTIALS=/Users/YOUR_USERNAME/.config/google-docs-mcp/credentials.json \
  -- npx -y @piotr-agier/google-drive-mcp
```

Or manually add to `~/.claude.json`:

```json
{
  "google-drive": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "@piotr-agier/google-drive-mcp"],
    "env": {
      "GOOGLE_DRIVE_OAUTH_CREDENTIALS": "/Users/YOUR_USERNAME/.config/google-docs-mcp/credentials.json"
    }
  }
}
```

## Step 4: Authenticate with Google

1. Quit Claude Code (`/exit`) and reopen it (`claude`)
2. Run `/mcp` to verify `google-drive` appears in the server list
3. The first time you use a Google Drive tool, it will prompt you to authenticate:
   - A URL will appear — open it in your browser
   - Sign in with your Google account
   - Authorize the requested permissions (Drive, Docs, Sheets)
   - Complete the redirect flow

## Step 5: Verify It Works

In Claude Code, try:

```
List the files in my Google Drive root folder
```

You should see your Drive files listed.

## Troubleshooting

**Server shows "failed" in `/mcp`:**
- Run `claude --debug` in a new session and check the error output
- Verify credentials.json exists: `ls ~/.config/google-docs-mcp/credentials.json`
- Verify the path in `~/.claude.json` matches your actual username

**Authentication errors:**
- Make sure all three APIs are enabled in Google Cloud Console (Drive, Docs, Sheets)
- Verify your OAuth credential type is "Desktop app" (not "Web application")
- Try deleting the token and re-authenticating: `rm ~/.config/google-docs-mcp/token.json`

**"npx" not found:**
- Install Node.js: `brew install node`
- Verify: `node --version` and `npx --version`

## What You Can Do With It

Once connected, you can ask Claude Code to:
- **Search:** "Find all documents in my Drive containing 'quarterly report'"
- **Read:** "Read the contents of [Google Doc URL]"
- **Create:** "Create a new Google Doc called 'Meeting Notes' with these bullet points..."
- **Edit:** "Update the Q1 spreadsheet with these new numbers"
- **Organize:** "Move all files from folder X to folder Y"

## Security Notes

- Your OAuth credentials are stored locally at `~/.config/google-docs-mcp/credentials.json`
- Your auth token is stored locally at `~/.config/google-docs-mcp/token.json`
- Neither file should be committed to git or shared
- The MCP server runs locally on your machine — no data passes through third-party servers
