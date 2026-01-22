# Building MCP application

## Introduction

Your task is to build an MCP app that helps with calendar events.

What this app does:

1. MCP server (calendar tools)
   - The server exposes tools like “list events” and “create event” over MCP
   - Those tools talk to a calendar backend (CalDAV)

2. MCP client endpoint (model uses the tools)
   - The server can send a user prompt to a chat model
   - The model can call the MCP tools, and the server runs them

3. Browser demo UI (voice in, answer out)
   - The web page records your voice
   - The server transcribes it to text
   - That text is sent to the MCP client endpoint
   - You see the model’s answer, and the browser can read it aloud

You can control the same calendar actions through a chat model or web UI, and MCP is the “tool connection” between the model and your server.

---

## Setup calendar backend

### Radicale (dev CalDAV server)

For local development you can run a minimal Radicale CalDAV server with no authentication.

### macOS / Linux

1. Install Radicale
   - go to a folder where you want to keep the config
   - then run:

   ```bash
   python3 -m pip install --user radicale
   ```

2. Create a local config + data folder

   ```bash
   mkdir -p radicale-data/collections
   cat > radicale.config <<'EOF'
   [server]
   hosts = 127.0.0.1:5232

   [auth]
   type = none

   [storage]
   filesystem_folder = ./radicale-data/collections
   EOF
   ```

3. Run Radicale

   ```bash
   radicale --config ./radicale.config
   ```

### Windows (PowerShell)

1. Install Radicale
   - go to a folder where you want to keep the config
   - then run:

   ```powershell
   py -m pip install --user radicale
   ```

2. Create a local config + data folder

   ```powershell
   New-Item -ItemType Directory -Force -Path .\radicale-data\collections | Out-Null
   @'
   [server]
   hosts = 127.0.0.1:5232

   [auth]
   type = none

   [storage]
   filesystem_folder = ./radicale-data/collections
   '@ | Set-Content -Encoding UTF8 .\radicale.config
   ```

3. Run Radicale

   ```powershell
   py -m radicale --config .\radicale.config
   ```

4. Open `http://localhost:5232/` in your browser to see Radicale is running.
   - Add a calendar with the + button
   - Subscribe to it in your calendar app (e.g., macOS Calendar, Thunderbird, etc.)

---

## Server setup

1. Clone the repo and go to the `mcp-lab-starter` folder:

   ```bash
   git clone https://github.com/ilkkamtk/mcp-lab-starter
   cd mcp-lab-starter
   ```

2. Install dependencies:

   ```bash
   npm install
   ```

3. Rename `.env.sample` to `.env` and set OPENAI_PROXY_URL. URL is in Oma assignment.

4. Start the server:

   ```bash
   npm run dev
   ```

5. Open `http://localhost:3000/` in your browser to check that the server is running.
