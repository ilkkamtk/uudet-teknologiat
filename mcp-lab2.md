# Building MCP application part 2

## Refining the app

### Timzone issues

Try to add events to the same time slot using different phrases, e.g., "next Wednesday at 5 PM", "tomorrow at 17:00", etc. Notice how the model struggles to interpret these phrases correctly without additional context. Add "if the time is available" to the system prompt to see if it helps.

You probalby will notice that the times are offset by 2 hours. This is because all the times are in UTC, and Finland is UTC+2. Try to fix this by describing the problem to the agent in VS Code. Ask it to save the events to the calendar in local time (Helsinki time) instead of UTC. You can send the timzone through the enponint as json:

```json
{
  "prompt": "Lisää ensi torstaille hammaslääkäri helsingissä kello 17, jos ei ole muita varauksia",
  "timezone": "Europe/Helsinki"
}
```

Ask the modifications to be as simple as possible; agents love to add helper files and huge functions. Adding the timezone info to the tool schema, tool and system prompt should be enough.

### Getting events by range

Add a new function to calendarClient.ts that fetches events by date range. Then create a new tool in `src/mcp-server/index.ts` that uses this function. Finally, update the system prompt to encourage the model to use this new tool when the user asks for events within a specific date range.

In the system prompt, specify the order of preference for tools when fetching events:

1. `getEventsByDateRange`
2. `listEvents`
3. `createEvent`

This will guide the model to use the most appropriate tool based on the user's request.

### Refining the system prompt

- Add explanations why the tools are in certain order.
- Prevent prompt injections
  - Tell the model to use only the provided tools
  - If no tool is suitable, refuse the request
  - Define a short refusal format

AI is a great tool for helping to write prompts. Use it plenty to refine your system prompt for better performance.

## Voice interface

Create an express middleware that converts speech to text using OpenAI's Whisper API. Then, integrate this middleware into your MCP server to handle voice inputs from users. Use the provided frontend code to test the voice interface (public folder).

Example using the Whisper API via our proxy:

```typescript
// save the file with multer

// then read the file with fs

const formData = new FormData();
formData.append('file', audioBlob, file.originalname ?? 'audio');
formData.append('model', process.env.OPENAI_TRANSCRIPTION_MODEL || 'whisper-1');

const transcription = await fetchData<SomeType>(
  openAiProxyUrl + '/v1/audio/transcriptions',
  {
    method: 'POST',
    body: formData,
  },
);

// inject resulted text to the body  (controller is expecting prompt in body)
req.body.prompt = transcription.text;

// continue to the next middleware

// cleanup the file with fs.unlink
```

### Send voice to the frontend

Current frontend uses the builtin JS api to speak the responses. You can try to use OpenAI's TTS models via our proxy to generate more natural-sounding speech. Add that functionality to the `mcpClientController.ts` file.
