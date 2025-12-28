# Step 3: Ink CLI/TUI Setup

Set up an Ink-based terminal UI app with React and TypeScript.

## 3.1 Create package.json

Create `apps/ink/package.json`:

```json
{
  "name": "@a-learning-machine/ink",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "main": "dist/cli.js",
  "bin": {
    "alm": "./dist/cli.js"
  },
  "scripts": {
    "dev": "tsx watch src/cli.tsx",
    "build": "tsup src/cli.tsx --format esm --dts",
    "typecheck": "tsc --noEmit",
    "lint": "eslint ."
  },
  "dependencies": {
    "ink": "^5.1.0",
    "ink-spinner": "^5.0.0",
    "ink-text-input": "^6.0.0",
    "ink-select-input": "^6.0.0",
    "react": "^18.3.1",
    "zustand": "^5.0.2",
    "commander": "^14.0.0",
    "dotenv": "^16.4.5"
  },
  "devDependencies": {
    "@types/react": "^18.3.12",
    "tsup": "^8.3.5",
    "tsx": "^4.19.2",
    "typescript": "^5.7.2",
    "eslint": "^9.0.0"
  }
}
```

## 3.2 Create tsconfig.json

Create `apps/ink/tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "Bundler",
    "jsx": "react-jsx",
    "outDir": "dist",
    "strict": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "declaration": true
  },
  "include": ["src"]
}
```

## 3.3 Create the CLI entry point

Create `apps/ink/src/cli.tsx`:

```tsx
#!/usr/bin/env node
import React from "react";
import { render } from "ink";
import { Command } from "commander";
import { App } from "./App.js";

const program = new Command();

program
  .name("alm")
  .description("A Learning Machine CLI")
  .version("0.1.0");

program
  .command("ui")
  .description("Launch the interactive UI")
  .action(() => {
    render(<App />);
  });

program
  .command("hello")
  .description("Say hello")
  .argument("[name]", "Name to greet", "world")
  .action((name: string) => {
    console.log(`Hello, ${name}!`);
  });

program.parse();
```

## 3.4 Create the main App component

Create `apps/ink/src/App.tsx`:

```tsx
import React, { useState } from "react";
import { Box, Text, useInput, useApp } from "ink";

export function App() {
  const { exit } = useApp();
  const [message, setMessage] = useState("Welcome to A Learning Machine");

  useInput((input, key) => {
    if (input === "q" || (key.ctrl && input === "c")) {
      exit();
    }
    if (input === "h") {
      setMessage("Press 'q' to quit, 'h' for help");
    }
  });

  return (
    <Box flexDirection="column" padding={1}>
      <Box marginBottom={1}>
        <Text bold color="cyan">
          A Learning Machine
        </Text>
      </Box>
      <Box>
        <Text>{message}</Text>
      </Box>
      <Box marginTop={1}>
        <Text dimColor>Press 'h' for help, 'q' to quit</Text>
      </Box>
    </Box>
  );
}
```

## 3.5 Create a components folder

Create `apps/ink/src/components/Header.tsx`:

```tsx
import React from "react";
import { Box, Text } from "ink";

interface HeaderProps {
  title: string;
}

export function Header({ title }: HeaderProps) {
  return (
    <Box borderStyle="round" borderColor="cyan" paddingX={2}>
      <Text bold>{title}</Text>
    </Box>
  );
}
```

## Install and run

```bash
cd apps/ink
pnpm install
pnpm dev ui
```

Or run a command directly:

```bash
pnpm dev hello "Learning Machine"
```

## Build for distribution

```bash
pnpm build
```

Then you can run the built version:

```bash
node dist/cli.js ui
```

Or link it globally:

```bash
pnpm link --global
alm ui
```

## Next Steps

- [04-py-server-setup.md](04-py-server-setup.md) - Set up the FastAPI Python server
