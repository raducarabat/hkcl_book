# tRPC API Layer

tRPC provides end-to-end type safety for Hackcontrol's API layer, eliminating the need for manual API type definitions.

## What is tRPC?

tRPC is a library that allows you to build fully type-safe APIs without schemas or code generation. It leverages TypeScript's type system to provide:

- **End-to-end type safety**: From database to UI
- **Auto-completion**: IDE support for API calls
- **Runtime safety**: Type validation at runtime
- **Excellent DX**: No manual type synchronization

## API Architecture

### Frontend → API Flow

1. **Frontend calls tRPC**: `api.hackathon.getAll.useQuery()`
2. **tRPC routes to Next.js**: All calls go to `/api/trpc/[trpc].ts`
3. **Single handler routes internally**: The `[trpc].ts` file routes to the correct tRPC router
4. **Router executes**: The specific router (e.g., `hackathon.router.ts`) handles the request

```
Frontend Component
       ↓
api.hackathon.getAll.useQuery()
       ↓
/api/trpc/[trpc].ts (Next.js API route)
       ↓
hackathon.router.ts (tRPC router)
       ↓
Prisma → CockroachDB
```

### Router Structure

```typescript
// src/trpc/root.ts
export const appRouter = createTRPCRouter({
  hackathon: hackathonRouter,      // Hackathon management
  participation: participationRouter, // Project submissions
  announcement: announcementRouter,   // Event announcements
  judge: judgeRouter,                // Judge management
  scoring: scoringRouter,            // Scoring system
});
```

## Client Configuration

### tRPC Client Setup

```typescript
// src/trpc/api.ts
export const api = createTRPCNext<AppRouter>({
  config() {
    return {
      transformer: superjson,  // Handle Date, undefined, etc.
      links: [
        loggerLink({
          enabled: (opts) =>
            process.env.NODE_ENV === "development" ||
            (opts.direction === "down" && opts.result instanceof Error),
        }),
        httpBatchLink({
          url: `${getBaseUrl()}/api/trpc`,
        }),
      ],
    };
  },
  ssr: false,  // Client-side rendering
});
```

### App Integration

```typescript
// src/pages/_app.tsx
const App: AppType<{ session: Session | null }> = ({ Component, pageProps }) => {
  return (
    <SessionProvider session={pageProps.session}>
      <Component {...pageProps} />
    </SessionProvider>
  );
};

export default api.withTRPC(App);  // Wrap app with tRPC
```

## Router Examples

### Public Procedures

```typescript
// Anyone can call these
export const hackathonRouter = createTRPCRouter({
  getAll: publicProcedure
    .query(async ({ ctx }) => {
      return ctx.prisma.hackathon.findMany({
        include: {
          participations: true,
          owner: true,
        },
      });
    }),

  getByUrl: publicProcedure
    .input(z.object({ url: z.string() }))
    .query(async ({ ctx, input }) => {
      return ctx.prisma.hackathon.findUnique({
        where: { url: input.url },
      });
    }),
});
```

### Protected Procedures

```typescript
// Requires authentication
create: protectedProcedure
  .input(hackathonSchema)
  .mutation(async ({ ctx, input }) => {
    return ctx.prisma.hackathon.create({
      data: {
        ...input,
        creatorId: ctx.session.user.id,
      },
    });
  }),

delete: protectedProcedure
  .input(z.object({ id: z.string() }))
  .mutation(async ({ ctx, input }) => {
    // Check ownership or admin role
    const hackathon = await ctx.prisma.hackathon.findUnique({
      where: { id: input.id },
    });
    
    if (hackathon?.creatorId !== ctx.session.user.id && 
        ctx.session.user.role !== "ADMIN") {
      throw new TRPCError({ code: "FORBIDDEN" });
    }

    return ctx.prisma.hackathon.delete({
      where: { id: input.id },
    });
  }),
```

## Usage in Components

### Queries (Data Fetching)

```typescript
// Simple query
const { data: hackathons, isLoading, error } = api.hackathon.getAll.useQuery();

// Query with parameters
const { data: hackathon } = api.hackathon.getByUrl.useQuery({ 
  url: router.query.url as string 
});

// Conditional queries
const { data: scores } = api.scoring.getScores.useQuery(
  { participationId: participation.id },
  { enabled: !!participation.id }  // Only run if ID exists
);
```

### Mutations (Data Changes)

```typescript
const utils = api.useContext();

const createHackathon = api.hackathon.create.useMutation({
  onSuccess: (newHackathon) => {
    // Invalidate and refetch
    utils.hackathon.getAll.invalidate();
    
    // Or optimistic update
    utils.hackathon.getAll.setData(undefined, (old) => 
      old ? [...old, newHackathon] : [newHackathon]
    );

    toast.success("Hackathon created!");
    router.push(`/app/${newHackathon.url}`);
  },
  onError: (error) => {
    toast.error(error.message);
  },
});

// Usage in form handler
const handleSubmit = (data: HackathonInput) => {
  createHackathon.mutate(data);
};
```

