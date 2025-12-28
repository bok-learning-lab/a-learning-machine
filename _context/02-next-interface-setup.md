# Step 2: Next.js Interface Setup

Set up the Next.js 15 web app with Tailwind v4 and shadcn/ui.

## Option A: Use create-next-app (recommended)

From repo root:

```bash
pnpm dlx create-next-app@latest apps/interface \
  --ts --eslint --tailwind --app --src-dir --import-alias "@/*"
```

This scaffolds a complete Next.js app. When prompted:
- TypeScript: Yes
- ESLint: Yes
- Tailwind CSS: Yes
- `src/` directory: Yes
- App Router: Yes
- Import alias: `@/*`

## Option B: Manual setup

### 2.1 Create package.json

Create `apps/interface/package.json`:

```json
{
  "name": "@a-learning-machine/interface",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "^15.1.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@types/node": "^22.10.2",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "typescript": "^5.7.2",
    "eslint": "^9.0.0",
    "eslint-config-next": "^15.1.0",
    "@tailwindcss/postcss": "^4.0.0",
    "tailwindcss": "^4.0.0"
  }
}
```

### 2.2 Create tsconfig.json

Create `apps/interface/tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### 2.3 Create next.config.ts

Create `apps/interface/next.config.ts`:

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  // Add config options here
};

export default nextConfig;
```

### 2.4 Create postcss.config.mjs

Create `apps/interface/postcss.config.mjs`:

```javascript
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};

export default config;
```

### 2.5 Create app structure

Create `apps/interface/src/app/globals.css`:

```css
@import "tailwindcss";
```

Create `apps/interface/src/app/layout.tsx`:

```tsx
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = {
  title: "A Learning Machine",
  description: "A workspace for learning, writing, and building",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

Create `apps/interface/src/app/page.tsx`:

```tsx
export default function Home() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-center p-24">
      <h1 className="text-4xl font-bold">A Learning Machine</h1>
      <p className="mt-4 text-lg text-gray-600">
        Your workspace for learning, writing, and building.
      </p>
    </main>
  );
}
```

## Add shadcn/ui (optional but recommended)

After initial setup:

```bash
cd apps/interface
pnpm dlx shadcn@latest init
```

When prompted:
- Style: Default
- Base color: Slate (or your preference)
- CSS variables: Yes

Then add components as needed:

```bash
pnpm dlx shadcn@latest add button card input
```

## Run it

```bash
cd apps/interface
pnpm dev
```

Open http://localhost:3000

## Next Steps

- [03-ink-cli-setup.md](03-ink-cli-setup.md) - Set up the Ink TUI/CLI
