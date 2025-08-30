# Prisma ORM

Prisma serves as the database layer for Hackcontrol, providing type-safe database operations and schema management.

## What is Prisma?

Prisma is a next-generation ORM that provides:
- **Type-safe database access**
- **Auto-generated client**
- **Schema-first development**
- **Migration management**
- **Query optimization**

## Database Configuration

### Schema Definition

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "cockroachdb"
  url      = env("DATABASE_URL")
}
```

### Client Setup

```typescript
// src/lib/prisma.ts
import { PrismaClient } from "@prisma/client";
import { env } from "@/env/index.mjs";

export const prisma =
  global.prisma ||
  new PrismaClient({
    log: env.NODE_ENV === "development" 
      ? ["query", "error", "warn"] 
      : ["error"],
  });

if (env.NODE_ENV !== "production") {
  global.prisma = prisma;
}
```

## Core Models

### Hackathon Model

```prisma
model Hackathon {
  id                   String          @id @default(cuid())
  name                 String
  url                  String          @unique
  description          String?
  rules                String?
  criteria             String?
  owner                User?           @relation(fields: [creatorId], references: [id])
  creatorId            String
  updatedAt            DateTime        @default(now()) @updatedAt
  is_finished          Boolean         @default(false)
  verified             Boolean         @default(false)
  min_judges_required  Int             @default(2)
  participations       Participation[]
  announcements        Announcement[]
  Judge                Judge[]

  @@unique([url, creatorId])
  @@map(name: "hackathons")
}
```

### User Model

```prisma
model User {
  id               String          @id @default(cuid())
  name             String?
  username         String?         @unique
  email            String?         @unique
  emailVerified    DateTime?
  image            String?
  role             Role            @default(USER)
  access           Boolean         @default(true)
  accounts         Account[]
  sessions         Session[]
  hackathons       Hackathon[]
  participations   Participation[]
  judgedHackathons Judge[]
  invitedJudges    Judge[]         @relation("JudgeInviter")

  @@map(name: "users")
}
```

### Participation Model

```prisma
model Participation {
  id             String     @id @default(cuid())
  is_reviewed    Boolean    @default(false)
  is_winner      Boolean    @default(false)
  title          String
  description    String
  project_url    String
  hackathon_name String
  hackathon_url  String
  createdBy      User?      @relation(fields: [creatorId], references: [id])
  creatorId      String
  creatorName    String
  createdAt      DateTime   @default(now())
  updatedAt      DateTime   @updatedAt
  hackathon      Hackathon? @relation(fields: [hackathon_url], references: [url], onDelete: Cascade)
  scores         Score[]
  team_members   Json?

  @@unique([creatorId, hackathon_url])
  @@map(name: "participations")
}
```

## Usage in tRPC Routers

### Basic Queries

```typescript
// src/trpc/routers/hackathon.router.ts
export const hackathonRouter = createTRPCRouter({
  getAll: publicProcedure
    .query(async ({ ctx }) => {
      return ctx.prisma.hackathon.findMany({
        include: {
          participations: {
            include: {
              createdBy: true,
            },
          },
          owner: true,
        },
        orderBy: {
          updatedAt: "desc",
        },
      });
    }),

  getByUrl: publicProcedure
    .input(z.object({ url: z.string() }))
    .query(async ({ ctx, input }) => {
      const hackathon = await ctx.prisma.hackathon.findUnique({
        where: { url: input.url },
        include: {
          participations: {
            include: {
              createdBy: true,
              scores: true,
            },
          },
          announcements: {
            orderBy: { createdAt: "desc" },
          },
          Judge: {
            include: {
              user: true,
            },
          },
        },
      });

      if (!hackathon) {
        throw new TRPCError({
          code: "NOT_FOUND",
          message: "Hackathon not found",
        });
      }

      return hackathon;
    }),
});
```

### Mutations with Validation

```typescript
create: protectedProcedure
  .input(hackathonSchema)
  .mutation(async ({ ctx, input }) => {
    // Check if URL is unique
    const existing = await ctx.prisma.hackathon.findUnique({
      where: { url: input.url },
    });

    if (existing) {
      throw new TRPCError({
        code: "CONFLICT",
        message: "Hackathon URL already exists",
      });
    }

    return ctx.prisma.hackathon.create({
      data: {
        ...input,
        creatorId: ctx.session.user.id,
      },
      include: {
        owner: true,
      },
    });
  }),
