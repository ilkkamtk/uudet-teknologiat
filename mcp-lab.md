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

5. Don't close the Radicale terminal, keep it running in the background.

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

## Calendar client

Create a [CalDAV](https://en.wikipedia.org/wiki/CalDAV) client that can communicate with the Radicale (or any CalDav) server.

There are several JavaScript/TypeScript libraries for CalDAV. Here we use [tsdav](https://www.npmjs.com/package/tsdav).

1. Open `src/calDav/calendarClient.ts` and implement the calendar client by filling in the TODOs.
2. Test the calendar client by running:

   ```bash
   npm run test
   ```

   - Comment out test that deletes event/events and check your calendar app to see that events are created.
   - To do cleanup, uncomment the "should delete all events" test and run again.

## MCP server tools

Implement MCP server tools that use the calendar client to list and create events.

### Create event tool

1. Install [MCP SDK](https://github.com/modelcontextprotocol/typescript-sdk/tree/v1.x?tab=readme-ov-file#overview) and [Zod](https://github.com/colinhacks/zod?tab=readme-ov-file#what-is-zod) libraries:

   ```bash
   npm install @modelcontextprotocol/sdk zod
   ```

   - Current version of MCP SDK is v1.x.
   - [New version should be released in Q1 of 2026](https://github.com/modelcontextprotocol/typescript-sdk?tab=readme-ov-file#mcp-typescript-sdk).
   - Zod is used to define and validate tool input/output schemas.

2. Create new file `src/mcp-server/index.ts`.

3. Create the first tool schema for creating calendar event in `src/mcp-server/index.ts`:

   ```typescript
   import { z } from 'zod';

   // tool schemas
   const createEventInputSchema = z.object({
     start: z.iso
       .datetime()
       .describe(
         'Start date and time of the event as ISO 8601 datetime (YYYY-MM-DDTHH:MM:SSZ) in UTC',
       ),
     end: z.iso
       .datetime()
       .optional()
       .describe(
         'End date and time of the event as ISO 8601 datetime (YYYY-MM-DDTHH:MM:SSZ) in UTC. If not provided, defaults to one hour after start time',
       ),
     title: z.string().describe('Short title of the event'),
     description: z
       .string()
       .optional()
       .describe('Optional detailed description'),
     location: z.string().optional().describe('Optional location of the event'),
   });

   // input types
   type CreateEventInput = z.infer<typeof createEventInputSchema>;
   ```

4. Implement the create event tool. Add the following to `src/mcp-server/index.ts`:

   ```typescript
   import { createEvent, getEventByUrl } from '@/calDav/calendarClient';
   import { icsToJson } from '@/utils/ics-to-json';
   import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';

   const mcpServer = new McpServer({
     name: 'calendar-server',
     version: '1.0.0',
   });

   mcpServer.registerTool(
     'createEvent',
     {
       title: 'Create a new Calendar Event',
       description:
         'Creates a new event in the calendar with the provided details.',
       inputSchema: createEventInputSchema,
     },
     async (input: CreateEventInput) => {
       const { start, end, title, description, location } = input;
       const eventEnd = end
         ? new Date(end)
         : new Date(new Date(start).getTime() + 60 * 60 * 1000); // Default to one hour after start if end is not provided
       try {
         // Call the calendar client to create the event
         const eventUrl = await createEvent({
           start: new Date(start),
           end: eventEnd,
           title,
           description,
           location,
         });
         const newEvent = await getEventByUrl(eventUrl);
         const newEventObject = icsToJson(newEvent.data);
         const eventSummaryStart = newEventObject[0]?.startDate ?? start;
         const eventSummaryEnd =
           newEventObject[0]?.endDate ?? end ?? eventEnd.toISOString();

         console.log([
           {
             type: 'text',
             text: `Event created successfully for "${title}" from ${eventSummaryStart} to ${eventSummaryEnd}.`,
           },
         ]);

         return {
           content: [
             {
               type: 'text',
               text: `Event created successfully for "${title}" from ${eventSummaryStart} to ${eventSummaryEnd}.`,
             },
           ],
         };
       } catch (error) {
         console.error('Error creating event:', error);
         return {
           content: [
             {
               type: 'text',
               text: `Failed to create event: ${(error as Error).message}`,
             },
           ],
         };
       }
     },
   );

   export { mcpServer };
   ```

5. Controller for MCP server. Modify `src/api/v1/controllers/mcpServerController.ts` to act as MCP server endpoint:

   ```typescript
   import { StreamableHTTPServerTransport } from '@modelcontextprotocol/sdk/server/streamableHttp.js';
   import { NextFunction, Request, Response } from 'express';
   import CustomError from '@/classes/CustomError';
   import { mcpServer } from '@/mcp-server';

   const postMcp = async (req: Request, res: Response, next: NextFunction) => {
     try {
       const transport = new StreamableHTTPServerTransport({
         sessionIdGenerator: undefined,
         enableJsonResponse: true,
       });

       res.on('close', () => transport.close());

       await mcpServer.connect(transport);
       await transport.handleRequest(req, res, req.body);
     } catch (error) {
       next(new CustomError((error as Error).message, 500));
     }
   };

   export { postMcp };
   ```

   - There are two ways to expose MCP servers:
     - Streamable HTTP for remote/web based servers (like Express)
     - STDIO for local/CLI based servers running as subprocesses.

6. Check if MCP server is offering the tools correctly with MCP inspector:

   ```bash
   npx @modelcontextprotocol/inspector
   ```

7. Test MCP server by adding the server url to host e.g. Copilot in VSCode.
   - Use `http://localhost:3000/api/v1/mcp` as MCP server URL.
   - Open command palette (Ctrl+Shift+P or Cmd+Shift+P) and type "MCP: Add Server".
   - Choose "Add MCP Server" and enter the URL.
   - In agent mode, try commands like "Create a calendar event titled 'Team Meeting' on [date] at [time]". _Note that MMLs don't know what time it is, so if you say `tomorrow` or `next week`, you need to tell what day it is today in the same prompt_.
   - Check Radicale calendar to see if event was created.
   - Use the delete tests with `npm test` to clean up events.

### List events tool

1. Use similar steps as with create event tool to implement list events tool.
   - Define tool schema for listing events. In this case inputSchema is and empty object `{}` since no input is needed.
   - Implement the tool `listEvents` using calendar client to fetch events.
     - mind the `description` property to clearly explain what the tool does.

2. Test the list events tool via MCP inspector and VSCode Copilot.
   - Use commands like "List all calendar events for today".
   - Verify the listed events match those in Radicale calendar.

3. Ensure both tools (create and list events) work correctly together by trying to create an event to a time already occupied by another event and see how the model handles it.

## MCP client endpoint

Implement an MCP client endpoint that accepts a JSON prompt, sends it to a chat model (via an OpenAI-compatible proxy), and lets the model call your MCP server tools (`createEvent`, `listEvents`) over MCP Streamable HTTP. The endpoint should be usable from Postman.

### Using AI Agent (VSCode Copilot Agent)

Scroll down for code-it-yourself instructions.

Use the instructions below VSCode's Copilot Agent to generate the code for you step-by-step.

Paste these into Copilot **Agent mode** one at a time.

---

#### Prompt 1 — Add a new `/client` API endpoint (router + mount)

“Add an MCP client HTTP endpoint under the existing v1 API.

- Create a new router file at mcpClientRouter.ts.
- Mount it under `/api/v1/client` in index.ts (follow the same style as how `/mcp` is mounted).
- Keep code style consistent with the other routers in this project.”

Quick check: run `npm run build`.

---

#### Prompt 2 — Add the controller for Postman JSON requests

“Implement the controller used by the `/api/v1/client` route.

- Create mcpClientController.ts.
- This endpoint must accept JSON like `{ "prompt": "..." }`.
- Validate the request body and return `400` with a small JSON error if it’s missing/invalid.
- On success, call the MCP client module (we’ll implement it next) and return its result as JSON.
- Use the same error handling approach as the other controllers (CustomError + next()).”

Quick check: start `npm run dev` and confirm `POST /api/v1/client` with `{}` returns 400.

---

#### Prompt 3 — Create the MCP client module skeleton (env + connect)

“Create an MCP client module that can call our MCP server over Streamable HTTP.

- Create index.ts.
- Add a function that takes a user prompt string and returns `{ answer, toolCalls }`.
- Read required env vars (`MCP_SERVER_URL`, `OPENAI_PROXY_URL`) and fail fast with clear errors when missing.
- Connect to the MCP server using the MCP SDK Streamable HTTP client transport.
- Make sure the transport is always closed even on errors.”

Quick check: `npm run build`.

---

#### Prompt 4 — Call the chat completions endpoint via `fetchData` (typed, no `any`)

“Extend index.ts so it calls the chat completions endpoint on our proxy:

- Use the existing helper in fetchData.ts to POST to `${OPENAI_PROXY_URL}/v1/chat/completions`.
- Keep TypeScript strict: avoid `any` in this module.
- Don’t rely on OpenAI SDK type imports (they’re fragile across versions); instead, define the minimal response/request/message types we actually use inside this file.
- Accept `OPENAI_MODEL` env var with a sensible default.”

Quick check: `npm run build` (this should catch most typing issues immediately).

---

#### Prompt 5 — Add a strong system prompt for Finnish calendar commands (tool-first)

“Improve the `messages` setup in index.ts:

- Add a detailed system message that makes the assistant behave like a Finnish calendar assistant.
- It must prefer using MCP tools for calendar actions:
  - If the user wants to list events → call `listEvents`.
  - If the user wants to create an event → call `createEvent`.
- It must interpret Finnish phrases like ‘ensi keskiviikko’, ‘huomenna’, ‘kello 17’, and ‘Helsingissä’.
- It must convert Europe/Helsinki local intent into UTC ISO 8601 strings for the `createEvent` tool inputs.
- Default event duration to 1 hour if end time isn’t given.
- After tool calls, the assistant’s final answer must be based only on tool output (no pretending success).”

Quick check: none yet—this is behavior-focused.

---

#### Prompt 6 — Implement the tool-calling loop (multi-round)

“Implement tool calling in index.ts:

- Fetch tool definitions from the MCP server with `mcpClient.listTools()` and expose them to chat completions as function tools.
- Run a multi-round loop with a max round limit to prevent infinite tool calls.
- If the model returns tool calls:
  - Parse tool arguments safely (handle invalid JSON gracefully).
  - Call the MCP tool using `mcpClient.callTool(...)`.
  - Convert the tool result into a string and append it back to the message history as a tool message.
  - Count tool calls.
- If the model returns no tool calls, return the final assistant content as `{ answer, toolCalls }`.
- Keep this module strictly typed end-to-end (no `any`).”

Quick check: `npm run build`.

---

#### Prompt 7 — Wire controller to the client module cleanly

“Update mcpClientController.ts to call the MCP client function you implemented and return `{ answer, toolCalls }` exactly.

- Keep validation and error handling simple and consistent with the rest of the API.”

Quick check: `npm run build` + `npm run dev`.

---

#### Prompt 8 — Postman test + small polish pass

“Do a small polish pass for real Postman usage:

- Confirm `POST /api/v1/client` accepts `{ "prompt": "Lisää ensi keskiviikolle illallinen helsingissä kello 17" }`.
- Response should be JSON `{ answer: string, toolCalls: number }`.
- Ensure error cases are readable (missing env vars, bad body, proxy down) and don’t leak huge stacks in production mode.”

Manual Postman test:

- URL `http://localhost:3000/api/v1/client`
- Method `POST`
- Header `Content-Type: application/json`
- Body `{ "prompt": "Lisää ensi keskiviikolle illallinen helsingissä kello 17" }`

---

## Code-it-yourself instructions

Step-by-step instructions to implement the MCP client endpoint yourself.

### 1) Add the route

Goal: create `POST /api/v1/client`.

- “Create mcpClientRouter.ts.
  - Use Express Router.
  - Add `router.post('/', postPrompt)`.
  - Import `postPrompt` from `../controllers/mcpClientController`.
  - Default-export the router.”

---

### 2) Add the controller (validate input + call MCP client)

Goal: validate `{ prompt }`, call the MCP client module, and return JSON.

- “Create mcpClientController.ts.
  - Implement `postPrompt` as `async (req: Request<unknown, unknown, { prompt: string }>, res: Response, next: NextFunction)`.
  - Read `prompt` from `req.body`.
  - If `prompt` is missing or not a string, respond `400` with `{ error: 'Invalid prompt' }`.
  - Call `runPromptWithMcpServer(prompt)` from `@/mcp-client` and `return res.json(result)`.
  - On errors: `next(new CustomError((error as Error).message, 500))`.”

Verification:

- Start dev server with `npm run dev`.
- Send an empty JSON body to `POST /api/v1/client` and confirm you get `400`.

---

### 3) Implement the MCP client module (strong types, no OpenAI SDK types)

Goal: create a tool-calling loop without `any` and without relying on OpenAI SDK type imports.

- “Create index.ts exporting:
  - `runPromptWithMcpServer(prompt: string): Promise<{ answer: string; toolCalls: number }>`
  - Read env vars and throw if missing:
    - `MCP_SERVER_URL`
    - `OPENAI_PROXY_URL`
    - `OPENAI_MODEL` (optional, default `gpt-4o`)
  - Connect to MCP server using `StreamableHTTPClientTransport` + `Client`.
  - Always close the transport in a `finally` block.
  - Do NOT import OpenAI SDK types. Instead, define minimal local TS types:
    - `ChatRole = 'system' | 'user' | 'assistant' | 'tool'`
    - `ToolCall` for `{ id, type: 'function', function: { name: string; arguments: string } }`
    - `ChatMessageParam` union:
      - system/user/assistant messages with `content: string | null` and optional `tool_calls?: ToolCall[]`
      - tool messages with `tool_call_id: string` and `content: string`
    - `ChatCompletionResponse` with `choices: Array<{ message: ... }>`
    - `ChatCompletionRequest` with `model`, `messages`, optional `tools`, `tool_choice`
  - Use `fetchData<ChatCompletionResponse>(\`\${OPENAI_PROXY_URL}/v1/chat/completions\`, ...)`so the response is typed (no`any`).”

---

### 4) Add the system prompt (detailed + matches your actual tools)

Goal: make the model reliably call `createEvent` / `listEvents`, and produce ISO datetimes for tool args.

- “In index.ts, build `messages: ChatMessageParam[]` starting with a detailed `system` message and then `{ role: 'user', content: prompt }`.
  The system message must instruct:
  - You are a calendar assistant for Finnish user requests.
  - Tools available: `createEvent` and `listEvents`.
  - Tool usage rules:
    - For listing requests → call `listEvents` first.
    - For creating requests → call `createEvent`.
    - You must use tools for calendar actions (don’t ‘pretend’).
  - Date/time handling:
    - Interpret phrases: ‘next week’, ‘this friday’, ‘tomorrow’, ‘at 17’, etc.
    - Assume user times are Europe/Helsinki local time.
    - Convert to UTC ISO 8601 strings for `createEvent.start` and `createEvent.end`.
    - If end time not given → default duration 1 hour.
  - Field mapping:
    - `title`: short (e.g. ‘Dinner’)
    - `location`: set if a place is mentioned (e.g. Helsinki)
    - `description`: optional extra details
  - After tool results:
    - Reply briefly based strictly on tool output.
    - Do not claim success unless tool output confirms it.”

---

### 5) Implement the tool-calling loop (MAX_ROUNDS, defensive parsing, typed messages)

Goal: iterate until the model stops requesting tools.

- “In index.ts, implement:
  - `const { tools } = await mcpClient.listTools()` and map to OpenAI function-tools:
    - `type OpenAIFunctionTool = { type: 'function'; function: { name: string; description?: string; parameters: unknown } }`
  - `MAX_ROUNDS = 10`
  - Loop:
    1. Call chat completions with `{ model, messages, tools, tool_choice: 'auto' }`
    2. Push the assistant message into `messages` (typed)
    3. If no `tool_calls`, return `{ answer: message.content ?? '', toolCalls: toolCallsCount }`
    4. For each tool call:
       - Parse `call.function.arguments`:
         - parse to `unknown`
         - if it’s an object use it as `Record<string, unknown>`, otherwise use `{}` and return a tool error message
       - Call `mcpClient.callTool({ name: call.function.name, arguments })`
       - Convert MCP tool result into a single string (join `text` parts + stringify `structuredContent` if present)
       - Push a tool message `{ role: 'tool', tool_call_id: call.id, content: '...' }` into `messages`
  - Keep a `toolCallsCount` number.
  - Avoid returning `null` from the tool call map (keep arrays strongly typed).”

---

### 6) Postman-ready behavior + response shape

Goal: simple JSON response for testing.

- `POST /api/v1/client` responds:
  - `{ "answer": "...", "toolCalls": 2 }`
- Invalid body responds:
  - status `400`, `{ "error": "Invalid prompt" }`

Verification (Postman):

- URL: `http://localhost:3000/api/v1/client`
- Method: `POST`
- Header: `Content-Type: application/json`
- Body:

  ```json
  { "prompt": "Lisää ensi keskiviikolle illallinen helsingissä kello 17" }
  ```
