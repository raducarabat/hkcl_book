# Next.js Framework

Next.js serves as the foundation of Hackcontrol, providing the React framework with powerful features for production applications.

## Next.js in Hackcontrol

### Version & Configuration
- **Version**: Next.js 15.4.6
- **Router**: Pages Router (not App Router)
- **Configuration**: `next.config.mjs`

### Key Features Used

#### File-Based Routing
```
src/pages/
├── index.tsx              # / (Landing page)
├── auth/
│   └── index.tsx          # /auth (Sign in page)
├── app/
│   ├── index.tsx          # /app (Dashboard)
│   └── [url].tsx          # /app/[hackathon-url]
└── send/
    └── [url].tsx          # /send/[hackathon-url]
```

#### API Routes
```
src/pages/api/
├── auth/
│   └── [...nextauth].ts  # /api/auth/* (NextAuth)
└── trpc/
    └── [trpc].ts         # /api/trpc/* (tRPC handler)
```

## Configuration

### Next.js Config

```javascript
// next.config.mjs
!process.env.SKIP_ENV_VALIDATION && (await import("./src/env/index.mjs"));

const nextConfig = {
  reactStrictMode: true,
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "avatars.githubusercontent.com",
        port: "",
        pathname: "/u/**",
      },
    ],
  },
};

export default nextConfig;
```

**Key settings:**
- **Environment validation**: Loads and validates env vars at build time
- **React Strict Mode**: Enabled for development safety
- **Image optimization**: Configured for GitHub avatars

### TypeScript Integration

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "strict": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

## App Structure

### Root App Component

```typescript
// src/pages/_app.tsx
const App: AppType<{ session: Session | null }> = ({
  Component,
  pageProps: { session, ...pageProps },
  router,
}) => {
  return (
    <SessionProvider session={session}>
      <DefaultSeo {...nextSeoConfig} />
      <NextNProgress />
      <main className="font-sans">
        <Header />
        <Show routerKey={router.route}>
          <Component {...pageProps} />
        </Show>
        <Toaster />
      </main>
    </SessionProvider>
  );
};

export default api.withTRPC(App);
```

**Features integrated:**
- **SessionProvider**: NextAuth session context
- **DefaultSeo**: SEO configuration
- **NextNProgress**: Page loading indicator
- **Show**: Page transition animations
- **Toaster**: Toast notifications (Sonner)
- **api.withTRPC**: tRPC client wrapper

### Document Structure

```typescript
// src/pages/_document.tsx
// Custom document for HTML structure and font loading
```

## Page Examples

### Dynamic Routes

```typescript
// src/pages/app/[url].tsx
import { type GetServerSideProps } from "next";
import { getServerAuthSession } from "@/lib/auth";

export default function HackathonPage() {
  const router = useRouter();
  const hackathonUrl = router.query.url as string;
  
  const { data: hackathon, isLoading } = api.hackathon.getByUrl.useQuery({
    url: hackathonUrl,
  });

  if (isLoading) return <Loading />;
  if (!hackathon) return <NotFound />;

  return <HackathonDetails hackathon={hackathon} />;
}

export const getServerSideProps: GetServerSideProps = async (ctx) => {
  const session = await getServerAuthSession(ctx);
  
  return {
    props: {
      session,
    },
  };
};
```

### API Route Handler

```typescript
// src/pages/api/trpc/[trpc].ts
import { createNextApiHandler } from "@trpc/server/adapters/next";
import { appRouter } from "@/trpc/root";
import { createTRPCContext } from "@/trpc";

export default createNextApiHandler({
  router: appRouter,
  createContext: createTRPCContext,
  onError: env.NODE_ENV === "development"
    ? ({ path, error }) => {
        console.error(`❌ tRPC failed on ${path}: ${error.message}`);
      }
    : undefined,
});
```

## Features & Integrations

### Server-Side Rendering (SSR)

```typescript
// For pages that need server-side data
export const getServerSideProps: GetServerSideProps = async (ctx) => {
  const session = await getServerAuthSession(ctx);
  
  // Pre-fetch data on server
  const helpers = createServerSideHelpers({
    router: appRouter,
    ctx: { session, prisma },
    transformer: superjson,
  });

  await helpers.hackathon.getByUrl.prefetch({ url: ctx.params?.url as string });

  return {
    props: {
      trpcState: helpers.dehydrate(),
      session,
    },
  };
};
```

### Image Optimization

```typescript
// GitHub avatar optimization
<Image
  src={user.image}
  alt={user.name}
  width={40}
  height={40}
  className="rounded-full"
/>
```

### SEO Configuration

```typescript
// next-seo.config.ts
export const nextSeoConfig: DefaultSeoProps = {
  title: "Hackcontrol",
  description: "Open-source hackathon management system",
  openGraph: {
    type: "website",
    locale: "en_US",
    url: "https://hackcontrol.vercel.app/",
    siteName: "Hackcontrol",
  },
};
```

## Performance Features

### Automatic Code Splitting
- Each page is automatically split into separate bundles
- Reduces initial bundle size
- Improves load times

### Static Optimization
- Static pages are pre-rendered at build time
- Dynamic pages use SSR when needed
- Optimal performance for different content types

### Image Optimization
- Automatic image optimization and resizing
- WebP format support
- Lazy loading by default

## Development Features

### Hot Reloading
- Instant feedback during development
- Preserves component state when possible
- Fast refresh for React components

### Error Overlay
- Detailed error information in development
- Source map support for debugging
- Stack trace with exact file locations

### TypeScript Integration
- Zero-config TypeScript support
- Fast compilation with SWC
- Built-in type checking

## Build Process

### Development Build
```bash
npm run dev
```
- Fast refresh enabled
- Source maps for debugging
- Detailed error reporting

### Production Build
```bash
npm run build
npm run start
```
- Optimized bundles
- Static asset optimization
- Performance optimizations

## Environment Handling

### Environment Variables
```typescript
// Automatic environment validation
!process.env.SKIP_ENV_VALIDATION && (await import("./src/env/index.mjs"));
```

### Runtime Configuration
- Client vs server environment separation
- Secure environment variable access
- Build-time validation

## Deployment

### Vercel Integration
```json
// vercel.json
{
  "functions": {
    "src/pages/api/**/*.ts": {
      "maxDuration": 30
    }
  }
}
```

### Static Assets
- Automatic optimization
- CDN distribution
- Caching headers

## Best Practices

### Page Organization
- Group related pages in directories
- Use meaningful file names
- Implement proper error boundaries

### Performance
- Minimize client-side JavaScript
- Use dynamic imports for large components
- Optimize images and assets

### SEO
- Implement proper meta tags
- Use semantic HTML structure
- Optimize for Core Web Vitals

## Routing Patterns

### Dynamic Routes
```typescript
// [url].tsx - Single dynamic segment
// [...slug].tsx - Catch-all routes
// [[...slug]].tsx - Optional catch-all
```

### Programmatic Navigation
```typescript
import { useRouter } from "next/router";

const router = useRouter();

// Navigate to different pages
router.push("/app");
router.push(`/app/${hackathon.url}`);
router.replace("/auth"); // Replace instead of push
```

## Next Steps

- Learn about [TypeScript Configuration](typescript.md)
- Explore [Tailwind CSS Integration](tailwind.md)
- Understand [Development Workflow](../development/workflow.md)