## Authentication Context

### Context Creation

```typescript
// src/trpc/index.ts
export const createTRPCContext = async (opts: CreateNextContextOptions) => {
  const { req, res } = opts;
  
  // Get session from NextAuth
  const session = await getServerAuthSession({ req, res });

  return {
    session,
    prisma,
  };
};
```

### Using Session in Procedures

```typescript
// Access current user
const currentUser = ctx.session?.user;

// Check authentication
if (!ctx.session) {
  throw new TRPCError({ code: "UNAUTHORIZED" });
}

// Check admin role
if (ctx.session.user.role !== "ADMIN") {
  throw new TRPCError({ code: "FORBIDDEN" });
}
```

## Data Validation

### Input Schemas

```typescript
// Using Zod schemas from src/schema/
import { hackathonSchema } from "@/schema/hackathon";

create: protectedProcedure
  .input(hackathonSchema)  // Validates input automatically
  .mutation(async ({ ctx, input }) => {
    // input is fully typed and validated
    return ctx.prisma.hackathon.create({ data: input });
  }),
```

### Custom Validation

```typescript
getParticipations: protectedProcedure
  .input(z.object({
    hackathonId: z.string().cuid(),
    limit: z.number().min(1).max(100).default(10),
    cursor: z.string().optional(),
  }))
  .query(async ({ ctx, input }) => {
    // Pagination example
    return ctx.prisma.participation.findMany({
      where: { hackathonId: input.hackathonId },
      take: input.limit + 1,
      cursor: input.cursor ? { id: input.cursor } : undefined,
    });
  }),
```

## Error Handling

### Server Errors

```typescript
// Standard tRPC error codes
throw new TRPCError({ code: "NOT_FOUND" });      // 404
throw new TRPCError({ code: "UNAUTHORIZED" });    // 401
throw new TRPCError({ code: "FORBIDDEN" });       // 403
throw new TRPCError({ code: "BAD_REQUEST" });     // 400
throw new TRPCError({ code: "INTERNAL_SERVER_ERROR" }); // 500

// With custom messages
throw new TRPCError({
  code: "NOT_FOUND",
  message: "Hackathon not found or access denied",
});
```

### Client Error Handling

```typescript
const { data, error, isError, isLoading } = api.hackathon.getById.useQuery({ id });

if (isError) {
  // Handle specific error types
  if (error.data?.code === "NOT_FOUND") {
    return <NotFoundPage />;
  }
  if (error.data?.code === "UNAUTHORIZED") {
    router.push("/auth");
    return null;
  }
  return <ErrorDisplay message={error.message} />;
}
```

## Performance Optimization

### Request Batching

tRPC automatically batches multiple requests:

```typescript
// These will be batched into a single HTTP request
const hackathons = api.hackathon.getAll.useQuery();
const announcements = api.announcement.getAll.useQuery();
const userProfile = api.user.getProfile.useQuery();
```

### Caching Strategies

```typescript
// Cache indefinitely for static data
const { data } = api.hackathon.getById.useQuery(
  { id },
  { 
    staleTime: Infinity,
    cacheTime: Infinity,
  }
);

// Background refetch for dynamic data
const { data } = api.participation.getAll.useQuery(
  undefined,
  { 
    refetchInterval: 30000,  // Refetch every 30 seconds
    refetchOnWindowFocus: true,
  }
);
```

## Type Inference

### Router Types

```typescript
// Auto-generated type helpers
import type { RouterInputs, RouterOutputs } from "@/trpc/api";

// Input types
type HackathonCreateInput = RouterInputs["hackathon"]["create"];
type ParticipationUpdateInput = RouterInputs["participation"]["update"];

// Output types  
type HackathonData = RouterOutputs["hackathon"]["getById"];
type ParticipationList = RouterOutputs["participation"]["getAll"];
```

## Best Practices

### Router Organization
- One router per feature domain
- Keep procedures focused and single-purpose
- Use consistent naming conventions

### Input Validation
- Always validate inputs with Zod
- Reuse schemas between routers
- Provide meaningful error messages

### Authentication
- Use middleware for common auth logic
- Implement role-based access control
- Handle unauthorized access gracefully

### Error Handling
- Use appropriate tRPC error codes
- Provide user-friendly messages
- Log errors for debugging

## Next Steps

- Learn about [Prisma Integration](prisma.md)
- Explore [Zod Validation](zod.md) 
- Understand [API Development Workflow](../development/api-development.md)