```

## Advanced Prisma Features

### Transactions

```typescript
// Atomic operations
updateWithScores: protectedProcedure
  .input(participationUpdateSchema)
  .mutation(async ({ ctx, input }) => {
    return ctx.prisma.$transaction(async (tx) => {
      // Update participation
      const participation = await tx.participation.update({
        where: { id: input.id },
        data: input.data,
      });

      // Update related scores
      await tx.score.updateMany({
        where: { participationId: input.id },
        data: { updatedAt: new Date() },
      });

      return participation;
    });
  }),
```

### Aggregations

```typescript
getStats: publicProcedure
  .input(z.object({ hackathonId: z.string() }))
  .query(async ({ ctx, input }) => {
    const stats = await ctx.prisma.participation.aggregate({
      where: { hackathon: { id: input.hackathonId } },
      _count: { id: true },
      _avg: { scores: { score: true } },
    });

    return {
      totalParticipations: stats._count.id,
      averageScore: stats._avg.scores?.score || 0,
    };
  }),
```

### Raw Queries

```typescript
// For complex queries not easily expressed in Prisma
getLeaderboard: publicProcedure
  .input(z.object({ hackathonUrl: z.string() }))
  .query(async ({ ctx, input }) => {
    return ctx.prisma.$queryRaw`
      SELECT 
        p.id,
        p.title,
        p."creatorName",
        AVG(s.score)::float as average_score,
        COUNT(s.id) as judge_count
      FROM participations p
      LEFT JOIN scores s ON p.id = s."participationId"
      WHERE p.hackathon_url = ${input.hackathonUrl}
      GROUP BY p.id, p.title, p."creatorName"
      ORDER BY average_score DESC NULLS LAST
    `;
  }),
```

## Migration Management

### Development Workflow

```bash
# Modify prisma/schema.prisma
# Then create migration
npx prisma migrate dev --name add-team-system

# This will:
# 1. Generate migration SQL
# 2. Apply to database  
# 3. Regenerate Prisma client
```

### Production Deployment

```bash
# Deploy migrations to production
npx prisma migrate deploy

# Generate client (usually in build process)
npx prisma generate
```

### Schema Changes

When adding new fields:

```prisma
model Participation {
  // ... existing fields
  team_members   Json?           // New field
  submission_url String?         // Another new field
}
```

Then run:
```bash
npx prisma migrate dev --name add-team-features
```

## Database Tools

### Prisma Studio

```bash
npm run studio
```

Visual database browser for:
- Viewing and editing data
- Testing queries
- Managing relationships
- Debugging data issues

### Query Logging

Enable in development:

```typescript
const prisma = new PrismaClient({
  log: ["query", "error", "warn"],  // See all SQL queries
});
```

## Best Practices

### Model Design
- Use descriptive field names
- Add appropriate indexes for performance
- Define proper relationships
- Use enums for constrained values

### Query Optimization
- Include only needed relations
- Use select for specific fields
- Implement pagination for large datasets
- Consider database indexes

### Error Handling
- Check for null results
- Handle unique constraint violations
- Provide meaningful error messages
- Use transactions for complex operations

### Security
- Validate all inputs
- Check user permissions
- Sanitize user data
- Use parameterized queries (automatic with Prisma)

## Common Patterns

### Finding with Relations

```typescript
const hackathonWithData = await prisma.hackathon.findUnique({
  where: { url: hackathonUrl },
  include: {
    participations: {
      include: {
        createdBy: true,
        scores: {
          include: {
            judge: {
              include: {
                user: true,
              },
            },
          },
        },
      },
    },
    announcements: true,
  },
});
```

### Creating with Relations

```typescript
const participation = await prisma.participation.create({
  data: {
    title: input.title,
    description: input.description,
    project_url: input.projectUrl,
    hackathon: {
      connect: { url: input.hackathonUrl },
    },
    createdBy: {
      connect: { id: ctx.session.user.id },
    },
  },
  include: {
    hackathon: true,
    createdBy: true,
  },
});
```

## Next Steps

- Learn about [Zod Validation](zod.md)
- Explore [Database Schema](../architecture/database-schema.md)
- Understand [Development Workflow](../development/workflow.md)
