# Project Structure

Understanding the project structure is essential for effective development and maintenance.

## Root Directory

```
hackcontrol/
├── prisma/                 # Database schema and migrations
├── public/                 # Static assets (images, icons)
├── src/                    # Application source code
├── scripts/                # Build and utility scripts
├── .env                    # Environment variables (local)
├── .env.example           # Environment template
├── package.json           # Dependencies and scripts
├── next.config.mjs        # Next.js configuration
├── tailwind.config.cjs    # Tailwind CSS configuration
├── tsconfig.json          # TypeScript configuration
└── vercel.json           # Vercel deployment settings
```

## Source Directory (`src/`)

### Application Structure
```
src/
├── animations/            # Framer Motion animations
├── components/           # Reusable React components
├── env/                 # Environment validation
├── layout/              # App layout components
├── lib/                 # Utility libraries and configurations
├── pages/               # Next.js pages and API routes
├── schema/              # Zod validation schemas
├── styles/              # Global CSS and Tailwind styles
├── trpc/                # tRPC API setup and routers
├── types/               # TypeScript type definitions
└── ui/                  # UI component library
```

## Detailed Breakdown

### `/prisma`
```
prisma/
├── migrations/          # Auto-generated migration files
└── schema.prisma       # Database schema definition
```

**Purpose**: Database schema management and migrations
- **schema.prisma**: Defines all models, relationships, and database configuration
- **migrations/**: Version-controlled database changes

### `/src/animations`
```
animations/
└── show.tsx            # Page transition animations
```

**Purpose**: Framer Motion animation components
- Page transitions and micro-interactions
- Consistent animation patterns across the app

### `/src/components`
```
components/
├── announcementDisplay.tsx    # Announcement rendering
├── announcementManager.tsx    # Admin announcement tools
├── card.tsx                   # Generic card component
├── copy.tsx                   # Copy-to-clipboard utility
├── createNew.tsx              # New hackathon creation
├── deleteHackathon.tsx        # Hackathon deletion
├── editHackathon.tsx          # Hackathon editing
├── finishHackathon.tsx        # Event completion
├── hackathonCard.tsx          # Hackathon display card
├── hackathonInfo.tsx          # Hackathon details
├── judgeManager.tsx           # Judge management
├── judgesDashboard.tsx        # Judge interface
├── leaderboardDisplay.tsx     # Results display
├── loading.tsx                # Loading states
├── participationCard.tsx      # Participation display
├── scoringInterface.tsx       # Scoring tools
├── sendProject.tsx            # Project submission
├── userSearch.tsx             # User lookup
└── viewProject.tsx            # Project viewing
```

**Purpose**: Feature-specific React components
- Business logic components
- Form handling and data submission
- Admin and user interfaces

### `/src/env`
```
env/
└── index.mjs           # Environment variable validation
```

**Purpose**: Runtime environment validation using Zod
- Validates all required environment variables
- Type-safe environment access
- Development vs production configurations

### `/src/layout`
```
layout/
├── header.tsx          # Application header
└── footer.tsx          # Application footer (if exists)
```

**Purpose**: Layout components used across all pages
- Navigation and branding
- User authentication state
- Consistent UI structure

### `/src/lib`
```
lib/
├── auth.ts             # NextAuth configuration
└── prisma.ts           # Prisma client setup
```

**Purpose**: Core library configurations and utilities
- **auth.ts**: NextAuth providers, callbacks, and session handling
- **prisma.ts**: Database client initialization and global instance

### `/src/pages`
```
pages/
├── api/
│   ├── auth/
│   │   └── [...nextauth].ts    # NextAuth API route
│   └── trpc/
│       └── [trpc].ts           # tRPC API handler
├── app/
│   ├── index.tsx               # Dashboard/app home
│   └── [url].tsx               # Individual hackathon pages
├── auth/
│   └── index.tsx               # Authentication page
├── send/
│   └── [url].tsx               # Project submission page
├── _app.tsx                    # App wrapper with providers
├── _document.tsx               # HTML document structure
└── index.tsx                   # Landing page
```

**Purpose**: Next.js file-based routing and API endpoints
- **pages/api/**: Server-side API routes
- **pages/app/**: Main application pages
- **pages/auth/**: Authentication flow
- **pages/send/**: Project submission

### `/src/schema`
```
schema/
├── announcement.ts     # Announcement validation schemas
├── hackathon.ts       # Hackathon validation schemas
├── participation.ts   # Participation validation schemas
└── scoring.ts         # Scoring validation schemas
```

**Purpose**: Zod validation schemas for API inputs
- Type-safe form validation
- API request/response validation
- Shared validation logic

### `/src/trpc`
```
trpc/
├── routers/
│   ├── announcement.router.ts  # Announcement API
│   ├── hackathon.router.ts     # Hackathon API
│   ├── judge.router.ts         # Judge API
│   ├── participation.router.ts # Participation API
│   └── scoring.router.ts       # Scoring API
├── api.ts              # Client-side tRPC setup
├── index.ts            # tRPC initialization
└── root.ts             # Main router composition
```

**Purpose**: tRPC API layer with type-safe endpoints
- **routers/**: Feature-specific API endpoints
- **api.ts**: Client configuration with React Query
- **root.ts**: Router composition and type exports

### `/src/types`
```
types/
├── hackathon.ts        # Hackathon type definitions
├── participation.ts    # Participation type definitions
└── next-auth.d.ts      # NextAuth type extensions
```

**Purpose**: TypeScript type definitions and extensions
- Database model types
- API response types
- Third-party library extensions

### `/src/ui`
```
ui/
├── icons/              # SVG icon components
└── [components].tsx    # Reusable UI components
```

**Purpose**: Design system and UI component library
- Built with Radix UI primitives
- Styled with Tailwind CSS
- Consistent design patterns

## File Naming Conventions

### Components
- **PascalCase** for component files: `HackathonCard.tsx`
- **camelCase** for utility functions: `getUserSession.ts`
- **kebab-case** for pages: `[hackathon-url].tsx`

### Directories
- **camelCase** for source directories: `src/components/`
- **kebab-case** for page directories: `pages/api/auth/`

## Import Patterns

### Path Aliases
```typescript
// Configured in tsconfig.json
{
  "paths": {
    "@/*": ["./src/*"]
  }
}
```

### Usage Examples
```typescript
// Absolute imports using alias
import { api } from "@/trpc/api";
import { authOptions } from "@/lib/auth";
import { HackathonCard } from "@/components/hackathonCard";

// Relative imports for nearby files
import { hackathonSchema } from "../schema/hackathon";
```

## Configuration Files

### Core Configuration
- **`next.config.mjs`**: Next.js framework settings
- **`tailwind.config.cjs`**: Tailwind CSS customization
- **`tsconfig.json`**: TypeScript compiler options
- **`prisma/schema.prisma`**: Database schema

### Development Tools
- **`.eslintrc.json`**: Linting rules
- **`prettier.config.cjs`**: Code formatting
- **`.prettierignore`**: Files to skip formatting

### Deployment
- **`vercel.json`**: Vercel deployment configuration
- **`.env.example`**: Environment variable template

## Development Workflow

### Adding New Features
1. Define types in `/src/types`
2. Create Zod schemas in `/src/schema`
3. Build tRPC routers in `/src/trpc/routers`
4. Develop UI components in `/src/components`
5. Create pages in `/src/pages`

### Database Changes
1. Modify `prisma/schema.prisma`
2. Run `npx prisma migrate dev --name feature-name`
3. Update related types and schemas
4. Test with `npm run studio`

## Next Steps

- Learn about [Database Schema](database-schema.md)
- Explore [tRPC API Structure](../technologies/trpc.md)
- Understand [Component Development](../development/components.md)
