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

Implement MCP client endpoint that can send user prompts to a chat model and use the MCP server tools.

Here are the steps to implement the MCP client endpoint. You can write the code yourself or use the instructions in VS Code Copilot Agent mode.

### Add the route

Create an Express router for the MCP client endpoint.

- Create mcpClientRouter.ts
- Use the same style as the existing MCP server router.
- Define `POST /` that calls a controller function named `postPrompt`.
- Default export the router.
- Keep imports consistent with the project (relative import to controller).”

---

### Add the controller (validate input + call client)

Create the controller for the MCP client endpoint.

- Create mcpClientController.ts
- Implement `postPrompt(req, res, next)`:
  - Expect JSON body `{ prompt: string }`
  - If missing/invalid, respond `400` with `{ error: 'Invalid prompt' }`
  - Call `runPromptWithMcpServer(prompt)` from the MCP client module and return its result as JSON
  - Wrap errors with `next(new CustomError(message, 500))`
- Match patterns used in the existing controllers (Express types, `CustomError` usage).

Verification:

- “Start dev server with `npm run dev` and confirm `POST /api/v1/client` returns 400 when body is empty.”

---

### Scaffold the MCP client module (env vars + connect to MCP server)

Create the MCP client module that can talk to the local MCP server over Streamable HTTP.

- Create src/mcp-client/index.ts
- Export an async function `runPromptWithMcpServer(prompt: string)` returning `{ answer: string; toolCalls: number }`
- Read env vars:
  - `MCP_SERVER_URL` (required) e.g. `http://localhost:3000/api/v1/mcp`
  - `OPENAI_PROXY_URL` (required) base URL that serves `/v1/chat/completions`
  - `OPENAI_MODEL` (optional default like `gpt-4o`)
- Create an MCP `Client` and connect with `StreamableHTTPClientTransport` to `MCP_SERVER_URL`
- Make sure transport is always closed in a `finally` block.

---

### Implement the tool-calling loop (Chat Completions → MCP tools)

Extend src/mcp-client/index.ts to implement an iterative tool-calling loop:

- Use `mcpClient.listTools()` to fetch tool definitions and convert them to OpenAI ‘function tool’ format.
- Call `${OPENAI_PROXY_URL}/v1/chat/completions` using the existing `fetchData` helper from fetchData.ts. The return type should match `ChatCompletion` from `openai` package.
- Provide `messages` with:
  - a `system` instruction that:
    - says the assistant must use tools to fulfill calendar actions
    - allows the model to produce ISO 8601 UTC datetimes for tool inputs (since the server tool schema requires ISO strings)
  - the user prompt
- Implement `MAX_ROUNDS` (e.g. 10):
  - If the model returns tool calls, parse JSON arguments, call `mcpClient.callTool`, append tool results back into `messages`
  - Track a `toolCallsCount`
  - If the model returns no tool calls, return `{ answer: message.content ?? '', toolCalls: toolCallsCount }`
- Be defensive:
  - Handle invalid JSON arguments for tool calls by returning a tool-message error string
  - Handle errors from `callTool` similarly
- Keep types compatible with `openai` package types (`ChatCompletion`, `ChatCompletionMessageParam`).”

Verification:

- “Send a simple prompt like `List my events` to `POST /api/v1/client`.”

---

### Postman-ready behavior + response shape

“Make sure the HTTP endpoint is pleasant for Postman testing:

- Confirm `POST /api/v1/client` responds JSON like:
  - `{ "answer": "...", "toolCalls": 2 }`
- Ensure invalid body returns `{ "error": "Invalid prompt" }` with status 400.
- Don’t add extra wrappers or nested fields unless necessary.”

Verification (Postman):

- URL: `http://localhost:3000/api/v1/client`
- Method: `POST`
- Header: `Content-Type: application/json`
- Body:

  ```json
  { "prompt": "Lisää ensi keskiviikolle illallinen helsingissä kello 17" }
  ```

- Expect: tool calls happen, and you get a final natural-language `answer` plus `toolCalls`.

---
