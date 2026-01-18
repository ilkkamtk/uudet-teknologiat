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

### Install npm packages

1. Navigate to desired project folder

2. Create `package.json`

   ```bash
   npm init -y
   ```

3. Install the first runtime packages (web server basics)

   ```bash
   npm install express cors helmet morgan dotenv
   ```

4. Install the first dev packages (TypeScript + dev server + formatting)

   ```bash
   npm install -D typescript ts-node tsconfig-paths tsc-alias nodemon prettier
   ```

Other packages (MCP SDK, CalDAV, OpenAI, uploads, validation, etc.) will be added later when we need them.

### Example npm scripts

These are the scripts we will use later:

```json
{
  "scripts": {
    "start": "node dist/index.js",
    "dev": "nodemon --exec ts-node -r tsconfig-paths/register src/index.ts",
    "build": "tsc && tsc-alias"
  }
}
```

### Init ESLint

1. Create a config

   ```bash
   npm init @eslint/config@latest
   ```

### TSConfig

Create a `tsconfig.json` file with the following content:

```json
{
  "compilerOptions": {
    "outDir": "dist",
    "sourceMap": true,
    "target": "esnext",
    "module": "commonjs",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "noImplicitAny": true,
    "strict": true,
    "skipLibCheck": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["./*.js", "src/**/*.ts", "test/**/*.ts"]
}
```

### .env file

Create a `.env` file in the project root with the following content:

```text
OPENAI_PROXY_URL=get-this-from-Oma
OPENAI_MODEL=gpt-4.1
MCP_SERVER_URL=http://localhost:3000/api/v1/mcp
DEBUG_MCP_CLIENT=1
```

## Express server boilerplate

Create a basic Express server in `src/app.ts`:

```typescript
import dotenv from 'dotenv';
dotenv.config();
import express from 'express';
import morgan from 'morgan';
import helmet from 'helmet';
import cors from 'cors';
import api from './api/v1';
import { errorHandler, notFound } from './middlewares';

const app = express();

const isDevelopment = process.env.NODE_ENV === 'development';

app.use(morgan('dev'));
app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: isDevelopment
        ? ["'self'", "'unsafe-eval'"] // unsafe-eval is needed for Apidoc in development
        : ["'self'"],
    },
  }),
);
app.use(cors());
app.use(express.json());

// serve public folder for static files
app.use(express.static('public'));

app.use('/api/v1', api);

app.use(notFound);
app.use(errorHandler);

export default app;
```

And the server entry point in `src/index.ts`:

```typescript
import app from './app';

const port = process.env.PORT || 3000;
app.listen(port, () => {
  console.log(`Listening: http://localhost:${port}`);
});
```

### Middlewares

Create basic middlewares in `src/middlewares.ts`:

```typescript
import { NextFunction, Request, Response } from 'express';
import { ErrorResponse } from './types/LocalTypes';
import CustomError from './classes/CustomError';

const notFound = (req: Request, res: Response, next: NextFunction) => {
  const error = new CustomError(`🔍 - Not Found - ${req.originalUrl}`, 404);
  next(error);
};

const errorHandler = (
  err: CustomError,
  req: Request,
  res: Response<ErrorResponse>,
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  next: NextFunction,
) => {
  // console.log(err);
  const statusCode = err.status && err.status >= 400 ? err.status : 500;
  res.status(statusCode).json({
    message: err.message,
    stack: process.env.NODE_ENV === 'production' ? '🥞' : err.stack,
  });
};

export { notFound, errorHandler };
```

### Endpoints structure

1. Create the API router in `src/api/v1/index.ts`:

   ```typescript
   import express, { Request, Response } from 'express';

   import mcpServerRouter from './routes/mcpServerRouter';
   import mcpClientRouter from './routes/mcpClientRouter';

   const router = express.Router();

   router.get('/', (req: Request, res: Response) => {
     res.json({
       message: 'media api v1',
     });
   });

   router.use('/mcp', mcpServerRouter);
   router.use('/client/', mcpClientRouter);

   export default router;
   ```

2. Create route files for MCP server and client in `src/api/v1/routes/mcpServerRouter.ts` and `src/api/v1/routes/mcpClientRouter.ts` respectively.
   - `src/api/v1/routes/mcpServerRouter.ts`:

   ```typescript
   import express from 'express';
   import { postMcp } from '../controllers/mcpServerController';

   const router = express.Router();

   router.route('/').post(postMcp);

   export default router;
   ```

   - `src/api/v1/routes/mcpClientRouter.ts`:

   ```typescript
   import express from 'express';

   const router = express.Router();

   router
   .route('/'), (req: express.Request, res: express.Response) => {
     res.json({ message: 'MCP Client Endpoint in the future' });
   });

   export default router;
   ```

3. Create controller for MCP server in `src/api/v1/controllers/mcpServerController.ts`:

   ```typescript
   import { NextFunction, Request, Response } from 'express';
   import CustomError from '@/classes/CustomError';

   const postMcp = async (req: Request, res: Response, next: NextFunction) => {
     try {
       res.json({ message: 'MCP Server Endpoint - not implemented yet' });
     } catch (error) {
       next(new CustomError((error as Error).message, 500));
     }
   };

   export { postMcp };
   ```
